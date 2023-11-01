### SQL语句1

- 通过`user_id`在数据配置权限表和数据配置表中查出当前用户所具有的所有配置数据以及对应权限：

  ```sql
  SELECT
      dcp.config_id,
      dcp.env,
      dcp.app_id,
      dc.app_name,
      dcp.authority_type
  FROM
      数据配置权限表 AS dcp
  INNER JOIN
      数据配置表 AS dc ON dcp.env = dc.env AND dcp.app_id = dc.app_id
  WHERE
      dcp.user_id = [user_id];
  
  ```

  这将返回一个结果集，其中包含用户具有权限的配置数据以及相应的权限。

- 使用上一步查询到的配置数据中的`env`和`app_id`来查询数据表，以查出对应的数据：

```sql
SELECT
    dt.data_id,
    dt.env,
    dt.app_id,
    dt.app_name,
    -- 其他数据表字段
FROM
    数据表 AS dt
INNER JOIN
    (
        SELECT DISTINCT
            dcp.env,
            dcp.app_id
        FROM
            数据配置权限表 AS dcp
        WHERE
            dcp.user_id = [user_id]
    ) AS config_data ON dt.env = config_data.env AND dt.app_id = config_data.app_id;

```

这将返回用户具有权限的配置数据对应的数据表中的数据。注意，需要将`[user_id]` 替换为实际的用户ID。

### sql语句2

如果数据配置权限表中不包含`env`和`app_id`字段，而是与配置表中的`config_id`对应，你可以使用以下SQL查询来查找当前用户所具有的所有配置数据以及对应权限，然后再使用这些数据查询数据表

- 通过`user_id`在数据配置权限表中查出当前用户所具有的所有`config_id`以及对应权限：

```sql
SELECT
    dcp.config_id,
    dcp.authority_type
FROM
    数据配置权限表 AS dcp
WHERE
    dcp.user_id = [user_id];

```

这将返回用户具有的`config_id`以及对应的权限。

- 使用上一步查询到的`config_id`来联接数据配置表，以获取`env`和`app_id`，然后再使用这些数据查询数据表：

```sql
SELECT
    dt.data_id,
    dt.env,
    dt.app_id,
    dt.app_name,
    -- 其他数据表字段
FROM
    数据表 AS dt
INNER JOIN
    数据配置表 AS dc ON dt.env = dc.env AND dt.app_id = dc.app_id
INNER JOIN
    (
        SELECT
            dcp.config_id
        FROM
            数据配置权限表 AS dcp
        WHERE
            dcp.user_id = [user_id]
    ) AS user_configs ON dc.config_id = user_configs.config_id;

```

这将返回用户具有权限的配置数据对应的数据表中的数据。请确保将`[user_id]` 替换为实际的用户ID。

这个查询将首先获取用户所具有的`config_id`，然后通过这些`config_id`联接配置表以获取相应的`env`和`app_id`，最后再通过这些数据查询数据表以获取相应的数据。

### sql语句3

- 通过`user_id`在数据配置权限表和数据配置表中查出当前用户所具有的所有配置数据以及对应权限

```sql
SELECT
    dcp.authority_id,
    dc.config_id,
    dc.env,
    dc.app_id,
    dc.app_name,
    dcp.authority_type
FROM
    数据配置权限表 AS dcp
INNER JOIN
    数据配置表 AS dc ON dcp.config_id = dc.config_id
WHERE
    dcp.user_id = [user_id];

```

这将返回一个结果集，其中包含用户具有权限的配置数据以及相应的权限。

- 使用上一步查询到的配置数据中的`config_id`来联接数据表，以查出对应的数据：

```sql
SELECT
    dt.data_id,
    dt.env,
    dt.app_id,
    dt.app_name,
    -- 其他数据表字段
FROM
    数据表 AS dt
INNER JOIN
    (
        SELECT DISTINCT
            dc.config_id
        FROM
            数据配置权限表 AS dcp
        INNER JOIN
            数据配置表 AS dc ON dcp.config_id = dc.config_id
        WHERE
            dcp.user_id = [user_id]
    ) AS user_configs ON dt.env = user_configs.env AND dt.app_id = user_configs.app_id;

```

