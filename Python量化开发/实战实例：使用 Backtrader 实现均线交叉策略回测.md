## ✅ 实战主题：使用 Backtrader 实现均线交叉策略回测

---

## 🧰 实战使用库

| 库名           | 作用          |
| ------------ | ----------- |
| `backtrader` | 回测框架、策略开发核心 |
| `pandas`     | 数据读取和处理     |
| `matplotlib` | 回测结果可视化     |

---

## 🪜 一、安装并准备环境

```bash
pip install backtrader pandas matplotlib
```

---

## 🧩 二、策略原理说明：双均线交叉策略（MA Cross）

> 简单策略：短期均线上穿长期均线时买入（“金叉”），下穿时卖出（“死叉”）

---

## 🧱 三、完整项目结构

```
ma_strategy_backtest/
├── main.py                  # 主程序：运行回测
├── strategy_ma_cross.py     # 策略类定义
├── data/
│   └── 000001.SZ.csv        # 股票历史数据
└── result/
    └── backtest_result.png  # 回测图输出
```

---

## 📜 四、策略定义：strategy\_ma\_cross.py

```python
import backtrader as bt

class SmaCross(bt.Strategy):
    params = dict(short=10, long=30)

    def __init__(self):
        sma1 = bt.ind.SMA(period=self.p.short)
        sma2 = bt.ind.SMA(period=self.p.long)
        self.crossover = bt.ind.CrossOver(sma1, sma2)

    def next(self):
        if not self.position:  # 没有持仓
            if self.crossover > 0:
                self.buy()
        elif self.crossover < 0:
            self.close()
```

---

## 🏁 五、主程序运行：main.py

```python
import backtrader as bt
import pandas as pd
from strategy_ma_cross import SmaCross

# 加载数据
df = pd.read_csv('data/000001.SZ.csv', parse_dates=['trade_date'])
df.set_index('trade_date', inplace=True)
df.rename(columns={'open': 'open', 'high': 'high', 'low': 'low', 'close': 'close', 'vol': 'volume'}, inplace=True)

# 转换为 Backtrader 数据格式
class PandasData(bt.feeds.PandasData):
    params = (
        ('datetime', None),
        ('open', 'open'),
        ('high', 'high'),
        ('low', 'low'),
        ('close', 'close'),
        ('volume', 'volume'),
        ('openinterest', -1),
    )

data = PandasData(dataname=df)

# 回测引擎 Cerebro
cerebro = bt.Cerebro()
cerebro.addstrategy(SmaCross)
cerebro.adddata(data)
cerebro.broker.set_cash(100000)
cerebro.broker.setcommission(commission=0.001)  # 0.1%手续费

# 运行回测
print('初始资金: %.2f' % cerebro.broker.getvalue())
cerebro.run()
print('回测结束资金: %.2f' % cerebro.broker.getvalue())

# 绘图
cerebro.plot(style='candlestick')
```

---

## 📊 六、绩效指标扩展（可选）

可接入 `bt.analyzers` 添加回测绩效评估：

```python
cerebro.addanalyzer(bt.analyzers.SharpeRatio, _name='sharpe')
cerebro.addanalyzer(bt.analyzers.DrawDown, _name='drawdown')
result = cerebro.run()
sharpe = result[0].analyzers.sharpe.get_analysis()
drawdown = result[0].analyzers.drawdown.get_analysis()

print("夏普比率：", sharpe)
print("最大回撤：", drawdown['max']['drawdown'])
```

---

## 🎁 七、支持多策略 & 多股票拓展建议

* 使用参数优化（`optstrategy`）进行网格搜索调参
* 同时加载多个数据（多标的策略）
* 添加自定义指标（如 ATR 止损、布林带）
* 写入交易日志并保存图像结果

---

## 📘 八、小结

| 模块   | 工具/方法           | 说明           |
| ---- | --------------- | ------------ |
| 数据准备 | pandas + CSV    | 加载本地历史行情     |
| 回测框架 | backtrader      | 多策略、多因子回测能力强 |
| 策略模块 | 自定义 Strategy 类  | 编写交易逻辑       |
| 结果评估 | analyzers, plot | 输出收益、回撤、图形分析 |
| 拓展性  | 多策略、多标的         | 实现完整量化交易框架   |

---

### ✅ 运行效果截图（可选）

可以将 `cerebro.plot()` 绘制的策略资金图保存为图片：

```python
fig = cerebro.plot(style='candlestick')[0][0]
fig.savefig('result/backtest_result.png')
```

---

### 🎯 下一步建议

* 封装为 CLI 工具支持命令行运行
* 用 Jupyter Notebook 创建“策略研究报告模板”
* 集成 PyFolio 或 QuantStats 输出 HTML 报告
* 部署到 Web Dashboard（如 Streamlit）进行实时展示
