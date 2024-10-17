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
优化内存使用是提高效率和性能的一个重要方面。尤其在处理大型数据集时，有效管理内存是至关重要的。通过`df.memory_usage(deep=True)`可以检查`DataFrame`的每列占用的内存量。

```
import pandas as pd
import numpy as np

# 创建一个示例 DataFrame
np.random.seed(0)
data = {
    'float_col': np.random.rand(10000),
    'int_col': np.random.randint(0, 100, size=10000),
    'category_col': np.random.choice(['A', 'B', 'C', 'D'], size=10000)
}
df = pd.DataFrame(data)

# 检查每列的内存使用情况
memory_usage_before = df.memory_usage(deep=True)

# 节省内存的数据类型转换
df['float_col'] = df['float_col'].astype('float32')
df['int_col'] = df['int_col'].astype('int16')

# 类别数据类型转换
df['category_col'] = df['category_col'].astype('category')

# 检查优化后的内存使用
memory_usage_after = df.memory_usage(deep=True)

# 输出内存使用情况
print(memory_usage_before, memory_usage_after)
```


### 优化int&float数据类型

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

### 优化object类型

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

### 优化date数据
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

# 使用索引
为`DataFrame`设置适当的索引可以提高数据检索的效率。高效地使用索引也是提升数据操作性能的关键之一。为了优化数据操作，首先应选择合适的索引。常见做法包括将频繁查询的列设置为索引，利用`set_index`方法，以及在复杂数据集上使用多级索引。访问数据时，应通过`loc`和`iloc`索引器高效地访问数据，特别是在使用索引列进行条件查询时，这比全表扫描更有效。同时，需要注意索引的内存消耗，过多索引会增加内存负担，因此在内存有限的情况下要平衡索引数量和性能提升。去除不再需要的索引可以使用`reset_index`方法。在索引操作上，应避免在循环中频繁修改`DataFrame`索引，这是一种低效的操作。
```
import pandas as pd
import numpy as np

# 构建一个示例股票数据集
np.random.seed(0)
dates = pd.date_range('20240101', periods=6)
stocks = ['AAPL', 'MSFT', 'GOOG']
data = pd.DataFrame(np.random.randn(6, 3), index=dates, columns=stocks)

# 将日期和股票代码作为多级索引
data = data.stack().reset_index()
data.columns = ['日期', '股票代码', '价格']
data.set_index(['日期', '股票代码'], inplace=True)

# 按索引排序
data.sort_index(inplace=True)

# 查询特定日期的所有股票数据
data_on_specific_date = data.loc['2024-01-01']

# 查询特定股票代码的数据
data_for_specific_stock = data.xs('AAPL', level='股票代码')

# 显示查询结果
print(data_on_specific_date, data_for_specific_stock)
```

# 避免链式赋值
链式赋值指的是在一个单独的表达式中连续对`DataFrame`进行多个操作。虽然这种写法看起来简洁，但可能会导致意外的行为和效率问题。链式赋值可能导致对`DataFrame`的修改无法确定是在原始数据上还是副本上进行，有时甚至可能导致警告或错误。此外，连续操作可能会导致不必要的数据复制，从而降低效率，并且过长的链式命令可能难以阅读和维护。为了避免这些问题，可以将操作分解为多个步骤，并对每个步骤显式地进行赋值。在对`DataFrame`的子集进行赋值时，使用`loc`或`iloc`进行索引。
```
import pandas as pd
import numpy as np

# 创建一个示例 DataFrame
np.random.seed(0)
df = pd.DataFrame(np.random.randn(10, 2), columns=['A', 'B'])

# 如我们需要修改列 B 的值，但只在列 A 的值大于 0 的情况下
# 不推荐的链式赋值
# df[df['A'] > 0]['B'] = 1  # 这样做可能导致 SettingWithCopyWarning

# 推荐的做法
df.loc[df['A'] > 0, 'B'] = 1

# 显示修改后的 DataFrame
print(df)
```

# 减少不必要的数据复制
数据复制不仅消耗内存，还可能导致代码运行缓慢，并增加出错的风险。减少不必要的数据复制是提高效率和性能的关键。在可能的情况下，为了提高效率，应优先考虑就地操作，如使用`inplace=True`参数，可以直接在原始`DataFrame`上修改数据而不创建副本。理解`Pandas`中视图和副本的区别也很重要，尽量操作视图以避免不必要的复制。如果确实需要副本，应使用`.copy()`方法来创建一个明确的副本，这有助于避免对原始数据的意外修改。
```
import pandas as pd
import numpy as np

# 创建一个示例 DataFrame
np.random.seed(0)
df = pd.DataFrame(np.random.randn(10, 2), columns=['A', 'B'])
df_copy = df.copy()  # 创建 df 的副本
print(df_copy)

# 准备一个新值数组，用于更新 DataFrame
new_values = np.random.rand(10)

# 错误的做法：创建不必要的副本（已注释）
# df_filtered = df[df['A'] > 0]
# df_filtered['B'] = new_values


# 正确的做法：避免不必要的副本
df.loc[df['A'] > 0, 'B'] = new_values[df['A'] > 0]

# 显示修改后的 DataFrame
print(df)
```

