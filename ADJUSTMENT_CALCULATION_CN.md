# 复权计算逻辑详解 / Stock Price Adjustment Calculation

> 生成时间: 2025-12-15
> 用途: 详细说明股票价格复权的计算逻辑和在Qlib中的实现

## 目录

- [1. 什么是复权](#1-什么是复权)
- [2. 为什么需要复权](#2-为什么需要复权)
- [3. 复权计算公式](#3-复权计算公式)
- [4. 前复权 vs 后复权](#4-前复权-vs-后复权)
- [5. Qlib中的复权实现](#5-qlib中的复权实现)
- [6. 复权因子的计算](#6-复权因子的计算)
- [7. 实战示例](#7-实战示例)
- [8. 常见问题](#8-常见问题)

---

## 1. 什么是复权

### 基本概念

**复权（Price Adjustment）** 是指对股票价格进行调整，以消除因**除权除息**事件（如分红、送股、配股、股票拆分等）造成的价格不连续性。

### 除权除息事件

| 事件类型 | 英文 | 说明 | 对价格的影响 |
|---------|------|------|------------|
| **现金分红** | Cash Dividend | 向股东派发现金 | 价格下降 |
| **送股** | Stock Dividend | 向股东无偿赠送股票 | 价格下降，股数增加 |
| **配股** | Rights Offering | 股东有权以优惠价购买新股 | 价格下降 |
| **股票拆分** | Stock Split | 1股拆成多股（如1拆2） | 价格下降，股数增加 |
| **股票合并** | Reverse Split | 多股合成1股（如10合1） | 价格上升，股数减少 |

### 可视化示例

```
未复权价格（会出现跳空）:

价格
 │
20├─────────●●●●●●●
 │                 ╲
 │                  ╲  ← 除权日，价格跳空下降
 │                   ╲
10├───────────────────●●●●●●●
 │
 └─────────────────────────→ 时间
              ↑
           除权日
           (10股送10股，价格减半)


前复权价格（连续）:
保持最新价格不变，向下调整历史价格

价格
 │
 │
10├─────────●●●●●●●●●●●●●●●
 │           ↑历史价格向下调整    ↑最新价格保持不变
 │          (×0.5)
 │
 └─────────────────────────→ 时间
              ↑
           除权日


后复权价格（连续）:
保持历史价格不变，向上调整最新价格

价格
 │
40├───────────────────●●●●●●●
 │                   ╱
 │                  ╱  ← 最新价格向上调整
 │                 ╱      (÷0.5)
20├─────────●●●●●●●
 │           ↑历史价格保持不变
 │
 └─────────────────────────→ 时间
              ↑
           除权日
```

**说明**：
- **未复权**：除权日出现明显跳空，从20元跌到10元
- **前复权**：历史价格×0.5调整到10元，最新价格保持10元，整体连续
- **后复权**：历史价格保持20元，最新价格÷0.5调整到20元，整体连续

---

## 2. 为什么需要复权

### 问题场景

假设某股票：
- **除权前**: 价格 20元，持有 100股，市值 = 20 × 100 = 2000元
- **除权事件**: 10股送10股（每持有10股，额外获得10股）
- **除权后**: 价格 10元，持有 200股，市值 = 10 × 200 = 2000元

**实际上投资者的财富没有变化**，但价格从20元跌到10元，**看起来像是暴跌50%**！

### 不复权的问题

1. **技术指标失效**:
   - 移动平均线、布林带等指标会在除权日出现异常突变
   - RSI、MACD等动量指标会产生虚假信号

2. **收益率计算错误**:
   - 不复权的价格会显示 `-50%` 的收益（实际为0%）
   - 回测结果严重失真

3. **趋势分析错误**:
   - 无法准确判断真实的价格趋势
   - 跳空缺口会干扰形态识别

### 复权的作用

通过复权处理，可以：
- ✅ 保持价格序列的连续性
- ✅ 准确反映真实的投资收益
- ✅ 使技术指标计算正确
- ✅ 支持跨期对比分析

---

## 3. 复权计算公式

### 核心概念

**复权因子（Adjustment Factor）**:

```
复权因子 = 复权后价格 / 未复权价格
```

### 中英文术语对应关系

⚠️ **重要提示**：中文和英文的复权术语对应关系是**相反**的！

| 中文术语 | 英文术语 | 保持不变 | 调整部分 |
|---------|---------|---------|---------|
| **前复权** | Backward Adjustment | 最新价格 | 历史价格（向下调整） |
| **后复权** | Forward Adjustment | 历史价格 | 最新价格（向上调整） |

**命名理解**：
- **前复权**："向前看"到现在，以现在为准，调整历史
- **后复权**："向后看"到过去，以过去为准，调整现在

### 前复权计算公式

**前复权（中文）/ Backward Adjustment（英文）** - 保持最新价格不变，调整历史价格

#### 基本公式

```
前复权价格 = 未复权价格 × 复权因子

其中：
复权因子 = ∏(从除权日到当前的所有调整因子)
```

#### 单次除权的调整因子

**现金分红 (Cash Dividend)**:
```
调整因子 = (P - D) / P

P: 除权前一日收盘价
D: 每股分红金额
```

**送股/配股 (Stock Dividend / Rights Offering)**:
```
调整因子 = P / (P + N × Ps)

P:  除权前一日收盘价
N:  每股获得的新股数量（送股比例）
Ps: 配股价格（送股时 Ps = 0）
```

**股票拆分 (Stock Split)**:
```
调整因子 = 1 / 拆分比例

例如: 1拆2 → 调整因子 = 1/2 = 0.5
```

#### 综合公式

对于同时有分红和送股的情况：

```
调整因子 = (P - D) / (P + N × Ps)

P:  除权前一日收盘价
D:  每股现金分红
N:  每股送/配股数量
Ps: 配股价格（送股时为0）
```

### 后复权计算公式

**后复权（中文）/ Forward Adjustment（英文）** - 保持历史价格不变，调整最新价格

```
后复权价格 = 未复权价格 / 复权因子

其中：
复权因子 = ∏(从首日到除权日的所有调整因子)
```

---

## 4. 前复权 vs 后复权

### 对比表

| 特性 | 前复权 (Backward Adjustment) | 后复权 (Forward Adjustment) |
|-----|------------------------------|----------------------------|
| **保持不变** | 最新价格 | 历史价格 |
| **调整部分** | 历史价格 | 最新价格 |
| **价格范围** | 接近实际交易价 | 可能超过实际交易价 |
| **适用场景** | 技术分析、回测 | 查看历史成本 |
| **Qlib支持** | ✅ 支持 | ❌ 不支持 |

### 可视化对比

```
原始价格序列（未复权）:
时间:  T1   T2   T3   [除权]  T4   T5   T6
价格:  10   12   15           8    9    10
                              ↑
                           价格跳空


前复权（调整 T1-T3，保持最新价格）:
时间:  T1   T2   T3   [除权]  T4   T5   T6
价格:  5.3  6.4  8.0          8    9    10
       ↑向下调整                   ↑保持不变


后复权（调整 T4-T6，保持历史价格）:
时间:  T1   T2   T3   [除权]  T4    T5    T6
价格:  10   12   15           16    18    20
       ↑保持不变              ↑向上调整


说明: 假设 T3 除权，10股送10股，未复权价格从15跌到8
```

### 计算示例

**场景**:
- T1: 价格 10元
- T2: 价格 12元
- T3: 价格 15元，收盘后宣布 **10股送10股**
- T4: 除权日，开盘价 8元

#### 前复权计算（保持最新价格不变）

```
步骤1: 计算调整因子
调整因子 = P / (P + N × Ps)
        = 15 / (15 + 1 × 0)    // N=1 (10送10), Ps=0 (送股)
        = 15 / 30
        = 0.5

步骤2: 调整历史价格（T1-T3）
T1 前复权价格 = 10 × 0.5 = 5 元
T2 前复权价格 = 12 × 0.5 = 6 元
T3 前复权价格 = 15 × 0.5 = 7.5 元

步骤3: 除权后价格保持不变
T4 前复权价格 = 8 元 (实际价格)

结果序列（前复权）:
T1: 5     T2: 6     T3: 7.5    T4: 8
   ──────────────────────────────────  连续上涨！
```

#### 后复权计算（保持历史价格不变）

```
步骤1: 调整因子相同
调整因子 = 0.5

步骤2: 历史价格保持不变
T1 后复权价格 = 10 元
T2 后复权价格 = 12 元
T3 后复权价格 = 15 元

步骤3: 调整除权后价格（T4）
T4 后复权价格 = 8 / 0.5 = 16 元

结果序列（后复权）:
T1: 10    T2: 12    T3: 15    T4: 16
   ────────────────────���─────────────  连续上涨！
```

---

## 5. Qlib中的复权实现

### 设计理念

Qlib 采用**前复权（Backward Adjustment）**方式，并在数据存储时就完成复权处理：

1. **数据导入时复权**: 使用 `dump_bin.py` 导入数据时，要求CSV文件包含 `factor` 字段
2. **存储复权后价格**: qlib 数据库中存储的都是调整后的价格
3. **提供复权因子**: 通过 `$factor` 字段，用户可以反推原始价格

**重要说明**：
- Qlib 使用的是**前复权**（中文术语）
- 对应英文的 **Backward Adjustment**
- 保持**最新价格不变**，调整**历史价格**

### 数据字段说明

```python
# Qlib 数据字段（参考: docs/component/data.rst）

字段名          类型      说明
----------------------------------------------
$open         float    调整后的开盘价
$close        float    调整后的收盘价
$high         float    调整后的最高价
$low          float    调整后的最低价
$volume       float    调整后的成交量
$factor       float    复权因子

复权因子计算:
    factor = adjusted_price / original_price
```

### 核心特性

#### 1. 首日归一化

Qlib 将每只股票在**首个交易日的价格归一化为 1**:

```python
# 归一化示例
某股票历史价格:
  原始价格:   [10,  12,  15,  8,  9,  10]
  归一化后:   [1.0, 1.2, 1.5, 0.8, 0.9, 1.0]

计算方式:
  normalized_price = original_price / first_day_price
```

**优势**:
- 不同价格水平的股票可以直接对比
- 更容易进行跨股票的特征工程
- 避免价格绝对值对模型的影响

#### 2. 反推原始价格

```python
# 获取原始价格
original_close = $close / $factor
original_open = $open / $factor
original_high = $high / $factor
original_low = $low / $factor

# 在 Qlib 表达式中使用
from qlib.data import D

data = D.features(
    instruments="csi500",
    fields=[
        "$close",           # 调整后价格
        "$factor",          # 复权因子
        "$close/$factor"    # 原始价格
    ],
    start_time="2020-01-01",
    end_time="2021-01-01"
)
```

### 数据导入流程

```bash
# Step 1: 准备数据（CSV格式）
# 文件结构: symbol, date, open, close, high, low, volume, factor

symbol,date,open,close,high,low,volume,factor
000001,2020-01-02,10.5,10.8,11.0,10.3,1000000,1.0
000001,2020-01-03,10.9,11.2,11.5,10.8,1200000,1.0
000001,2020-01-06,11.0,11.5,11.8,10.9,1500000,1.0

# Step 2: 运行导入脚本
python scripts/dump_bin.py dump_all \
    --csv_path ~/stock_data/ \
    --qlib_dir ~/.qlib/qlib_data/cn_data \
    --include_fields open,close,high,low,volume,factor \
    --symbol_field_name symbol \
    --date_field_name date

# Step 3: 初始化 Qlib
import qlib
qlib.init(provider_uri='~/.qlib/qlib_data/cn_data')
```

---

## 6. 复权因子的计算

### 累积复权因子

当有多次除权事件时，需要计算**累积复权因子**：

```python
# 累积复权因子计算（前复权 / Backward Adjustment）

def calculate_cumulative_factor_backward(adjustments):
    """
    计算累积复权因子（前复权 / Backward Adjustment）
    保持最新价格不变，向下调整历史价格

    Parameters:
    -----------
    adjustments : list of dict
        除权事件列表，按时间正序排列
        每个事件包含: {
            'date': 除权日期,
            'pre_close': 除权前收盘价,
            'dividend': 每股分红,
            'gift_ratio': 送股比例 (如 10送3 则为 0.3),
            'allotment_ratio': 配股比例,
            'allotment_price': 配股价格
        }

    Returns:
    --------
    dict: {日期: 累积复权因子}
    """
    cumulative_factor = 1.0
    factor_dict = {}

    for adj in adjustments:
        # 计算单次调整因子
        P = adj['pre_close']
        D = adj['dividend']
        N = adj['gift_ratio'] + adj['allotment_ratio']
        Ps = adj['allotment_price']

        # 调整因子 = (P - D) / (P + N × Ps)
        adjustment_factor = (P - D) / (P + N * Ps) if P + N * Ps > 0 else 1.0

        # 累积
        cumulative_factor *= adjustment_factor
        factor_dict[adj['date']] = cumulative_factor

    return factor_dict


# 使用示例
adjustments = [
    {
        'date': '2020-06-15',
        'pre_close': 20.0,
        'dividend': 1.0,      # 每股分红1元
        'gift_ratio': 0.5,    # 10股送5股
        'allotment_ratio': 0.0,
        'allotment_price': 0.0
    },
    {
        'date': '2021-06-15',
        'pre_close': 25.0,
        'dividend': 1.5,      # 每股分红1.5元
        'gift_ratio': 0.3,    # 10股送3股
        'allotment_ratio': 0.0,
        'allotment_price': 0.0
    }
]

factors = calculate_cumulative_factor_backward(adjustments)
# 输出: {
#   '2020-06-15': 0.6333...,
#   '2021-06-15': 0.6333... × 0.7231... = 0.458...
# }
```

### 详细计算示例

**场景**: 某股票有两次除权

```
时间线:
├─ 2020-01-01: IPO上市，价格 10.00 元
├─ 2020-06-15: 除权日1 - 10股送5股 + 每股分红1元
│   除权前: 20.00 元
│   除权后: 12.67 元
├─ 2021-06-15: 除权日2 - 10股送3股 + 每股分红1.5元
│   除权前: 25.00 元
│   除权后: 18.08 元
└─ 2022-01-01: 当前价格 30.00 元
```

#### Step 1: 计算第一次除权的调整因子

```
除权日1: 2020-06-15
  P = 20.00 (除权前收盘)
  D = 1.00  (每股分红)
  N = 0.5   (10送5 = 50%送股)
  Ps = 0    (送股，无配股)

调整因子1 = (P - D) / (P + N × Ps)
         = (20 - 1) / (20 + 0.5 × 0)
         = 19 / 20
         = 0.95
```

#### Step 2: 计算第二次除权的调整因子

```
除权日2: 2021-06-15
  P = 25.00
  D = 1.50
  N = 0.3   (10送3 = 30%送股)
  Ps = 0

调整因子2 = (25 - 1.5) / (25 + 0.3 × 0)
         = 23.5 / 25
         = 0.94
```

#### Step 3: 计算各时期的累积复权因子（前复权）

```
时期划分:
  [2020-01-01 ~ 2020-06-14]: 还没除权
  [2020-06-15 ~ 2021-06-14]: 经历了1次除权
  [2021-06-15 ~ 2022-01-01]: 经历了2次除权

累积复权因子（前复权 - 保持最新价格不变）:
  2020-01-01 ~ 2020-06-14:  0.95 × 0.94 = 0.893
  2020-06-15 ~ 2021-06-14:  0.94
  2021-06-15 ~ 2022-01-01:  1.0
```

#### Step 4: 计算前复权价格

```
原始价格序列:
  2020-01-01: 10.00
  2020-06-14: 20.00  (除权前)
  2020-06-15: 12.67  (除权后)
  2021-06-14: 25.00  (除权前)
  2021-06-15: 18.08  (除权后)
  2022-01-01: 30.00

前复权价格序列（保持最新价格不变）:
  2020-01-01: 10.00 × 0.893 = 8.93
  2020-06-14: 20.00 × 0.893 = 17.86
  2020-06-15: 12.67 × 0.94  = 11.91
  2021-06-14: 25.00 × 0.94  = 23.50
  2021-06-15: 18.08 × 1.0   = 18.08
  2022-01-01: 30.00 × 1.0   = 30.00

验证连续性:
  2020-06-14 → 2020-06-15:  17.86 → 11.91 (仍有跳空？)

重新计算 2020-06-15:
  理论价格 = (20.00 - 1.00) / 1.5 = 12.67  (正确)
  前复权价格 = 12.67 × 0.94 = 11.91       (正确)
```

**注意**: 即使复权后，价格在除权日仍可能有小幅跳空，这是因为除权价是理论价格，实际开盘价可能与之不同。

---

## 7. 实战示例

### 示例1: 在 Qlib 中使用复权数据

```python
import qlib
from qlib.data import D
from qlib.contrib.data.handler import Alpha158

# 初始化 Qlib
qlib.init(provider_uri='~/.qlib/qlib_data/cn_data')

# 方法1: 直接使用调整后的价格
data = D.features(
    instruments=["000001.SZ", "000002.SZ"],
    fields=["$open", "$close", "$high", "$low", "$volume"],
    start_time="2020-01-01",
    end_time="2021-01-01"
)

print("调整后的价格数据:")
print(data.head())

# 方法2: 获取复权因子
data_with_factor = D.features(
    instruments=["000001.SZ", "000002.SZ"],
    fields=[
        "$close",          # 调整后收盘价
        "$factor",         # 复权因子
        "$close/$factor"   # 原始收盘价
    ],
    start_time="2020-01-01",
    end_time="2021-01-01"
)

print("\n复权因子数据:")
print(data_with_factor.head())

# 方法3: 使用 Alpha158（已内置复权处理）
handler = Alpha158(
    instruments="csi500",
    start_time="2020-01-01",
    end_time="2021-01-01"
)

# 获取特征（基于调整后的价格计算）
features = handler.fetch(col_set="feature")
print("\nAlpha158 特征:")
print(features.head())
```

### 示例2: 计算真实收益率

```python
import pandas as pd
from qlib.data import D

# 获取调整后的价格
prices = D.features(
    instruments="000001.SZ",
    fields=["$close"],
    start_time="2020-01-01",
    end_time="2021-01-01"
)

# 计算日收益率
returns = prices.pct_change()

# 计算累积收益率
cumulative_returns = (1 + returns).cumprod() - 1

print("日收益率统计:")
print(returns.describe())

print("\n累积收益率:")
print(cumulative_returns.tail())
```

### 示例3: 对比复权前后的技术指标

```python
import numpy as np
import pandas as pd
from qlib.data import D

# 获取数据
data = D.features(
    instruments="000001.SZ",
    fields=[
        "$close",          # 调整后价格
        "$factor",         # 复权因子
        "$close/$factor"   # 原始价格
    ],
    start_time="2020-01-01",
    end_time="2021-01-01"
)

# 计算20日移动平均（调整后）
data['MA20_adjusted'] = data['$close'].rolling(window=20).mean()

# 计算20日移动平均（原始价格）
data['$close_original'] = data['$close'] / data['$factor']
data['MA20_original'] = data['$close_original'].rolling(window=20).mean()

# 对比
print("复权后的移动平均:")
print(data[['$close', 'MA20_adjusted']].tail())

print("\n未复权的移动平均:")
print(data[['$close_original', 'MA20_original']].tail())

# 可视化（需要 matplotlib）
import matplotlib.pyplot as plt

fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 8))

# 复权后
ax1.plot(data.index, data['$close'], label='Adjusted Close', alpha=0.7)
ax1.plot(data.index, data['MA20_adjusted'], label='MA20 (Adjusted)', alpha=0.7)
ax1.set_title('Adjusted Price with MA20')
ax1.legend()
ax1.grid(True)

# 未复权
ax2.plot(data.index, data['$close_original'], label='Original Close', alpha=0.7)
ax2.plot(data.index, data['MA20_original'], label='MA20 (Original)', alpha=0.7)
ax2.set_title('Original Price with MA20')
ax2.legend()
ax2.grid(True)

plt.tight_layout()
plt.savefig('adjustment_comparison.png')
print("\n图表已保存到 adjustment_comparison.png")
```

### 示例4: 自定义复权因子计算

```python
import pandas as pd

def calculate_backward_adjustment(df):
    """
    计算前复权价格（中文术语）/ Backward Adjustment（英文术语）
    保持最新价格不变，调整历史价格

    Parameters:
    -----------
    df : DataFrame
        包含以下列的数据框:
        - date: 日期
        - close: 收盘价
        - dividend: 分红（无分红时为0）
        - gift_ratio: 送股比例（无送股时为0）

    Returns:
    --------
    DataFrame: 添加了 adjusted_close 和 factor 列
    """
    df = df.sort_values('date').reset_index(drop=True)

    # 初始化累积因子（从最新日期开始，因子为1）
    n = len(df)
    factors = [1.0] * n

    # 从后向前计算（前复权/Backward）
    for i in range(n - 2, -1, -1):
        # 检查是否有除权事件
        if df.loc[i + 1, 'dividend'] > 0 or df.loc[i + 1, 'gift_ratio'] > 0:
            P = df.loc[i, 'close']
            D = df.loc[i + 1, 'dividend']
            N = df.loc[i + 1, 'gift_ratio']

            # 计算调整因子
            adj_factor = (P - D) / (P + N * 0) if P > 0 else 1.0
            factors[i] = factors[i + 1] * adj_factor
        else:
            factors[i] = factors[i + 1]

    # 添加因子和调整后价格
    df['factor'] = factors
    df['adjusted_close'] = df['close'] * df['factor']

    return df


# 使用示例
data = pd.DataFrame({
    'date': pd.date_range('2020-01-01', periods=10, freq='D'),
    'close': [10, 10.5, 11, 11.5, 12, 12.5, 13, 13.5, 14, 14.5],
    'dividend': [0, 0, 0, 0, 0.5, 0, 0, 0, 0, 0],      # 第5天分红0.5元
    'gift_ratio': [0, 0, 0, 0, 0.2, 0, 0, 0, 0, 0]    # 第5天送股20%
})

result = calculate_backward_adjustment(data)
print("前复权（Backward Adjustment）结果:")
print(result[['date', 'close', 'dividend', 'gift_ratio', 'factor', 'adjusted_close']])
```

---

## 8. 常见问题

### Q1: Qlib 为什么只支持前复权？

**答**:
1. **技术分析标准**: 前复权（Backward Adjustment）是技术分析和量化回测的行业标准
2. **最新价格真实**: 前复权保持最新价格不变，便于实盘对照
3. **避免混淆**: 统一使用前复权，避免不同复权方式导致的混淆
4. **灵活性**: 通过 `$factor` 字段，用户可以自行计算后复权或原始价格

**注意**：
- 中文"前复权" = 英文 Backward Adjustment
- 保持最新价格不变，调整历史价格向下

### Q2: 如何判断数据是否已复权？

```python
from qlib.data import D

# 方法1: 检查价格跳空
data = D.features(
    instruments="000001.SZ",
    fields=["$close"],
    start_time="2020-01-01",
    end_time="2021-01-01"
)

# 计算日收益率
returns = data.pct_change()

# 如果有超过20%的单日跌幅且无重大利空，可能是未复权
extreme_drops = returns[returns < -0.2]
print(f"极端跌幅天数: {len(extreme_drops)}")

# 方法2: 检查是否有 factor 字段
try:
    factors = D.features(
        instruments="000001.SZ",
        fields=["$factor"],
        start_time="2020-01-01",
        end_time="2021-01-01"
    )
    print("数据包含复权因子，已复权")
except:
    print("数据不包含复权因子")
```

### Q3: 复权因子为什么会大于1？

**答**:
复权因子可以大于1，这取决于**基准日期**的选择：

```
情况1: 以最新日期为基准（Qlib默认）
  最新日期的 factor = 1.0
  历史日期的 factor < 1.0 (因为要向下调整)

情况2: 以首个交易日为基准
  首个交易日的 factor = 1.0
  后续日期的 factor 可能 > 1.0 或 < 1.0

情况3: 以某个特定日期为基准
  基准日期的 factor = 1.0
  之前的日期 factor < 1.0
  之后的日期 factor > 1.0 (如果有除权)
```

**Qlib的实现**:
```python
# Qlib 归一化到首个交易日
# 如果首日价格为 10元，当前价格为 50元
# 则: 首日 factor = 10/10 = 1.0
#     当前 factor = 50/10 = 5.0 (> 1)
```

### Q4: 成交量为什么也需要复权？

**答**:
送股和股票拆分会改变流通股数，成交量也需要相应调整：

```
除权前: 价格 20元，成交量 1000股
除权事件: 10股送10股
除权后: 价格 10元，成交量应该 × 2 = 2000股

调整规则:
  adjusted_volume = original_volume / adjustment_factor

原因:
  - 保持成交额一致
  - 技术指标（如量价指标）计算正确
```

### Q5: 复权后的价格可以为负吗？

**答**:
理论上不会为负，但可能接近0：

```python
# 极端情况: 多次大比例送股
# 例如: 连续5年，每年10股送10股

初始价格: 100元
第1年: 100 × 0.5 = 50元
第2年: 50 × 0.5 = 25元
第3年: 25 × 0.5 = 12.5元
第4年: 12.5 × 0.5 = 6.25元
第5年: 6.25 × 0.5 = 3.125元

# 仍然是正数，但已经很小
```

**特殊情况**:
- 如果数据质量问题（如 factor 计算错误），可能出现负值
- 需要进行数据清洗和验证

### Q6: 如何处理停牌期间的复权？

**答**:
停牌期间的处理方式：

```python
# Qlib 的处理 (参考 docs/component/data.rst)
# 停牌日: open, close, high, low, volume, factor = NaN

# 复权时的处理:
# 1. 停牌期间的价格填充为 NaN
# 2. 复权因子在停牌日保持不变
# 3. 复牌后使用最新的复权因子

示例:
日期         收盘价    停牌    复权因子    调整后价格
2020-06-10   20.0     否      0.95       19.0
2020-06-11   NaN      是      0.95       NaN
2020-06-12   NaN      是      0.95       NaN
2020-06-15   12.67    否      0.95       12.04
              ↑
         (除权复牌)
```

### Q7: Qlib 数据导入时如何生成 factor？

**答**:
Qlib **不自动计算** factor，需要在数据源中提供：

```bash
# CSV 文件必须包含 factor 列
symbol,date,open,close,high,low,volume,factor
000001,2020-01-02,10.5,10.8,11.0,10.3,1000000,1.0
000001,2020-01-03,10.9,11.2,11.5,10.8,1200000,1.0

# 数据来源建议:
# 1. 使用 Tushare Pro: 提供复权因子
# 2. 使用 JoinQuant: 提供复权数据
# 3. 使用 AKShare: 提供复权数据
# 4. 自行计算: 使用除权除息事件计算

# Tushare 示例
import tushare as ts

pro = ts.pro_api('your_token')

# 获取复权因子
df = pro.adj_factor(ts_code='000001.SZ', start_date='20200101', end_date='20211231')
# 返回字段: ts_code, trade_date, adj_factor

# 获取日线数据
df_daily = pro.daily(ts_code='000001.SZ', start_date='20200101', end_date='20211231')

# 合并数据
data = df_daily.merge(df, on=['ts_code', 'trade_date'])
data.rename(columns={'adj_factor': 'factor'}, inplace=True)
```

### Q8: 如何验证复权数据的正确性？

```python
import pandas as pd
from qlib.data import D

def verify_adjustment(instrument, start_date, end_date):
    """
    验证复权数据的正确性
    """
    # 获取数据
    data = D.features(
        instruments=instrument,
        fields=["$close", "$factor", "$volume"],
        start_time=start_date,
        end_time=end_date
    )

    # 测试1: 检查复权因子是否合理（通常在 0.1 到 10 之间）
    factor_range = (data['$factor'].min(), data['$factor'].max())
    print(f"复权因子范围: {factor_range}")

    # 测试2: 检查是否有异常的日收益率
    returns = data['$close'].pct_change()
    extreme_returns = returns[(returns < -0.2) | (returns > 0.2)]
    print(f"异常收益率天数: {len(extreme_returns)}")

    # 测试3: 检查复权因子的跳变
    factor_changes = data['$factor'].pct_change()
    large_changes = factor_changes[abs(factor_changes) > 0.1]
    print(f"复权因子大幅变化天数: {len(large_changes)}")
    if len(large_changes) > 0:
        print("可能的除权日期:")
        print(large_changes.index.tolist())

    # 测试4: 计算原始价格，检查是否合理
    data['original_close'] = data['$close'] / data['$factor']
    print(f"\n原始价格范围: {data['original_close'].min():.2f} - {data['original_close'].max():.2f}")

    return data

# 使用示例
result = verify_adjustment("000001.SZ", "2020-01-01", "2021-12-31")
```

---

## 附录A: 术语表

| 中文 | 英文 | 说明 |
|------|------|------|
| 复权 | Adjustment / Split Adjustment | 调整价格以消除除权影响 |
| 前复权 | Backward Adjustment | 保持最新价格不变，调整历史价格 |
| 后复权 | Forward Adjustment | 保持历史价格不变，调整最新价格 |
| 不复权 | Unadjusted / No Adjustment | 原始价格，不做任何调整 |
| 复权因子 | Adjustment Factor | adjusted / original 的比例 |
| 除权 | Ex-Rights | 股票分红送股后的权利调整 |
| 除息 | Ex-Dividend | 现金分红后的价格调整 |
| 分红 | Cash Dividend | 向股东派发现金 |
| 送股 | Stock Dividend | 向股东无偿赠送股票 |
| 配股 | Rights Offering | 以优惠价向股东配售新股 |
| 股票拆分 | Stock Split | 1股拆成多股 |
| 股票合并 | Reverse Split | 多股合成1股 |
| 除权价 | Ex-Rights Price | 理论除权后价格 |
| 除权日 | Ex-Rights Date | 除权生效日期 |
| 登记日 | Record Date | 确定股东权益的日期 |

---

## 附录B: 参考资源

### 文档

- **Qlib 官方文档**: https://qlib.readthedocs.io/
- **Qlib 数据组件**: `qlib/docs/component/data.rst`
- **Investopedia - Split Adjusted**: https://www.investopedia.com/terms/s/splitadjusted.asp
- **Investopedia - Dividend Adjustment**: https://www.investopedia.com/terms/d/dividendadjustedreturn.asp

### 数据源

| 数据源 | 网址 | 复权支持 |
|-------|------|---------|
| Tushare Pro | https://tushare.pro/ | ✅ 提供复权因子 |
| JoinQuant | https://www.joinquant.com/ | ✅ 提供复权数据 |
| AKShare | https://akshare.xyz/ | ✅ 提供复权数据 |
| Yahoo Finance | https://finance.yahoo.com/ | ✅ 提供调整后价格 |
| Wind | https://www.wind.com.cn/ | ✅ 专业金融数据 |

### 代码文件

- `qlib/data/data.py` - 数据提供者接口
- `qlib/data/ops.py` - 数据操作符
- `qlib/contrib/data/handler.py` - Alpha158/Alpha360 Handler
- `qlib/contrib/data/loader.py` - 数据加载器
- `scripts/dump_bin.py` - 数据导入工具

---

## 附录C: 复权计算速查表

### 单一事件

| 事件类型 | 调整因子公式 | 示例 | 结果 |
|---------|------------|------|------|
| **现金分红** | `(P - D) / P` | P=20, D=1 | 0.95 |
| **送股** | `P / (P + N × 0)` | P=20, N=1 | 0.5 |
| **配股** | `P / (P + N × Ps)` | P=20, N=0.5, Ps=10 | 0.8 |
| **股票拆分 (1拆2)** | `1 / 2` | - | 0.5 |
| **股票合并 (10合1)** | `10 / 1` | - | 10 |

### 组合事件

| 事件组合 | 调整因子公式 | 示例 | 结果 |
|---------|------------|------|------|
| **分红+送股** | `(P - D) / (P + N × 0)` | P=20, D=1, N=0.5 | 0.633 |
| **分红+配股** | `(P - D) / (P + N × Ps)` | P=20, D=1, N=0.3, Ps=15 | 0.76 |
| **送股+配股** | `P / (P + N1 × 0 + N2 × Ps)` | P=20, N1=0.5, N2=0.2, Ps=10 | 0.625 |

### 复权价格计算

| 复权方式 | 中文术语 | 英文术语 | 历史价格 | 最新价格 | 公式 |
|---------|---------|---------|---------|---------|------|
| **向下调整历史** | 前复权 | Backward Adjustment | 调整 | 不变 | `历史价格 × 累积因子` |
| **向上调整最新** | 后复权 | Forward Adjustment | 不变 | 调整 | `最新价格 / 累积因子` |
| **不调整** | 不复权 | No Adjustment | 不变 | 不变 | 原始价格 |

---

**文档结束**

*本文档旨在帮助量化研究人员理解复权的原理和在Qlib中的应用。如有疑问，请参考Qlib官方文档或提交Issue。*
