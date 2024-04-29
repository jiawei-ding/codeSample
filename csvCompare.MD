#### PANDAS

```python
import pandas as pd

def compare_csv(file1, file2):
    # 读取两个CSV文件
    df1 = pd.read_csv(file1)
    df2 = pd.read_csv(file2)

    # 比较两个DataFrame
    diff_df = df1.compare(df2)

    # 输出不同的行
    if diff_df.empty:
        print("两个CSV文件内容完全相同！")
    else:
        print("以下是内容不同的行：")
        print(diff_df)

# 用法示例
file1 = "file1.csv"
file2 = "file2.csv"
compare_csv(file1, file2)

```

在这个脚本中，我们首先使用 pandas 的 `read_csv` 函数读取两个 CSV 文件，然后使用 `compare` 函数比较两个 DataFrame，找出不同之处，并将结果存储在一个新的 DataFrame 中。最后，如果存在不同的行，则将其输出；否则，提示两个文件内容完全相同。

#### csv

```python
import csv

def compare_csv(file1, file2):
    with open(file1, 'r') as f1, open(file2, 'r') as f2:
        reader1 = csv.reader(f1)
        reader2 = csv.reader(f2)

        line_number = 1
        for row1, row2 in zip(reader1, reader2):
            if row1 != row2:
                print(f"文件1第{line_number}行：{row1}")
                print(f"文件2第{line_number}行：{row2}")
            line_number += 1

# 用法示例
file1 = "file1.csv"
file2 = "file2.csv"
compare_csv(file1, file2)

```

在这个脚本中，我们打开两个文件，并使用 `csv.reader` 逐行读取它们。然后，使用 `zip` 函数将两个文件的行进行逐行配对，然后比较这两行是否相同。如果不同，就输出它们及其所在的行号。