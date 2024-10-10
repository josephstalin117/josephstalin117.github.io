---
layout: post
title:  "python性能优化"
date:   2024-10-9 20:35:35 +0800
categories: command
---

# 生成18万条随机数据

```
import pandas as pd
import numpy as np
 
 
def gen_big_data(csv_file: str, big_data_count=90000):
    chars = 'abcdefghijklmnopqrstuvwxyz'
    dates = pd.date_range(start='2020-01-01', periods=big_data_count, freq='30s')
    big_data_cols = ['Name']
    for group in range(1, 31):
        big_data_cols.extend([f'date str {group}',
                              f'bool {group}',
                              f'int {group}',
                              f'float {group}',
                              f'str {group}'])
    big_data = []
    for i in range(0, big_data_count):
        row = [f'Name Item {(i + 1)}']
        for _ in range(0, 30):
            row.extend([str(dates[i]),
                        i % 2 == 0,
                        np.random.randint(10000, 100000),
                        10000 * np.random.random(),
                        chars[np.random.randint(0, 26)] * 15])
        big_data.append(row)
    df = pd.DataFrame(data=big_data, columns=big_data_cols)
    df.to_csv(csv_file, index=None)
 
 
if __name__ == '__main__':
    # 修改存放路径以及模拟数据量(默认9万)
    gen_big_data('./files/custom_big_data.csv', 180000)
```

# 内存占用查询

```
# 查看内存占用
df.info(memory_usage='deep')

# 查看数据类型内存占用
for d_type in ['bool', 'float64', 'int64', 'object']:
    d_type_selected = df.select_dtypes(include=[d_type])
    mem_mean_bit = d_type_selected.memory_usage(deep=True).mean()
    mem_mean_mb = mem_mean_bit / 1024 ** 2
    print(f'mean memory usage: {d_type:<7} - {mem_mean_mb:.3f} M')
```

查看某类型内存占用
```
def info_mem_usage_mb(pd_obj):
    if isinstance(pd_obj, pd.DataFrame):
        mem_usage = pd_obj.memory_usage(deep=True).sum()
    else:
        mem_usage = pd_obj.memory_usage(deep=True)
    # 转换为MB返回
    return f'{mem_usage / 1024 ** 2:02.3f} MB'
```

# 数据类型优化

## 优化int&float数据类型

优化数据类型可以减少内存使用。对于数值数据，可以选择使用内存占用更小的数值类型，如`int8`或`float32`，而非默认的`int64`或`float64`。同样，对于值重复率高的字符串列，将其转换为`category`类型可以显著降低内存使用。日期和时间数据最好使用专门的`datetime`类型。

对于`int`和`float`类型的数据，`Pandas`加载到内存中的数据，默认是`int64`和`float64`。一般场景下的数据，用`int32`和`float32`就足够了，用`numpy.iinfo`和`numpy.finfo`可以打印对应类型的取值范围
```
def optimize_int_and_float():
    df_int = df.select_dtypes(include=['int64'])
    df_int_converted = df_int.apply(pd.to_numeric, downcast='unsigned')
    df_float = df.select_dtypes(include=['float64'])
    df_float_converted = df_float.apply(pd.to_numeric, downcast='float')
 
    print('int before     ', info_mem_usage_mb(df_int))
    print('int converted  ', info_mem_usage_mb(df_int_converted))
 
    print('float before   ', info_mem_usage_mb(df_float))
    print('float converted', info_mem_usage_mb(df_float_converted))


df['A'] = df['A'].astype('int8')  # 更小的整数类型
df['B'] = df['B'].astype('category')  # 分类类型
```

## 优化object类型

获取`object`类型数据，并调用`describe()`展示统计信息
```
df_object = df.select_dtypes(include=['object])
df_object.describe()
```
对于区分度较低的`str`，一共只有26个可能的值，可以考虑转换为`Pandas`中的`categroy`类型，这里将区分度小于40%的列转换为`category`类型
```
def optimize_obj():
    df_obj = df.select_dtypes(include=['object'])
    df_obj_converted = pd.DataFrame()
    for col in df_obj.columns:
        unique_count = len(df_obj[col].unique())
        total_count = len(df_obj[col])
        # 将区分度小于40%的列转换为category类型
        if unique_count / total_count <= 0.4:
            df_obj_converted.loc[:, col] = df_obj[col].astype('category')
        else:
            df_obj_converted.loc[:, col] = df_obj[col]
 
    print('object before   ', info_mem_usage_mb(df_obj))
    print('object converted', info_mem_usage_mb(self.df_obj_converted))
```

