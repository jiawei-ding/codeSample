- 如何实现复杂查询

  首先，创建一个通用的`UserSpecification`类，用于构建不同查询条件：

```java
import org.springframework.data.jpa.domain.Specification;

public class UserSpecification {

    public static Specification<User> ageGreaterThanOrEqual(int age) {
        return (root, query, criteriaBuilder) ->
                criteriaBuilder.greaterThanOrEqualTo(root.get("age"), age);
    }

    public static Specification<User> usernameNotContaining(String username) {
        return (root, query, criteriaBuilder) ->
                criteriaBuilder.notLike(root.get("username"), "%" + username + "%");
    }

    public static Specification<User> activeIsFalse() {
        return (root, query, criteriaBuilder) ->
                criteriaBuilder.isFalse(root.get("active"));
    }
}

```

​	然后，在你的`UserService`中，你可以使用这些`Specification`来构建查询条件：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.data.jpa.domain.Specifications;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {

    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<User> findUsersByComplexCriteria(int age, String username) {
        Specification<User> spec = Specifications
                .where(UserSpecification.ageGreaterThanOrEqual(age))
                .and(UserSpecification.usernameNotContaining(username))
                .or(UserSpecification.activeIsFalse());

        return userRepository.findAll(spec);
    }
}

```

​		在这个示例中，我们使用`Specifications`来组合多个`Specification`，从而构建一个包含		多个条件的查询条件。然后，我们使用这个查询条件在`userRepository`上执行查询。

​		这种方式允许你动态构建查询条件，根据需要组合不同的条件，而不需要为每种查询情况编		写单独的接口和实现类。这对于处理通用的、多条件的查询非常有用。

- 上述代码中findUsersByComplexCriteria依然写死了，我想要一种更通用的方法，比如可以实现任意组合查询

  - 要实现大于等于25岁并且小于等于30岁并且用户名不包含"admin"或者用户不是激活状态的用户的查询，你可以继续使用上述通用性方法，只需设置不同的查询条件。在`UserSearchCriteria`类中，你可以添加新的属性来表示年龄的上限，例如`maxAge`。然后，根据这些新的条件进行查询。

    首先，更新`UserSearchCriteria`类：

    ```java
    public class UserSearchCriteria {
        private Integer minAge;
        private Integer maxAge;
        private String username;
        private boolean active;
    
        // 省略 getter 和 setter 方法
    }
    
    ```

    接下来，你可以在前端设置新的查询条件：

    ```java
    UserSearchCriteria criteria = new UserSearchCriteria();
    criteria.setMinAge(25);
    criteria.setMaxAge(30);
    criteria.setUsername("admin");
    criteria.setActive(false);
    
    List<User> result = userService.findUsersByCriteria(criteria);
    
    ```

    最后，在`UserService`中，使用这些新的条件进行查询：

    ```java
    public List<User> findUsersByCriteria(UserSearchCriteria criteria) {
        return userRepository.findAll((root, query, criteriaBuilder) -> {
            List<Predicate> predicates = new ArrayList<>();
    
            if (criteria.getMinAge() != null) {
                predicates.add(criteriaBuilder.greaterThanOrEqualTo(root.get("age"), criteria.getMinAge()));
            }
            
            if (criteria.getMaxAge() != null) {
                predicates.add(criteriaBuilder.lessThanOrEqualTo(root.get("age"), criteria.getMaxAge()));
            }
    
            if (criteria.getUsername() != null && !criteria.getUsername().isEmpty()) {
                predicates.add(criteriaBuilder.notLike(root.get("username"), "%" + criteria.getUsername() + "%"));
            }
    
            if (!criteria.isActive()) {
                predicates.add(criteriaBuilder.isFalse(root.get("active")));
            }
    
            // Add more conditions as needed...
    
            return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
        });
    }
    
    ```

    这样，你可以实现大于等于25岁并且小于等于30岁并且用户名不包含"admin"或者用户不是激活状态的查询，无需修改后端代码，只需设置不同的查询条件。这种通用性方法适用于各种不同的组合查询条件。

- 上面代码中取消lambda表达式的用法，换为传统函数式的代码

  - 当你想要避免使用Lambda表达式时，可以使用传统的匿名内部类来编写JPA的Specification。以下是使用传统的匿名内部类的示例：

    首先，更新`UserService`类的`findUsersByCriteria`方法：

    ```java
    public List<User> findUsersByCriteria(UserSearchCriteria criteria) {
        return userRepository.findAll(new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
                List<Predicate> predicates = new ArrayList<>();
    
                if (criteria.getMinAge() != null) {
                    predicates.add(criteriaBuilder.greaterThanOrEqualTo(root.get("age"), criteria.getMinAge()));
                }
    
                if (criteria.getMaxAge() != null) {
                    predicates.add(criteriaBuilder.lessThanOrEqualTo(root.get("age"), criteria.getMaxAge()));
                }
    
                if (criteria.getUsername() != null && !criteria.getUsername().isEmpty()) {
                    predicates.add(criteriaBuilder.notLike(root.get("username"), "%" + criteria.getUsername() + "%"));
                }
    
                if (!criteria.isActive()) {
                    predicates.add(criteriaBuilder.isFalse(root.get("active")));
                }
    
                // Add more conditions as needed...
    
                return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
            }
        });
    }
    
    ```

    在这个示例中，我们使用了匿名内部类来实现`Specification`接口，并在`toPredicate`方法中构建查询条件。

    这种方法避免了Lambda表达式的使用，但代码相对冗长。根据个人喜好和项目的代码规范，你可以选择使用Lambda表达式或传统的匿名内部类来编写JPA的Specification。

    最后一句代码 `return criteriaBuilder.and(predicates.toArray(new Predicate[0]));` 的作用是将所有的查询条件通过逻辑"与"（AND）运算组合在一起，以构建完整的查询条件。

    在这段代码中，我们创建了一个 `predicates` 列表，用于存储所有的查询条件（例如年龄、用户名、激活状态等条件）。然后，我们使用 `criteriaBuilder.and(...)` 方法将这些条件组合成一个整体的查询条件，要求所有条件都必须满足。这意味着只有当所有条件都为真时，才会匹配到结果。

    通过 `predicates.toArray(new Predicate[0])`，我们将 `predicates` 列表转换为 `Predicate` 数组，以便传递给 `criteriaBuilder.and(...)` 方法。

    这种方式允许你将多个条件组合在一起，构建复杂的查询，同时确保所有条件都必须满足才能返回结果。如果你需要使用"或"运算，可以使用 `criteriaBuilder.or(...)` 方法来组合条件。