这将返回用户具有权限的配置数据对应的数据表中的数据。请确保将`[user_id]` 替换为实际的用户ID。

这个查询将首先获取用户所具有的`config_id`，然后通过这些`config_id`联接数据表以获取相应的数据。

### 代码实例

- 创建数据表对应的实体类、数据配置表对应的实体类以及数据配置权限表对应的实体类。在这些实体类中定义字段对应的属性，并在数据配置表实体类中包含一个配置数据的集合。这些实体类通常需要使用 `@Entity` 注解（如果你使用 JPA）或者 `@Mapper` 注解（如果你使用 MyBatis）。

  ```java
  // 示例实体类
  @Entity
  public class DataEntity {
      private Long dataId;
      private String env;
      private String appId;
      private String appName;
      // 其他字段和方法
  }
  
  @Entity
  public class DataConfigEntity {
      private Long configId;
      private String env;
      private String appId;
      private String appName;
      // 其他字段和方法
  }
  
  @Entity
  public class DataConfigAuthorityEntity {
      private Long authorityId;
      private Long configId;
      private Long userId;
      private Integer authorityType;
      // 其他字段和方法
  }
  
  ```

- 创建数据访问层（DAO）接口和相应的 XML 映射文件，以定义数据库查询操作。在这些接口中，你可以定义用于查询数据配置、数据配置权限和数据的方法。

```java
// 示例 DataRepository.java 接口
public interface DataRepository {
    List<DataConfigEntity> findDataConfigsByUserId(Long userId);
    List<DataEntity> findDataByConfigIds(List<Long> configIds);
    // 其他查询方法
}

```

```xml
<!-- 示例 DataMapper.xml 文件 -->
<mapper namespace="com.example.mapper.DataRepository">
    <select id="findDataConfigsByUserId" resultType="com.example.entity.DataConfigEntity">
        SELECT * FROM data_config
        WHERE config_id IN (
            SELECT config_id FROM data_config_authority
            WHERE user_id = #{userId}
        )
    </select>
    <!-- 其他查询语句 -->
</mapper>

```

- 创建一个服务类，将数据访问层的方法封装为业务逻辑。在服务类中，你可以调用数据访问层的方法来获取数据配置以及权限，然后进一步查询数据。

```java
@Service
public class DataService {
    @Autowired
    private DataRepository dataRepository;

    public List<DataConfigEntity> findDataConfigsByUserId(Long userId) {
        return dataRepository.findDataConfigsByUserId(userId);
    }

    public List<DataEntity> findDataByConfigIds(List<Long> configIds) {
        return dataRepository.findDataByConfigIds(configIds);
    }
    // 其他业务逻辑方法
}

```

- 最后，在你的控制器（Controller）中，接收用户的请求，调用服务类的方法，并返回结果。

```java
@RestController
@RequestMapping("/api/data")
public class DataController {
    @Autowired
    private DataService dataService;

    @GetMapping("/user/{userId}")
    public ResponseEntity<List<DataEntity>> getDataForUser(@PathVariable Long userId) {
        List<DataConfigEntity> dataConfigs = dataService.findDataConfigsByUserId(userId);
        List<Long> configIds = dataConfigs.stream().map(DataConfigEntity::getConfigId).collect(Collectors.toList());
        List<DataEntity> data = dataService.findDataByConfigIds(configIds);
        return ResponseEntity.ok(data);
    }
    // 其他控制器方法
}

```

这是一个简单的示例，你可以根据你的具体需求和数据模型进行更详细的实现。确保在 Spring Boot 项目中配置好数据库连接和 MyBatis，以及正确设置相关的注解和 XML 映射文件。