# 谨慎使用apply和map
尽管`apply`和`map`很强大，但它们不总是最高效的选择。尽可能使用向量化方法。需要谨慎，以确保代码的效率和性能。虽然这些函数提供了很大的灵活性，但不当使用可能会导致性能问题。`apply`函数虽然提供了处理每一行或列的强大灵活性，但在内部进行循环处理，可能会比`Pandas`的内置向量化函数运行得慢。对于`map`函数，它通常用于`Series`数据，将指定函数应用于每个元素，但在可能的情况下，应考虑使用`replace`或其他向量化方法，这些方法通常更为高效。如果需要对`DataFrame`的每个元素进行操作，可以使用`applymap`。
```
import pandas as pd
import numpy as np

# 创建一个示例 DataFrame
df = pd.DataFrame({'column': np.random.randint(1, 10, size=5)})

# 定义一个复杂的计算函数
def complex_calculation(x):
    return x * x - x + 2

# 使用 apply 应用函数
df['apply_result'] = df['column'].apply(complex_calculation)

# 使用向量化操作
df['vectorized_result'] = df['column'] * df['column'] - df['column'] + 2

# 显示 DataFrame 结果
print(df)
```

### 布尔掩码
对于耗时量大的`apply`方法，可以考虑使用布尔掩码的方式进行判断，提升效率非常明显
```
df['TempTradingTime'] = pd.to_datetime(df['TradingTime'])

# Create boolean masks for each condition
mask_1 = (df['TempTradingTime'].dt.time >= pd.to_datetime('09:15:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('09:25:00').time())
mask_2 = (df['TempTradingTime'].dt.time >= pd.to_datetime('09:25:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('09:30:00').time())
mask_3 = ((df['TempTradingTime'].dt.time >= pd.to_datetime('09:30:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('11:30:00').time())) | \
            ((df['TempTradingTime'].dt.time >= pd.to_datetime('13:00:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('14:57:00').time()))
mask_4 = (df['TempTradingTime'].dt.time >= pd.to_datetime('11:30:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('13:00:00').time())
mask_5 = (df['TempTradingTime'].dt.time >= pd.to_datetime('14:57:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('15:00:00').time())

# Use numpy.select to assign values based on masks
df['TradingPhaseCode'] = np.select(
    [mask_1, mask_2, mask_3, mask_4, mask_5],
    [1, 2, 3, 4, 5],
    default=-1
)
```
例子2
```
df['TempTradingTime'] = pd.to_datetime(df['TradingTime'], format='%Y-%m-%d %H:%M:%S.%f')
# Check if TradingTime is between 9:25 and 9:30
mask = (df['TempTradingTime'].dt.time >= pd.to_datetime('09:25:00').time()) & (df['TempTradingTime'].dt.time < pd.to_datetime('09:30:00').time())

# Create a temporary TradingPhaseCode column
df['TempTradingPhaseCode'] = '2'  # Set to '2' for times between 9:25 and 9:30

# For times outside 9:25-9:30, use the existing logic
if market == 'SH':
    df.loc[~mask, 'TempTradingPhaseCode'] = df.loc[~mask, 'TradeStatus'].map(sh_phase_mapping).fillna('-1')
else:  # For 'SZ' market
    df.loc[~mask, 'TempTradingPhaseCode'] = df.loc[~mask, 'SecurityPhaseTag'].apply(lambda x: sz_phase_mapping.get(x[0], '-1'))
# Assign the temporary column to TradingPhaseCode
df['TradingPhaseCode'] = df['TempTradingPhaseCode']
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

# 分块处理大型数据集
对于大型数据集，考虑分块处理而不是一次性加载整个数据集。由于内存限制，直接加载整个数据集可能不可行或效率低下。这种情况下，分块处理大型数据集是一种有效的解决方案。可以通过分块读取并逐块计算某列的总和，最后将所有块的结果累加。这种分块处理方法使处理大型数据集变得可行，特别是在有限的内存资源下，能有效提高数据处理的效率和可行性。
```
import pandas as pd
import numpy as np

total_sum = 0
chunksize = 10  # 假设每块包含 10000 行

# 设置随机数据的参数
num_rows = 1000  # 数据行数
column_name = 'target_column'  # 列名

# 创建包含随机数据的 DataFrame
# 数据是随机的浮点数，可以根据需要调整数据类型和生成逻辑
df = pd.DataFrame({column_name: np.random.rand(num_rows)})

# 写入 CSV 文件
csv_file = 'large_dataset.csv'
df.to_csv(csv_file, index=False)


# 分块读取
for chunk in pd.read_csv('large_dataset.csv', chunksize=chunksize):
    total_sum += chunk['target_column'].sum()

print(total_sum)
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

# 优化数据存储
对于多进程并行产生的数据，需要将其写入到单个文件中，考虑构建数据队列和监听进程，对多进程产生的数据，导入到数据队列中，然后再由数据队列统一写入文件中。
```
import multiprocessing as mp
import time

fn = 'c:/temp/temp.txt'

def worker(arg, q):
    '''stupidly simulates long running process'''
    start = time.clock()
    s = 'this is a test' 
    txt = s
    for i in range(200000):
        txt += s 
    done = time.clock() - start
    with open(fn, 'rb') as f:
        size = len(f.read())
    res = 'Process' + str(arg), str(size), done
    q.put(res)
    return res

def listener(q):
    '''listens for messages on the q, writes to file. '''

    with open(fn, 'w') as f:
        while 1:
            m = q.get()
            if m == 'kill':
                f.write('killed')
                break
            f.write(str(m) + '\n')
            f.flush()

def main():
    #must use Manager queue here, or will not work
    manager = mp.Manager()
    q = manager.Queue()    
    pool = mp.Pool(mp.cpu_count() + 2)

    #put listener to work first
    watcher = pool.apply_async(listener, (q,))

    #fire off workers
    jobs = []
    for i in range(80):
        job = pool.apply_async(worker, (i, q))
        jobs.append(job)

    # collect results from the workers through the pool result queue
    for job in jobs: 
        job.get()

    #now we are done, kill the listener
    q.put('kill')
    pool.close()
    pool.join()

if __name__ == "__main__":
   main()
```


