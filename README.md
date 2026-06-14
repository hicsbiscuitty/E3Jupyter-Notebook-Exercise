# E3 Jupyter-Notebook-Exercise

**实验环境准备**

启动 Jupyter Notebook

新建 Notebook：点击右上角 **New > Python 3 (ipykernel)** 新建一个 Notebook 文档。

---

**1. 选择排序算法实现**

**文件：** `Simple_Python_Procedure.ipynb`
def test_interactive():
    user_input = input("请输入一组数字（用空格分隔，例如：5 2 9 1）：")
    
    try:
        data = [int(x) for x in user_input.split()]
        print(f"你输入的数据: {data}")
        result = selection_sort(data)
        print(f"排序后的结果: {result}")
    except ValueError:
        print("错误：请输入有效的整数数字！")

test_interactive()

---

**2. 财富世界500强数据分析**

**文件：** `Fortune500_Analyse.ipynb`

1. 从 https://www.jianguoyun.com/p/DabvAJEQ7JmuCRjI1LwEIAA 下载数据集文件 `fortune500.csv`

2. 导入数据分析所需的工具库，并设置绘图环境：

```python
%matplotlib inline
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 加载数据集
df = pd.read_csv('fortune500.csv')
```

3. 查看数据集的头部和尾部数据：

```python
# 查看前五行
df.head()

# 查看后五行
df.tail()

# 重命名属性列以便于访问
df.columns = ['year', 'rank', 'company', 'revenue', 'profit']

# 检查记录总数
print("总记录数：", len(df))

# 检查各属性列的数据类型
print(df.dtypes)
```

---

***4. 清洗非数字字符***

属性列 `profit` 的预期类型应为浮点数（float），但在基础检查中其类型显示为 object。这表明该列可能包含非数字的异常值。

***4.1 查找非数字记录***

使用正则表达式筛选不包含数字、小数点或负号的记录。

```python
non_numeric_profits = df.profit.str.contains('[^0-9.-]')
df.loc[non_numeric_profits].head()
```

***4.2 统计数量***

```python
print("非数字记录总数：", len(df.profit[non_numeric_profits]))
```

***4.3 分析分布***

使用直方图查看这些异常记录在年份上的分布。

```python
bin_sizes, _, _ = plt.hist(
    df.year[non_numeric_profits],
    bins=range(1955, 2006)
)
```

***4.4 剔除异常数据***

由于单年份异常记录占比较低（均低于 4%），决定剔除这些数据，并将该列转换为数值类型。

```python
df = df.loc[~non_numeric_profits]
df.profit = df.profit.apply(pd.to_numeric)

# 重新确认转换结果
print("清洗后记录总数：", len(df))
print(df.dtypes)
```

---

***5. 数据可视化分析***

***5.1 计算年度平均值***

```python
group_by_year = df.loc[
    :,
    ['year', 'revenue', 'profit']
].groupby('year')

avgs = group_by_year.mean()

x = avgs.index
y1 = avgs.profit
y2 = avgs.revenue

# 定义绘图辅助函数
def plot(x, y, ax, title, y_label):
    ax.set_title(title)
    ax.set_ylabel(y_label)
    ax.plot(x, y)
    ax.margins(x=0, y=0)
```

***5.2 绘制利润与收入趋势图***

利润趋势图：

```python
fig, ax = plt.subplots()

plot(
    x,
    y1,
    ax,
    'Increase in mean Fortune 500 company profits from 1955 to 2005',
    'Profit (millions)'
)
```

收入趋势图：

```python
fig, ax = plt.subplots()

plot(
    x,
    y2,
    ax,
    'Increase in mean Fortune 500 company revenues from 1955 to 2005',
    'Revenue (millions)'
)
```

***5.3 带有标准差区间趋势图***

展示不同公司之间在利润与收入上的波动差异：

```python
def plot_with_std(
    x,
    y,
    stds,
    ax,
    title,
    y_label
):
    ax.fill_between(
        x,
        y - stds,
        y + stds,
        alpha=0.2
    )
    plot(
        x,
        y,
        ax,
        title,
        y_label
    )

fig, (ax1, ax2) = plt.subplots(
    ncols=2
)

title = (
    'Increase in mean and std Fortune 500 company %s '
    'from 1955 to 2005'
)

stds1 = group_by_year.std().profit.values
stds2 = group_by_year.std().revenue.values

plot_with_std(
    x,
    y1.values,
    stds1,
    ax1,
    title % 'profits',
    'Profit (millions)'
)

plot_with_std(
    x,
    y2.values,
    stds2,
    ax2,
    title % 'revenues',
    'Revenue (millions)'
)

fig.set_size_inches(14, 4)
fig.tight_layout()
```

---

**3. 扩展截图**

1. 打开命令行终端（在 Anaconda 环境下），依次执行以下四条命令：

```bash
pip install jupyter_contrib_nbextensions

jupyter contrib nbextension install --user

pip install jupyter_nbextensions_configurator

jupyter nbextensions_configurator enable --user
```

2. 等待命令执行完毕后，重新启动 Jupyter Notebook。

3. 在主页点击新增的 **Nbextensions** 标签页。

4. 在列表中找到并勾选 **Hinterland** 插件。

5. 配置完成后，在编辑模式下，代码补全提示将在输入或按下 **Tab** 键时自动触发。
