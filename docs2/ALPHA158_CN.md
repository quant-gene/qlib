# Alpha158 特征集详解

> 版本: Qlib 预定义特征集
> 特征数量: 158个
> 适用市场: 股票市场（日频数据）
> 数据前提: 底层价格须为**前复权价**（Qlib 标准数据默认满足，详见附录D）

## 目录

- [1. Alpha158 概览](#1-alpha158-概览)
- [2. K线特征 (9个)](#2-k线特征-9个)
- [3. 价格特征 (4个)](#3-价格特征-4个)
- [4. 滚动窗口特征 (145个)](#4-滚动窗口特征-145个)
- [5. 标签定义](#5-标签定义)
- [6. 数据预处理器](#6-数据预处理器)
- [7. 使用示例](#7-使用示例)
- [8. 特征重要性分析](#8-特征重要性分析)

---

## 1. Alpha158 概览

### 什么是 Alpha158?

**Alpha158** 是Qlib预定义的一个特征集，包含 **158个技术指标特征**，是量化投资中最常用的基础特征集之一。

### 特征组成

```
Alpha158 (158个特征)
├── K线特征 (9个)      - 单日价格形态
├── 价格特征 (4个)      - 当日价格标准化
└── 滚动窗口特征 (145个) - 多时间窗口技术指标
    ├── 趋势指标 (15个)
    ├── 波动性指标 (15个)
    ├── 价格位置指标 (30个)
    ├── 动量指标 (15个)
    ├── 量价关系指标 (40个)
    └── 成交量指标 (30个)
```

### 设计理念

1. **多维度**: 从价格、成交量、趋势、动量等多个角度描述股票
2. **多时间尺度**: 使用5、10、20、30、60天等不同窗口
3. **标准化**: 所有特征都经过标准化，消除价格水平影响
4. **通用性**: 适用于大多数股票市场和交易策略

---

## 2. K线特征 (9个)

### 特征列表

基于开盘价、最高价、最低价、收盘价的K线形态特征：

| 特征名 | 公式 | 说明 |
|--------|------|------|
| `KMID` | `($close-$open)/$open` | K线实体涨跌幅（阳线为正，阴线为负） |
| `KLEN` | `($high-$low)/$open` | K线总长度（振幅） |
| `KMID2` | `($close-$open)/($high-$low+1e-12)` | 实体占总长比例（收盘位置强度） |
| `KUP` | `($high-Greater($open,$close))/$open` | 上影线长度 |
| `KUP2` | `($high-Greater($open,$close))/($high-$low+1e-12)` | 上影线占比 |
| `KLOW` | `(Less($open,$close)-$low)/$open` | 下影线长度 |
| `KLOW2` | `(Less($open,$close)-$low)/($high-$low+1e-12)` | 下影线占比 |
| `KSFT` | `(2*$close-$high-$low)/$open` | K线重心偏移 |
| `KSFT2` | `(2*$close-$high-$low)/($high-$low+1e-12)` | 重心偏移比例 |

### 详细说明

#### KMID - K线实体涨跌幅

```python
KMID = ($close - $open) / $open
```

**含义**: 当日开盘到收盘的涨跌幅

**示例**:
```python
# 阳线
开盘10元，收盘11元 → KMID = (11-10)/10 = 0.1 (上涨10%)

# 阴线
开盘10元，收盘9元 → KMID = (9-10)/10 = -0.1 (下跌10%)

# 十字星
开盘10元，收盘10元 → KMID = 0
```

**用途**: 衡量多空力量对比

#### KLEN - K线总长度

```python
KLEN = ($high - $low) / $open
```

**含义**: 相对于开盘价的振幅

**示例**:
```python
开盘10元，最高11元，最低9元
KLEN = (11-9)/10 = 0.2 (振幅20%)
```

**用途**: 衡量当日波动程度

#### KMID2 - 实体占比

```python
KMID2 = ($close - $open) / ($high - $low + 1e-12)
```

**含义**: 实体在整个K线中的占比

**取值范围**: [-1, 1]
- 接近 +1: 强势阳线（光头光脚阳线）
- 接近 -1: 强势阴线（光头光脚阴线）
- 接近 0: 十字星或纺锤线

**示例**:
```python
# 光头光脚阳线
开10，高11，低10，收11 → KMID2 = (11-10)/(11-10) = 1.0

# 十字星
开10，高11，低9，收10 → KMID2 = (10-10)/(11-9) = 0.0

# 长上影线阳线
开10，高12，低10，收10.5 → KMID2 = (10.5-10)/(12-10) = 0.25
```

**用途**: 判断收盘价位置强度

#### KUP / KUP2 - 上影线

```python
KUP = ($high - Greater($open, $close)) / $open
KUP2 = ($high - Greater($open, $close)) / ($high - $low + 1e-12)
```

**Greater($open, $close)**: 实体上边
- 阳线时 = `$close`
- 阴线时 = `$open`

**含义**:
- KUP: 上影线绝对长度
- KUP2: 上影线占整个K线的比例

**用途**: 识别上方压力

#### KLOW / KLOW2 - 下影线

```python
KLOW = (Less($open, $close) - $low) / $open
KLOW2 = (Less($open, $close) - $low) / ($high - $low + 1e-12)
```

**Less($open, $close)**: 实体下边
- 阳线时 = `$open`
- 阴线时 = `$close`

**含义**:
- KLOW: 下影线绝对长度
- KLOW2: 下影线占整个K线的比例

**用途**: 识别下方支撑

#### KSFT / KSFT2 - 重心偏移

```python
KSFT = (2*$close - $high - $low) / $open
KSFT2 = (2*$close - $high - $low) / ($high - $low + 1e-12)
```

**含义**: 收盘价相对于K线中点的偏移

**推导**:
```python
K线中点 = ($high + $low) / 2
偏移 = $close - 中点
     = $close - ($high + $low) / 2
     = (2*$close - $high - $low) / 2
```

**取值**:
- 正值: 收盘价在上半部分
- 负值: 收盘价在下半部分
- 接近0: 收盘价在中点

**用途**: 判断收盘价在K线中的位置

### K线特征的用途

捕捉单日价格形态信息：
- **上涨/下跌强度**: KMID
- **波动程度**: KLEN
- **多空力量对比**: KMID2
- **上方压力**: KUP, KUP2
- **下方支撑**: KLOW, KLOW2
- **收盘位置**: KSFT, KSFT2

---

## 3. 价格特征 (4个)

### 配置

```python
"price": {
    "windows": [0],  # 当日(0天前)
    "feature": ["OPEN", "HIGH", "LOW", "VWAP"]
}
```

### 特征列表

当日各类价格相对于收盘价的标准化值：

| 特征名 | 公式 | 说明 |
|--------|------|------|
| `OPEN0` | `$open/$close` | 开盘价/收盘价 |
| `HIGH0` | `$high/$close` | 最高价/收盘价 |
| `LOW0` | `$low/$close` | 最低价/收盘价 |
| `VWAP0` | `$vwap/$close` | 成交量加权均价/收盘价 |
| `CLOSE0` | `$close/$close = 1` | 收盘价基准（=1 常量，不作为独立特征计列） |

### 标准化原理

**为什么除以收盘价?**
- 消除价格水平差异（10元股票和100元股票可比）
- 特征值在相似范围内，便于机器学习模型训练
- 相对价格比绝对价格更有预测价值

**示例**:
```python
# 股票A: 10元
开10.2, 高10.5, 低9.8, 收10, VWAP 10.1
OPEN0 = 10.2/10 = 1.02
HIGH0 = 10.5/10 = 1.05
LOW0 = 9.8/10 = 0.98
VWAP0 = 10.1/10 = 1.01

# 股票B: 100元
开102, 高105, 低98, 收100, VWAP 101
OPEN0 = 102/100 = 1.02
HIGH0 = 105/100 = 1.05
LOW0 = 98/100 = 0.98
VWAP0 = 101/100 = 1.01

# 两只股票的特征完全相同！
```

### 用途

- 提供价格分布信息
- 标准化处理使特征在不同价格水平上可比
- VWAP反映当日平均成交价格，与收盘价对比可判断收盘强弱

---

## 4. 滚动窗口特征 (145个)

基于过去多个时间窗口 **(5, 10, 20, 30, 60天)** 计算的技术指标。

### 4.1 趋势指标 (15个)

| 特征类型 | 窗口 | 数量 | 说明 |
|----------|------|------|------|
| **ROC** (Rate of Change) | 5,10,20,30,60 | 5个 | 价格变化率，衡量涨跌幅度 |
| **MA** (Moving Average) | 5,10,20,30,60 | 5个 | 简单移动平均，平滑价格趋势 |
| **BETA** (Slope) | 5,10,20,30,60 | 5个 | 线性回归斜率，趋势强度 |

#### 公式示例

```python
# ROC - 价格变化率
ROC5 = Ref($close, 5) / $close        # 5天前价格/当前价格
ROC10 = Ref($close, 10) / $close
# 大于1: 价格上涨; 小于1: 价格下跌

# MA - 移动平均
MA10 = Mean($close, 10) / $close      # 10天均价/当前价格
MA20 = Mean($close, 20) / $close
# 大于1: 当前价格低于均线; 小于1: 当前价格高于均线

# BETA - 线性回归斜率
BETA20 = Slope($close, 20) / $close   # 20天价格斜率
# 正值: 上升趋势; 负值: 下降趋势
```

#### 用途
- ROC: 识别短期/长期涨跌幅
- MA: 判断价格相对于均线位置（支撑/压力）
- BETA: 衡量趋势强度和方向

### 4.2 波动性指标 (15个)

| 特征类型 | 窗口 | 数量 | 说明 |
|----------|------|------|------|
| **STD** (Standard Deviation) | 5,10,20,30,60 | 5个 | 价格标准差，衡量波动性 |
| **RSQR** (R-Square) | 5,10,20,30,60 | 5个 | 回归R²，趋势线性度 |
| **RESI** (Residual) | 5,10,20,30,60 | 5个 | 回归残差，偏离趋势程度 |

#### 公式示例

```python
# STD - 标准差
STD30 = Std($close, 30) / $close           # 30天价格标准差
# 值越大，波动越大

# RSQR - R平方
RSQR60 = Rsquare($close, 60)               # 60天线性拟合度
# 接近1: 趋势明显; 接近0: 无明显趋势

# RESI - 残差
RESI10 = Resi($close, 10) / $close         # 10天回归残差
# 正值: 高于趋势线; 负值: 低于趋势线
```

#### 用途
- STD: 衡量风险（波动率）
- RSQR: 判断趋势可靠性
- RESI: 识别均值回归机会

### 4.3 价格位置指标 (30个)

| 特征类型 | 窗口 | 数量 | 说明 |
|----------|------|------|------|
| **MAX** | 5,10,20,30,60 | 5个 | 区间最高价 |
| **MIN** | 5,10,20,30,60 | 5个 | 区间最低价 |
| **QTLU** (Quantile Upper) | 5,10,20,30,60 | 5个 | 80%分位数 |
| **QTLD** (Quantile Lower) | 5,10,20,30,60 | 5个 | 20%分位数 |
| **RANK** | 5,10,20,30,60 | 5个 | 当前价格在区间内的排名 |
| **RSV** (Raw Stochastic Value) | 5,10,20,30,60 | 5个 | 随机指标原始值 |

#### 公式示例

```python
# MAX/MIN - 区间最高/最低价
MAX20 = Max($high, 20) / $close                    # 20天最高价
MIN20 = Min($low, 20) / $close                     # 20天最低价
# 用于识别支撑阻力位

# QTLU/QTLD - 分位数
QTLU30 = Quantile($close, 30, 0.8) / $close       # 30天80%分位数
QTLD30 = Quantile($close, 30, 0.2) / $close       # 30天20%分位数
# 识别价格分布的上下边界

# RANK - 排名
RANK10 = Rank($close, 10)                          # 当前价在10天内排名
# 0-1之间，接近1: 接近最高价; 接近0: 接近最低价

# RSV - 随机值（KDJ指标的基础）
RSV5 = ($close - Min($low, 5)) / (Max($high, 5) - Min($low, 5) + 1e-12)
# 当前价格在区间内的相对位置
```

#### 用途
- MAX/MIN: 识别支撑阻力
- QTLU/QTLD: 布林带类似功能
- RANK: 判断价格在区间内的相对强弱
- RSV: 超买超卖信号（KDJ基础）

### 4.4 动量指标 (15个)

| 特征类型 | 窗口 | 数量 | 说明 |
|----------|------|------|------|
| **IMAX** (Index Max) | 5,10,20,30,60 | 5个 | 距离最高点天数 |
| **IMIN** (Index Min) | 5,10,20,30,60 | 5个 | 距离最低点天数 |
| **IMXD** (Max-Min Distance) | 5,10,20,30,60 | 5个 | 最高点与最低点时间差 |

#### 公式示例

```python
# IMAX - 距离最高点天数
IMAX30 = IdxMax($high, 30) / 30      # 30天内最高点出现位置
# 0: 今天是最高点; 1: 30天前是最高点

# IMIN - 距离最低点天数
IMIN30 = IdxMin($low, 30) / 30       # 30天内最低点出现位置
# 0: 今天是最低点; 1: 30天前是最低点

# IMXD - 高低点时间差
IMXD30 = (IdxMax($high, 30) - IdxMin($low, 30)) / 30
# 正值: 先最低后最高(上涨); 负值: 先最高后最低(下跌)
```

#### 用途（基于Aroon指标思想）
- IMAX: 距离最高点越近，上涨动能越强
- IMIN: 距离最低点越近，下跌动能越强
- IMXD: 判断动量方向

### 4.5 量价关系指标 (40个)

| 特征类型 | 窗口 | 数量 | 说明 |
|----------|------|------|------|
| **CORR** | 5,10,20,30,60 | 5个 | 价格与成交量相关性 |
| **CORD** | 5,10,20,30,60 | 5个 | 价格变化与成交量相关性 |
| **CNTP** | 5,10,20,30,60 | 5个 | 上涨天数占比 |
| **CNTN** | 5,10,20,30,60 | 5个 | 下跌天数占比 |
| **CNTD** | 5,10,20,30,60 | 5个 | 上涨下跌天数差 |
| **SUMP** | 5,10,20,30,60 | 5个 | 上涨日价格变化占比（RSI） |
| **SUMN** | 5,10,20,30,60 | 5个 | 下跌日价格变化占比 |
| **SUMD** | 5,10,20,30,60 | 5个 | 涨跌变化占比差（RSI） |

#### 公式示例

```python
# CORR - 价量相关性
CORR20 = Corr($close, Log($volume+1), 20)    # 20天价量相关性
# 正相关: 价涨量增; 负相关: 价涨量缩

# CORD - 价格变化与成交量相关性
CORD10 = Corr(
    $close / Ref($close, 1),
    Log($volume / Ref($volume, 1) + 1),
    10
)

# CNTP - 上涨天数比例
CNTP30 = Mean($close > Ref($close, 1), 30)      # 30天上涨天数比例
# 大于0.5: 上涨天数多; 小于0.5: 下跌天数多

# CNTN - 下跌天数比例
CNTN30 = Mean($close < Ref($close, 1), 30)

# CNTD - 上涨下跌天数差
CNTD30 = CNTP30 - CNTN30

# SUMP - 上涨日价格变化占比（RSI）
SUMP20 = Sum(Greater($close - Ref($close, 1), 0), 20) / (Sum(Abs($close - Ref($close, 1)), 20) + 1e-12)
# 上涨日价格变动占总变动的比例，越大代表上涨越集中

# SUMN - 下跌日价格变化占比
SUMN20 = Sum(Greater(Ref($close, 1) - $close, 0), 20) / (Sum(Abs($close - Ref($close, 1)), 20) + 1e-12)
# SUMN = 1 - SUMP

# SUMD - 涨跌变化占比差（RSI 指标）
SUMD20 = SUMP20 - SUMN20
```

#### 用途
- CORR/CORD: 判断量价配合情况
- CNTP/CNTN: 统计胜率
- CNTD: 趋势一致性
- SUMP/SUMN/SUMD: 价格涨跌变化占比（类似RSI）

### 4.6 成交量指标 (30个)

| 特征类型 | 窗口 | 数量 | 说明 |
|----------|------|------|------|
| **VSUMP** | 5,10,20,30,60 | 5个 | 量增日占比 |
| **VSUMN** | 5,10,20,30,60 | 5个 | 量减日占比 |
| **VSUMD** | 5,10,20,30,60 | 5个 | 量增量减占比差（成交量RSI） |
| **VMA** | 5,10,20,30,60 | 5个 | 成交量移动平均 |
| **VSTD** | 5,10,20,30,60 | 5个 | 成交量标准差 |
| **WVMA** | 5,10,20,30,60 | 5个 | 成交量加权价格波动 |

#### 公式示例

```python
# VSUMP - 量增占比
VSUMP20 = Sum(Greater($volume - Ref($volume, 1), 0), 20) / (Sum(Abs($volume - Ref($volume, 1)), 20) + 1e-12)
# 量增日成交量变化占总变化的比例

# VSUMN - 量减占比
VSUMN20 = Sum(Greater(Ref($volume, 1) - $volume, 0), 20) / (Sum(Abs($volume - Ref($volume, 1)), 20) + 1e-12)
# VSUMN = 1 - VSUMP

# VSUMD - 量增量减占比差（成交量RSI）
VSUMD20 = VSUMP20 - VSUMN20

# VMA - 成交量均值
VMA10 = Mean($volume, 10) / ($volume + 1e-12)
# 当前成交量相对于平均水平

# VSTD - 成交量波动
VSTD30 = Std($volume, 30) / ($volume + 1e-12)

# WVMA - 成交量加权价格波动
WVMA60 = Std(Abs($close / Ref($close, 1) - 1) * $volume, 60) / (Mean(Abs($close / Ref($close, 1) - 1) * $volume, 60) + 1e-12)
# 价格波动加权的成交量波动
```

#### 用途
- VSUMP/VSUMN/VSUMD: 成交量增减占比（成交量的RSI指标）
- VMA: 成交量放大/缩小
- VSTD: 成交量稳定性
- WVMA: 波动与成交量关系

---

## 5. 标签定义

### 标签公式

```python
label = "Ref($close, -2)/Ref($close, -1) - 1"
```

### 含义解释

```python
# 未来第2天收盘价 / 未来第1天收盘价 - 1
# = 未来第1天到第2天的收益率

Ref($close, -2)  # 未来第2天的收盘价（T+2）
Ref($close, -1)  # 未来第1天的收盘价（T+1）

label = (T+2价格 / T+1价格) - 1
```

### 为什么这样设计？

**避免使用当天收盘价进行交易**

```
时间线:
T日     T+1日    T+2日
收盘    收盘     收盘
  ↓      ↓        ↓
 训练   (交易)   目标
        买入点   卖出点
```

**实际交易流程**:
1. T日收盘后，使用T日及之前的数据进行预测
2. T+1日开盘/盘中进行交易
3. 目标是预测T+1到T+2的收益率

**优势**:
- 更符合实际交易场景
- 避免使用不可交易价格（T日收盘价）
- 给出明确的持有期（1天）

### 其他常见标签

```python
# 1. 预测明日收益率（较激进）
"Ref($close, -1)/$close - 1"

# 2. 预测未来5日收益率
"Ref($close, -5)/Ref($close, -1) - 1"

# 3. 预测未来最高价（价格预测）
"Max($high, 5) / $close"

# 4. 预测是否上涨（分类）
"Ref($close, -1) > $close"
```

---

## 6. 数据预处理器

### 6.1 学习阶段处理器 (learn_processors)

用于训练数据的预处理：

```python
_DEFAULT_LEARN_PROCESSORS = [
    {"class": "DropnaLabel"},        # 删除标签缺失的样本
    {"class": "CSZScoreNorm",        # 截面Z-Score标准化
     "kwargs": {"fields_group": "label"}}
]
```

#### DropnaLabel

**功能**: 移除没有标签的样本

**原因**:
- 停牌股票无法计算未来收益率
- 退市股票
- 数据缺失

#### CSZScoreNorm

**CS = Cross-Sectional (截面)**

**功能**: 每个时间点对所有股票的标签进行Z-Score标准化

**公式**:
```python
normalized_label = (label - mean_t) / std_t
```

其中 `mean_t` 和 `std_t` 是t时刻所有股票标签的均值和标准差

**目的**:
- 消除市场整体涨跌影响
- 专注于相对表现（相对收益）
- 使标签均值为0，标准差为1

**示例**:
```python
# 某日所有股票收益率
股票A: +5%
股票B: +3%
股票C: +1%
股票D: -1%
股票E: -3%
均值: +1%

# 截面标准化后
股票A: +1.5  # 高于平均
股票B: +0.75
股票C: 0
股票D: -0.75
股票E: -1.5  # 低于平均
```

### 6.2 推理阶段处理器 (infer_processors)

用于预测时的特征预处理：

```python
_DEFAULT_INFER_PROCESSORS = [
    {"class": "ProcessInf"},    # 处理无穷值
    {"class": "ZScoreNorm"},    # Z-Score标准化
    {"class": "Fillna"}         # 填充缺失值
]
```

#### ProcessInf

**功能**: 将无穷值 (inf, -inf) 替换为极大/极小有限值

**原因**: 某些计算可能产生无穷值（如除零）

#### ZScoreNorm

**功能**: 时间序列Z-Score标准化

**与CSZScoreNorm的区别**:
- ZScoreNorm: 使用训练期统计量标准化
- CSZScoreNorm: 每个时间点独立标准化

**公式**:
```python
normalized_feature = (feature - mean_train) / std_train
```

其中 `mean_train` 和 `std_train` 是训练期计算的均值和标准差

#### Fillna

**功能**: 用0填充缺失值

**原因**: 机器学习模型通常不能处理NaN

### 6.3 时间范围配置

```yaml
start_time: 2008-01-01      # 全部数据起始时间
end_time: 2020-08-01        # 全部数据结束时间
fit_start_time: 2008-01-01  # 预处理器拟合起始时间
fit_end_time: 2014-12-31    # 预处理器拟合结束时间
```

**关键点**:
- `fit_*` 时间用于计算标准化参数（均值、标准差）
- 只使用训练期数据计算统计量
- 避免使用未来数据（Look-ahead Bias）

**数据流**:
```
原始数据 (2008-2020)
    ↓
计算统计量 (2008-2014)  ← fit_start_time 到 fit_end_time
    ↓
应用标准化 (2008-2020)  ← 全部数据使用同一标准化参数
    ↓
标准化后的数据
```

---

## 7. 使用示例

### 7.1 配置文件使用

```yaml
# workflow_config.yaml
task:
    dataset:
        class: DatasetH
        module_path: qlib.data.dataset
        kwargs:
            handler:
                class: Alpha158
                module_path: qlib.contrib.data.handler
                kwargs:
                    start_time: 2008-01-01
                    end_time: 2020-08-01
                    fit_start_time: 2008-01-01
                    fit_end_time: 2014-12-31
                    instruments: csi300
            segments:
                train: [2008-01-01, 2014-12-31]
                valid: [2015-01-01, 2016-12-31]
                test: [2017-01-01, 2020-08-01]
```

### 7.2 Python代码使用

```python
import qlib
from qlib.contrib.data.handler import Alpha158
from qlib.data.dataset import DatasetH

# 初始化Qlib
qlib.init(provider_uri="~/.qlib/qlib_data/cn_data", region="cn")

# 创建Alpha158处理器
handler = Alpha158(
    instruments="csi300",
    start_time="2008-01-01",
    end_time="2020-08-01",
    fit_start_time="2008-01-01",
    fit_end_time="2014-12-31"
)

# 创建数据集
dataset = DatasetH(
    handler=handler,
    segments={
        "train": ("2008-01-01", "2014-12-31"),
        "valid": ("2015-01-01", "2016-12-31"),
        "test": ("2017-01-01", "2020-08-01"),
    }
)

# 获取训练数据
train_data = dataset.prepare("train", col_set=["feature", "label"])
print(f"特征维度: {train_data['feature'].shape}")
print(f"标签维度: {train_data['label'].shape}")

# 输出示例:
# 特征维度: (540000, 158)  # 约300只股票 × 1800交易日 × 158特征
# 标签维度: (540000, 1)
```

### 7.3 查看特征名称

```python
# 获取所有特征名称
feature_names = handler.get_feature_config()[1]
print(f"特征数量: {len(feature_names)}")
print(f"前10个特征: {feature_names[:10]}")

# 输出:
# 特征数量: 158
# 前10个特征: ['KMID', 'KLEN', 'KMID2', 'KUP', 'KUP2', 'KLOW', 'KLOW2', 'KSFT', 'KSFT2', 'OPEN0']
```

---

## 8. 特征重要性分析

### 8.1 特征分类统计

| 类别 | 数量 | 占比 | 主要作用 |
|------|------|------|----------|
| K线特征 | 9 | 5.7% | 日内形态 |
| 价格特征 | 4 | 2.5% | 价格分布 |
| 趋势指标 | 15 | 9.5% | 趋势方向和强度 |
| 波动指标 | 15 | 9.5% | 风险度量 |
| 位置指标 | 30 | 19.0% | 支撑阻力 |
| 动量指标 | 15 | 9.5% | 动量强度 |
| 量价指标 | 40 | 25.3% | 资金流向 |
| 成交量指标 | 30 | 19.0% | 交易活跃度 |
| **总计** | **158** | **100%** | - |

### 8.2 常见的高重要性特征

基于实证研究，以下特征通常具有较高的预测能力：

#### Top 20 重要特征（示例）

1. **BETA20** - 20日趋势强度
2. **ROC20** - 20日动量
3. **RESI10** - 10日回归残差（均值回归）
4. **CORR20** - 20日量价相关性
5. **RSV20** - 20日随机指标
6. **KMID2** - K线实体占比
7. **RANK20** - 20日价格排名
8. **STD30** - 30日波动率
9. **SUMD20** - 20日成交量差
10. **MA10** - 10日均线
11. **IMXD30** - 30日高低点时间差
12. **QTLU30** - 30日上分位数
13. **RSQR30** - 30日趋势线性度
14. **CORD10** - 10日价格变化与量相关性
15. **KLOW2** - 下影线占比
16. **VMA20** - 20日成交量均值
17. **CNTP30** - 30日上涨天数比例
18. **KSFT2** - K线重心偏移
19. **WVMA60** - 60日加权成交量波动
20. **IMAX20** - 20日距最高点天数

**注意**: 特征重要性会随着市场状态、股票池、时间段而变化。

### 8.3 特征工程建议

#### 已包含的信息
- ✅ 价格趋势（ROC、MA、BETA）
- ✅ 价格波动（STD、RSQR、RESI）
- ✅ 价格位置（MAX、MIN、RANK、RSV）
- ✅ 价格动量（IMAX、IMIN、IMXD）
- ✅ 量价关系（CORR、CORD、CNTP、CNTD）
- ✅ 成交量信息（VMA、VSTD、SUMP、SUMN）

#### 可以增强的方向
- ❌ 基本面因子（市盈率、ROE、营收增长等）
- ❌ 市场微观结构（买卖价差、订单簿）
- ❌ 情绪指标（新闻、社交媒体）
- ❌ 行业/板块信息
- ❌ 宏观经济指标
- ❌ 另类数据

### 8.4 使用注意事项

1. **特征相关性**: 某些特征高度相关（如不同窗口的MA）
2. **过拟合风险**: 158个特征较多，注意正则化
3. **特征选择**: 可以使用LASSO、树模型特征重要性筛选
4. **归一化**: 已经除以$close标准化，但仍需ZScore归一化
5. **缺失值**: 新上市股票、停牌会有缺失值
6. **前视偏差**: 确保fit_time不包含测试期

---

## 附录A: Alpha158 完整特征列表

### K线特征 (9个)
KMID, KLEN, KMID2, KUP, KUP2, KLOW, KLOW2, KSFT, KSFT2

### 价格特征 (4个)
OPEN0, HIGH0, LOW0, VWAP0
(注: CLOSE0 = $close/$close = 1 为基准常量，不作为独立特征计列)

### 滚动窗口特征 (145个)

#### 窗口: 5, 10, 20, 30, 60天

- ROC5, ROC10, ROC20, ROC30, ROC60 (5个)
- MA5, MA10, MA20, MA30, MA60 (5个)
- STD5, STD10, STD20, STD30, STD60 (5个)
- BETA5, BETA10, BETA20, BETA30, BETA60 (5个)
- RSQR5, RSQR10, RSQR20, RSQR30, RSQR60 (5个)
- RESI5, RESI10, RESI20, RESI30, RESI60 (5个)
- MAX5, MAX10, MAX20, MAX30, MAX60 (5个)
- MIN5, MIN10, MIN20, MIN30, MIN60 (5个)
- QTLU5, QTLU10, QTLU20, QTLU30, QTLU60 (5个)
- QTLD5, QTLD10, QTLD20, QTLD30, QTLD60 (5个)
- RANK5, RANK10, RANK20, RANK30, RANK60 (5个)
- RSV5, RSV10, RSV20, RSV30, RSV60 (5个)
- IMAX5, IMAX10, IMAX20, IMAX30, IMAX60 (5个)
- IMIN5, IMIN10, IMIN20, IMIN30, IMIN60 (5个)
- IMXD5, IMXD10, IMXD20, IMXD30, IMXD60 (5个)
- CORR5, CORR10, CORR20, CORR30, CORR60 (5个)
- CORD5, CORD10, CORD20, CORD30, CORD60 (5个)
- CNTP5, CNTP10, CNTP20, CNTP30, CNTP60 (5个)
- CNTN5, CNTN10, CNTN20, CNTN30, CNTN60 (5个)
- CNTD5, CNTD10, CNTD20, CNTD30, CNTD60 (5个)
- SUMP5, SUMP10, SUMP20, SUMP30, SUMP60 (5个)
- SUMN5, SUMN10, SUMN20, SUMN30, SUMN60 (5个)
- SUMD5, SUMD10, SUMD20, SUMD30, SUMD60 (5个)
- VSUMP5, VSUMP10, VSUMP20, VSUMP30, VSUMP60 (5个)
- VSUMN5, VSUMN10, VSUMN20, VSUMN30, VSUMN60 (5个)
- VSUMD5, VSUMD10, VSUMD20, VSUMD30, VSUMD60 (5个)
- VMA5, VMA10, VMA20, VMA30, VMA60 (5个)
- VSTD5, VSTD10, VSTD20, VSTD30, VSTD60 (5个)
- WVMA5, WVMA10, WVMA20, WVMA30, WVMA60 (5个)

**总计**: 9 + 4 + 145 = **158个特征**

---

## 附录B: 相关资源

### Qlib文档
- **Alpha158源码**: `qlib/contrib/data/handler.py`
- **特征配置**: `qlib/contrib/data/loader.py` → `Alpha158DL.get_feature_config()`
- **数据操作符**: `qlib/data/ops.py`

### 学术论文
- **Qlib论文**: [Qlib: An AI-oriented Quantitative Investment Platform](https://arxiv.org/abs/2009.11189)
- **Alpha因子研究**: 《101 Formulaic Alphas》

### 在线资源
- **Qlib文档**: https://qlib.readthedocs.io/
- **GitHub**: https://github.com/microsoft/qlib

---

## 附录C: 因子公式与源码对齐校验

> 校验日期: 2026-08-10
> 校验方式: 将文档中的公式与 `qlib/contrib/data/loader.py` → `Alpha158DL.get_feature_config()` 实际生成的 158 个特征表达式,剥离空白与注释后逐字符比对;差值型定义另做数学等价验证。

### 结论

文档中全部 **158 个特征** 的计算公式与代码实现**完全对齐**。

### C.1 归一化后与源码逐字符一致 (41 处)

- **K线 (9)**: `KMID`, `KLEN`, `KMID2`, `KUP`, `KUP2`, `KLOW`, `KLOW2`, `KSFT`, `KSFT2`
- **价格 (4)**: `OPEN0`, `HIGH0`, `LOW0`, `VWAP0`
- **滚动 (28 处示例, 覆盖 26 种类型)**: `ROC`, `MA`, `STD`, `BETA`, `RSQR`, `RESI`, `MAX`, `MIN`, `QTLU`, `QTLD`, `RANK`, `RSV`, `IMAX`, `IMIN`, `IMXD`, `CORR`, `CORD`, `CNTP`, `CNTN`, `SUMP`, `SUMN`, `VMA`, `VSTD`, `WVMA`, `VSUMP`, `VSUMN`

### C.2 差值定义, 数学等价 (3 种类型)

| 因子 | 文档写法 | 源码实现 | 等价性 |
|------|----------|----------|--------|
| `CNTD` | `CNTD30 = CNTP30 - CNTN30` | `Mean($close>Ref($close,1),d) - Mean($close<Ref($close,1),d)` | 恒等 |
| `SUMD` | `SUMD20 = SUMP20 - SUMN20` | `(Σ涨-Σ跌) / (Σ\|Δclose\| + 1e-12)` | 恒等(两式同分母) |
| `VSUMD` | `VSUMD20 = VSUMP20 - VSUMN20` | `(Σ量增-Σ量减) / (Σ\|Δvolume\| + 1e-12)` | 恒等(两式同分母) |

### C.3 非真实特征 (1 个)

- `CLOSE0 = $close/$close = 1`: 基准常量,源码不生成,不计入 158 个特征。

### C.4 覆盖性说明

滚动特征在文档中以"每类一个窗口示例"呈现(如 `SUMP20`、`WVMA60`),源码对 **29 类 × 5 窗口**(5/10/20/30/60)使用同一表达式模板。示例窗口比对通过,即该类型全部 5 个窗口均通过。

**总计: 9(K线) + 4(价格) + 145(滚动) = 158 个特征,公式全部对齐。**

---

## 附录D: 复权数据说明与校验

> 相关文档: [ADJUSTMENT_CALCULATION_CN.md](ADJUSTMENT_CALCULATION_CN.md)

### 因子与复权的关系

Alpha158 本身不进行任何复权计算,只是对 `$close/$open/$high/$low/$volume` 等字段做标准化(公式对齐见附录C)。底层数据是否复权,直接决定因子池是否基于复权价:

- **前复权**(Qlib 标准): 保持最新价不变,向下调整历史价;`$open/$close/$high/$low/$volume` 均为调整后价格
- **未复权**: 除权日出现跳空,ROC/MA/STD/量价相关等技术指标失真

### 校验方法

```python
from qlib.data import D

# 方法1: 检查是否存在复权因子字段
try:
    factors = D.features(["000001.SZ"], ["$factor"])
    print("数据包含复权因子,已复权")
except Exception:
    print("数据不包含复权因子,可能未复权")

# 方法2: 检查除权日是否出现异常跌幅(>20% 且无重大利空)
returns = D.features(["000001.SZ"], ["$close"]).pct_change()
print("异常跌幅天数:", (returns < -0.2).sum().sum())
```

### 注意

- `dump_bin.py` 只做格式转换,不计算复权因子;factor 需由数据源(Tushare 等)在 CSV 中提供
- 前复权为"动态历史":每次除权后历史价格整体重算;长周期回测如需稳定的历史序列,可自行用 `$factor` 处理后复权

---

**文档结束**

*本文档详细介绍了Qlib中Alpha158特征集的构成、计算方法和使用方式，是量化投资研究的重要参考资料。*