## 优化date数据
将`date`字符串转换为`date`类型
```
def optimize_date_str():
    df_date = pd.DataFrame()
    df_date_converted = pd.DataFrame()
    for col_name in df.columns:
        if col_name.startswith('date str'):
            df_date.loc[:, col_name] = df[col_name]
            df_date_converted.loc[:, col_name] = pd.to_datetime(df[col_name])
 
    print('date before   ', info_mem_usage_mb(df_date))
    print('date converted', info_mem_usage_mb(df_date_converted))
```

# 向量化操作
尽量使用`Pandas`的内置向量化操作而非循环。向量化操作通常更高效。`Pandas`提供了大量的向量化操作，可以提高数据操作的效率。如`sum()`、`mean()`、`max()`等函数可以直接作用于整个`DataFrame` 或`Series`，而不需要使用循环。可以显著提高数据处理的速度和效率，特别是在处理大型数据集时。它们利用了`Pandas`和`NumPy`库的内部优化，使得操作更加高效，避免了相对开销较大的`Python`循环。
```
import pandas as pd
import numpy as np

# 创建示例 DataFrame
df = pd.DataFrame({
    'column1': [1, 2, -3, 4, -5],
    'column2': [5, 6, 7, -8, -9]
})

# 向量化操作
df['sum'] = df['column1'] + df['column2']

# 使用 apply() 方法
df['transformed_column1'] = df.apply(lambda x: x['column1'] * 2 if x['column2'] > 0 else x['column1'], axis=1)

# 使用 map() 和 applymap()
df['mapped_column1'] = df['column1'].map(lambda x: x * 2)
df = df.applymap(lambda x: x * 2 if isinstance(x, int) else x)

# 使用 groupby() 进行分组操作
grouped_sum = df.groupby('mapped_column1').sum()

# 使用 Pandas 的内置函数
total_column1 = df['column1'].sum()

# 使用条件表达式
df['new_column'] = np.where(df['column2'] > 0, 'positive', 'non-positive')

# 显示结果
print("DataFrame with Applied Operations:\n", df)
print("\nGrouped Sum:\n", grouped_sum)
print("\nTotal of 'column1':", total_column1)
```

# 优化文件读取

指定读取文件数据类型
```
def big_data_optimized_read_csv(self):
    d_type_dict = {}
    date_indexes = []
    for i in range(1, 31):
        d_type_dict[f'int {i}'] = 'int32'
        d_type_dict[f'float {i}'] = 'float32'
        d_type_dict[f'str {i}'] = 'category'
 
        date_indexes.append(5 * (i - 1) + 1)
    self.df = pd.read_csv(self.csv_file, dtype=d_type_dict, parse_dates=date_indexes)
    print('optimized read_csv: ', self.info_mem_usage_mb(self.df))
```

# 并行运行

对于大型操作，考虑使用并行处理来加速。使用并行处理是一个有效的优化技巧。尽管`Pandas`本身不是为并行处理而设计的，但可以通过一些方法来利用多核处理器的能力，从而加速数据处理任务。`Dask`提供了一个与`Pandas`类似的大型并行`DataFrame`，适用于处理大数据集；`Joblib`可以高效运行多个`Python`进程，适合简单的并行化任务；而`Python`的`multiprocessing`模块允许手动创建并行任务，通过将大型`DataFrame`分割成多个小块，在每个处理器核心上并行处理这些块。
```
import pandas as pd
import numpy as np
from multiprocessing import Pool

# 示例函数，对数据进行某种复杂计算
def my_complex_function(data_chunk):
    return data_chunk.apply(np.sin)

# 创建一个大型 DataFrame
df = pd.DataFrame(np.random.rand(1000000, 4), columns=['A', 'B', 'C', 'D'])

# 将 DataFrame 分割成多个小块
data_chunks = np.array_split(df, 4)

# 创建一个进程池并并行处理每个数据块
with Pool(4) as pool:
    results = pool.map(my_complex_function, data_chunks)

# 合并结果
final_result = pd.concat(results)
print(final_result)
```


