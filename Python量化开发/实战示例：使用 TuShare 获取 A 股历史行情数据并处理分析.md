## ✅ 实战示例：使用 TuShare 获取 A 股历史行情数据并处理分析

---

### 🧰 使用库：

* `tushare`：获取 A 股行情、财报等数据（需注册 Token）
* `pandas`：数据处理
* `numpy`：收益率计算
* `matplotlib`：可视化
* （可选）`os`：数据存储路径管理

---

## 🧩 第一步：准备工作（注册 TuShare、安装包）

1. 注册并获取 token：[https://tushare.pro/register](https://tushare.pro/register)
2. 安装库：

```bash
pip install tushare
```

---

## 📦 第二步：下载股票日线数据并保存为 CSV

```python
import tushare as ts
import pandas as pd
import os

# 设置 TuShare Token
ts.set_token('your_token_here')
pro = ts.pro_api()

# 参数设置
ts_code = '000001.SZ'  # 平安银行
start_date = '20240101'
end_date = '20240601'

# 下载日线行情数据
df = pro.daily(ts_code=ts_code, start_date=start_date, end_date=end_date)

# 数据清洗与格式调整
df['trade_date'] = pd.to_datetime(df['trade_date'])
df = df.sort_values('trade_date')
df.set_index('trade_date', inplace=True)

# 保存为本地CSV
os.makedirs('data', exist_ok=True)
csv_path = f'data/{ts_code}_daily.csv'
df.to_csv(csv_path)
print(f"数据保存至：{csv_path}")
```

---

## 🧹 第三步：数据预处理

```python
# 重新读取数据
df = pd.read_csv(csv_path, index_col='trade_date', parse_dates=True)

# 查看空值情况
print("缺失值统计：\n", df.isnull().sum())

# 删除缺失行
df.dropna(inplace=True)

# 添加对数收益率列
import numpy as np
df['log_return'] = np.log(df['close'] / df['close'].shift(1))

# 删除前1个NaN
df.dropna(inplace=True)
```

---

## 🧪 第四步：数据可视化与分析

```python
import matplotlib.pyplot as plt

# 收盘价走势图
plt.figure(figsize=(12,5))
plt.plot(df.index, df['close'], label='收盘价')
plt.title(f"{ts_code} 日线收盘价")
plt.xlabel('日期')
plt.ylabel('价格')
plt.grid(True)
plt.legend()
plt.tight_layout()
plt.show()

# 对数收益率走势
plt.figure(figsize=(12,3))
plt.plot(df.index, df['log_return'], label='对数收益率', color='orange')
plt.axhline(0, linestyle='--', color='gray')
plt.title(f"{ts_code} 日对数收益率")
plt.grid(True)
plt.tight_layout()
plt.show()
```

---

## 💾 第五步：长期存储与多股票管理（拓展）

### 多股票批量抓取：

```python
stock_list = ['000001.SZ', '600519.SH', '002415.SZ']  # 平安银行、茅台、海康威视

for code in stock_list:
    df = pro.daily(ts_code=code, start_date='20240101', end_date='20240601')
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    df = df.sort_values('trade_date')
    df.set_index('trade_date', inplace=True)
    df.to_csv(f'data/{code}_daily.csv')
```

### 保存为 Parquet（二进制压缩格式）：

```python
df.to_parquet('data/000001.SZ_daily.parquet')
```

---

## 📊 第六步：数据处理总结报告（打印分析结果）

```python
print("数据概况：")
print(df.describe())

print("\n最大涨幅（对数收益）:", df['log_return'].max())
print("最大跌幅（对数收益）:", df['log_return'].min())
print("平均日收益:", df['log_return'].mean())
print("波动率:", df['log_return'].std())
```

---

## 🔚 小结

| 步骤 | 内容             | 工具                     |
| -- | -------------- | ---------------------- |
| 1  | 数据源获取（TuShare） | `tushare.pro_api()`    |
| 2  | 数据清洗与时间格式处理    | `pandas`               |
| 3  | 收益率计算与缺失值处理    | `numpy`, `pandas`      |
| 4  | 可视化分析          | `matplotlib`           |
| 5  | 批量处理 & 存储优化    | `os`, `parquet`        |
| 6  | 报告与结果输出        | `df.describe()`, print |

