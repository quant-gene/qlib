# K线(蜡烛图)术语表 / Candlestick Terminology

> 生成时间: 2025-12-15
> 用途: K线技术分析中英文对照术语

## 目录

- [1. K线基本结构](#1-k线基本结构)
- [2. 四个价格点](#2-四个价格点)
- [3. K线类型](#3-k线类型)
- [4. 特殊K线形态](#4-特殊k线形态)
- [5. K线组合形态](#5-k线组合形态)
- [6. 技术指标术语](#6-技术指标术语)
- [7. 交易术语](#7-交易术语)
- [8. Qlib中的K线特征](#8-qlib中的k线特征)
- [9. 常用函数和操作符](#9-常用函数和操作符)

---

## 1. K线基本结构

### 可视化示例

```
阳线 (Bullish)              阴线 (Bearish)              十字星 (Doji)

     ● High 最高价               ● High 最高价               ● High 最高价
     ┃                          ┃                          ┃
     ┃ Upper Shadow             ┃ Upper Shadow             ┃ Upper Shadow
     ┃ 上影线                    ┃ 上影线                    ┃ 上影线
     ┃                          ┃                          ┃
   ┏━━━┓                      ┏━━━┓                        ┃
   ┃   ┃ ← Close 收盘价        ┃   ┃ ← Open 开盘价           ┃
   ┃   ┃                      ┃   ┃                        ┃
   ┃   ┃ ← Body 实体           ┃   ┃ ← Body 实体         ──────── ← Body 实体 (Open≈Close)
   ┃   ┃                      ┃   ┃                        ┃
   ┃   ┃ ← Open 开盘价         ┃   ┃ ← Close 收盘价          ┃
   ┗━━━┛                      ┗━━━┛                        ┃
     ┃                          ┃                          ┃
     ┃ Lower Shadow             ┃ Lower Shadow             ┃ Lower Shadow
     ┃ 下影线                    ┃ 下影线                    ┃ 下影线
     ┃                          ┃                          ┃
     ● Low 最低价                ● Low 最低价                ● Low 最低价
```

### 主要组成部分

| 中文 | 英文名称 | 别名 | 说明 |
|------|---------|------|------|
| **K线** | Candlestick | K-Line / OHLC Bar / Japanese Candlestick | 整体图形 |
| **实体** | Body | Real Body | 开盘价到收盘价之间的矩形部分 |
| **上影线** | Upper Shadow | Upper Wick / Upper Tail | 实体上方到最高价的线段 |
| **下影线** | Lower Shadow | Lower Wick / Lower Tail | 实体下方到最低价的线段 |
| **影线** | Shadow | Wick / Tail | 上影线+下影线的统称 |
| **实体上边** | Upper Edge of Body | Top of Real Body | `Greater($open, $close)` |
| **实体下边** | Lower Edge of Body | Bottom of Real Body | `Less($open, $close)` |

---

## 2. 四个价格点

### OHLC

| 中文 | 英文全称 | 缩写 | Qlib变量 | 说明 |
|------|----------|------|----------|------|
| **开盘价** | Open / Opening Price | O | `$open` | 交易日第一笔成交价格 |
| **最高价** | High / Highest Price | H | `$high` | 交易日内最高成交价格 |
| **最低价** | Low / Lowest Price | L | `$low` | 交易日内最低成交价格 |
| **收盘价** | Close / Closing Price | C | `$close` | 交易日最后一笔成交价格 |

**合称**: OHLC (Open, High, Low, Close)

### 其他价格相关

| 中文 | 英文 | Qlib变量 | 计算公式 |
|------|------|----------|----------|
| **成交量** | Volume / Trading Volume | `$volume` | 累计成交股数 |
| **成交额** | Turnover / Trading Value | `$amount` | 累计成交金额 |
| **成交量加权均价** | VWAP (Volume Weighted Average Price) | `$vwap` | 成交额 / 成交量 |
| **均价** | Average Price | - | `($high + $low) / 2` |
| **典型价格** | Typical Price | - | `($high + $low + $close) / 3` |

---

## 3. K线类型

### 基本分类

| 中文 | 英文 | 别名 | 特征 | 颜色表示 |
|------|------|------|------|----------|
| **阳线** | Bullish Candle | Up Candle / White Candle / Green Candle | `Close > Open` | 白色/红色/绿色(西方) |
| **阴线** | Bearish Candle | Down Candle / Black Candle / Red Candle | `Close < Open` | 黑色/绿色/红色(西方) |
| **十字星** | Doji | Cross / Plus Sign | `Close ≈ Open` | 实体极小 |
| **一字线** | Four Price Doji | Flat Line | `Open = High = Low = Close` | 完全水平线 |

**注意**: 中国和西方的颜色约定相反
- 中国: 红色=阳线(上涨), 绿色=阴线(下跌)
- 西方: 绿色=阳线(上涨), 红色=阴线(下跌)

### 按实体大小分类

| 中文 | 英文 | 特征 |
|------|------|------|
| **大阳线** | Long White Candle / Long Bullish | 实体很长的阳线 |
| **大阴线** | Long Black Candle / Long Bearish | 实体很长的阴线 |
| **小阳线** | Short White Candle / Small Bullish | 实体较短的阳线 |
| **小阴线** | Short Black Candle / Small Bearish | 实体较短的阴线 |
| **秃头** | Bald / Shaven Head | 无上影线 |
| **秃脚** | Shaven Bottom | 无下影线 |

### 按影线特征分类

| 中文 | 英文 | 特征 |
|------|------|------|
| **光头光脚** | Marubozu | 无上下影线，实体占据全部 |
| **光头阳线** | Shaven Head Bullish / Opening Marubozu | 无上影线的阳线 |
| **光头阴线** | Shaven Head Bearish | 无上影线的阴线 |
| **光脚阳线** | Shaven Bottom Bullish / Closing Marubozu | 无下影线的阳线 |
| **光脚阴线** | Shaven Bottom Bearish | 无下影线的阴线 |

---

## 4. 特殊K线形态

### 单根K线形态

| 中文 | 英文 | 特征 | 含义 | 位置 |
|------|------|------|------|------|
| **锤子线** | Hammer | 长下影线(≥2倍实体)，实体小，上影线短或无 | 看涨反转 | 下跌趋势底部 |
| **倒锤线** | Inverted Hammer | 长上影线(≥2倍实体)，实体小，下影线短或无 | 看涨反转 | 下跌趋势底部 |
| **上吊线** | Hanging Man | 长下影线(≥2倍实体)，实体小，上影线短或无 | 看跌反转 | 上涨趋势顶部 |
| **射击之星** | Shooting Star | 长上影线(≥2倍实体)，实体小，下影线短或无 | 看跌反转 | 上涨趋势顶部 |
| **纺锤线** | Spinning Top | 实体小，上下影线都较长 | 犹豫不决 | 任何位置 |
| **长腿十字** | Long-Legged Doji | 十字星，上下影线都很长 | 极度不确定 | 任何位置 |
| **蜻蜓十字** | Dragonfly Doji | 十字星，只有长下影线 | 看涨信号 | 下跌趋势底部 |
| **墓碑十字** | Gravestone Doji | 十字星，只有长上影线 | 看跌信号 | 上涨趋势顶部 |
| **四价十字** | Four Price Doji | 开高低收全部相等 | 极端情况 | 涨跌停板 |

**形态对比**:
```
 锤子线          倒锤线         上吊线         射击之星
 Hammer      Inv. Hammer    Hanging Man   Shooting Star

    ┃              ┃              ┃              ┃
    ┃              ┃              ┃              ┃
  ━━━━           ━━━━           ━━━━           ━━━━
    ┃                             ┃
    ┃                             ┃
    ┃                             ┃

  底部看涨        底部看涨        顶部看跌        顶部看跌
```

### 强弱特征

| 中文 | 英文 | 特征 |
|------|------|------|
| **强势阳线** | Strong Bullish | 收盘价接近最高价 |
| **强势阴线** | Strong Bearish | 收盘价接近最低价 |
| **弱势阳线** | Weak Bullish | 收盘价接近开盘价，上影线长 |
| **弱势阴线** | Weak Bearish | 收盘价接近开盘价，下影线长 |

---

## 5. K线组合形态

### 两根K线组合

| 中文 | 英文 | 特征 | 含义 |
|------|------|------|------|
| **看涨吞没** | Bullish Engulfing | 阳线完全包住前一根阴线 | 强烈看涨 |
| **看跌吞没** | Bearish Engulfing | 阴线完全包住前一根阳线 | 强烈看跌 |
| **乌云盖顶** | Dark Cloud Cover | 阴线开盘高于前阳线收盘，收盘深入前阳线 | 看跌反转 |
| **曙光初现** | Piercing Pattern | 阳线开盘低于前阴线收盘，收盘深入前阴线 | 看涨反转 |
| **十字孕线** | Harami Cross | 十字星完全在前一根K线实体内 | 趋势反转 |
| **孕线** | Harami | 小K线实体完全在前一根大K线实体内 | 趋势减弱 |
| **平顶** | Tweezers Top | 两根K线最高价相同 | 阻力位 |
| **平底** | Tweezers Bottom | 两根K线最低价相同 | 支撑位 |

### 三根K线组合

| 中文 | 英文 | 特征 | 含义 |
|------|------|------|------|
| **启明星** | Morning Star | 大阴+小K+大阳(底部) | 强烈看涨 |
| **黄昏星** | Evening Star | 大阳+小K+大阴(顶部) | 强烈看跌 |
| **三只乌鸦** | Three Black Crows | 三根连续下跌的阴线 | 强烈看跌 |
| **三只白兵** | Three White Soldiers | 三根连续上涨的阳线 | 强烈看涨 |
| **红三兵** | Three Advancing White Soldiers | 三根逐步上涨的阳线 | 持续看涨 |
| **内包日** | Inside Day | 当日K线完全在前日内 | 盘整 |
| **外包日** | Outside Day | 当日K线完全包住前日 | 突破 |

### 其他组合形态

| 中文 | 英文 | 说明 |
|------|------|------|
| **岛形反转** | Island Reversal | 跳空缺口形成的孤立K线群 |
| **头肩顶** | Head and Shoulders Top | 经典顶部反转形态 |
| **头肩底** | Head and Shoulders Bottom / Inverse H&S | 经典底部反转形态 |
| **双顶** | Double Top / M Top | M形顶部形态 |
| **双底** | Double Bottom / W Bottom | W形底部形态 |
| **三角形** | Triangle | 收敛整理形态 |
| **矩形** | Rectangle / Trading Range | 横盘整理区间 |
| **楔形** | Wedge | 倾斜的收敛形态 |
| **旗形** | Flag | 短期整理形态 |
| **三角旗** | Pennant | 小型对称三角形 |

---

## 6. 技术指标术语

### 趋势指标

| 中文 | 英文 | 缩写 | 说明 |
|------|------|------|------|
| **移动平均** | Moving Average | MA | 平滑价格趋势 |
| **指数移动平均** | Exponential Moving Average | EMA | 加权移动平均 |
| **布林带** | Bollinger Bands | BB / BOLL | 价格通道指标 |
| **平均真实波幅** | Average True Range | ATR | 波动率指标 |
| **抛物线SAR** | Parabolic SAR | SAR | 止损和反转指标 |

### 动量指标

| 中文 | 英文 | 缩写 | 说明 |
|------|------|------|------|
| **相对强弱指标** | Relative Strength Index | RSI | 超买超卖指标 |
| **随机指标** | Stochastic Oscillator | KDJ / Stoch | K值、D值、J值 |
| **变化率** | Rate of Change | ROC | 价格变化速度 |
| **动量指标** | Momentum | MOM | 价格动量 |
| **威廉指标** | Williams %R | WR | 超买超卖 |

### 趋势强度

| 中文 | 英文 | 缩写 | 说明 |
|------|------|------|------|
| **平均趋向指标** | Average Directional Index | ADX | 趋势强度 |
| **MACD** | Moving Average Convergence Divergence | MACD | 趋势和动量 |
| **商品通道指标** | Commodity Channel Index | CCI | 超买超卖 |

### 成交量指标

| 中文 | 英文 | 缩写 | 说明 |
|------|------|------|------|
| **能量潮** | On Balance Volume | OBV | 累计成交量 |
| **量价确认指标** | Volume Price Confirmation Indicator | VPCI | 量价背离 |
| **资金流量指标** | Money Flow Index | MFI | 成交量加权RSI |
| **成交量比率** | Volume Ratio | VR | 量能对比 |

---

## 7. 交易术语

### 价格术语

| 中文 | 英文 | 说明 |
|------|------|------|
| **涨跌幅** | Return / Price Change / Percentage Change | `(Close - Prev_Close) / Prev_Close` |
| **振幅** | Range / Amplitude / Price Range | `(High - Low) / Prev_Close` |
| **涨停** | Limit Up / Upper Limit | 当日最大涨幅(中国A股±10%) |
| **跌停** | Limit Down / Lower Limit | 当日最大跌幅 |
| **停牌** | Trading Halt / Suspension | 暂停交易 |
| **复牌** | Resume Trading | 恢复交易 |
| **跳空** | Gap | 开盘价与前收盘价之间的空隙 |
| **向上跳空** | Gap Up | 开盘价高于前收盘价 |
| **向下跳空** | Gap Down | 开盘价低于前收盘价 |
| **缺口** | Gap / Window | 价格不连续区域 |

### 市场状态

| 中文 | 英文 | 说明 |
|------|------|------|
| **牛市** | Bull Market | 上涨趋势市场 |
| **熊市** | Bear Market | 下跌趋势市场 |
| **盘整** | Consolidation / Sideways | 横向整理 |
| **突破** | Breakout | 突破关键价位 |
| **回调** | Pullback / Retracement | 上涨中的短期下跌 |
| **反弹** | Rally / Bounce | 下跌中的短期上涨 |
| **反转** | Reversal | 趋势方向改变 |
| **支撑位** | Support Level | 价格难以跌破的位置 |
| **阻力位** | Resistance Level | 价格难以突破的位置 |
| **趋势线** | Trend Line | 连接高点或低点的直线 |

### 交易行为

| 中文 | 英文 | 说明 |
|------|------|------|
| **做多** | Long / Go Long / Buy | 买入看涨 |
| **做空** | Short / Go Short / Sell Short | 卖出看跌 |
| **平仓** | Close Position / Exit | 结束持仓 |
| **止损** | Stop Loss | 限制亏损 |
| **止盈** | Take Profit | 锁定利润 |
| **追涨** | Chase Rally | 高价买入 |
| **杀跌** | Panic Sell | 低价卖出 |
| **建仓** | Open Position / Establish Position | 开始持仓 |
| **加仓** | Add to Position / Pyramid | 增加持仓 |
| **减仓** | Reduce Position | 减少持仓 |
| **满仓** | Full Position | 全部资金持仓 |
| **空仓** | No Position / Cash | 没有持仓 |

---

## 8. Qlib中的K线特征

### Alpha158 K线特征 (KBAR Features)

基于 `qlib/contrib/data/handler.py` 中的 Alpha158 定义：

| 特征名 | 英文含义 | 公式 | 说明 |
|--------|----------|------|------|
| **KMID** | K Middle (Return) | `($close-$open)/$open` | K线实体涨跌幅（阳线为正，阴线为负） |
| **KLEN** | K Length | `($high-$low)/$open` | K线总长度（振幅） |
| **KMID2** | K Middle 2 | `($close-$open)/($high-$low+1e-12)` | 实体占总长比例（收盘位置强度） |
| **KUP** | K Upper Shadow | `($high-Greater($open,$close))/$open` | 上影线长度 |
| **KUP2** | K Upper Shadow 2 | `($high-Greater($open,$close))/($high-$low+1e-12)` | 上影线占比 |
| **KLOW** | K Lower Shadow | `(Less($open,$close)-$low)/$open` | 下影线长度 |
| **KLOW2** | K Lower Shadow 2 | `(Less($open,$close)-$low)/($high-$low+1e-12)` | 下影线占比 |
| **KSFT** | K Shift | `(2*$close-$high-$low)/$open` | K线重心偏移 |
| **KSFT2** | K Shift 2 | `(2*$close-$high-$low)/($high-$low+1e-12)` | 重心偏移比例 |

### 特征详解

#### KMID - K线实体涨跌幅

```python
KMID = ($close - $open) / $open
```

**含义**:
- 正值: 阳线，当日上涨
- 负值: 阴线，当日下跌
- 数值大小: 涨跌幅度

**示例**:
```python
开盘10元，收盘11元 → KMID = (11-10)/10 = 0.1 (上涨10%)
开盘10元，收盘9元  → KMID = (9-10)/10 = -0.1 (下跌10%)
```

#### KLEN - K线总长度(振幅)

```python
KLEN = ($high - $low) / $open
```

**含义**: 相对于开盘价的振幅

**示例**:
```python
开盘10元，最高11元，最低9元 → KLEN = (11-9)/10 = 0.2 (振幅20%)
```

#### KMID2 - 实体占比

```python
KMID2 = ($close - $open) / ($high - $low + 1e-12)
```

**含义**: 实体在整个K线中的占比
- 接近 +1: 强势阳线（光头光脚阳线）
- 接近 -1: 强势阴线（光头光脚阴线）
- 接近 0: 十字星

**示例**:
```python
# 光头光脚阳线
开10，高11，低10，收11 → KMID2 = (11-10)/(11-10) = 1.0

# 十字星
开10，高11，低9，收10 → KMID2 = (10-10)/(11-9) = 0.0
```

#### KUP / KUP2 - 上影线

```python
KUP = ($high - Greater($open, $close)) / $open
KUP2 = ($high - Greater($open, $close)) / ($high - $low + 1e-12)
```

**Greater($open, $close)**: 实体上边
- 阳线: `$close`
- 阴线: `$open`

**含义**:
- KUP: 上影线绝对长度
- KUP2: 上影线占整个K线的比例

#### KLOW / KLOW2 - 下影线

```python
KLOW = (Less($open, $close) - $low) / $open
KLOW2 = (Less($open, $close) - $low) / ($high - $low + 1e-12)
```

**Less($open, $close)**: 实体下边
- 阳线: `$open`
- 阴线: `$close`

**含义**:
- KLOW: 下影线绝对长度
- KLOW2: 下影线占整个K线的比例

#### KSFT / KSFT2 - 重心偏移

```python
KSFT = (2*$close - $high - $low) / $open
KSFT2 = (2*$close - $high - $low) / ($high - $low + 1e-12)
```

**含义**: 收盘价相对于K线中点的偏移
- 正值: 收盘价偏向最高价（上半部分）
- 负值: 收盘价偏向最低价（下半部分）
- 接近0: 收盘价在K线中点

**中点公式**: `($high + $low) / 2`

**推导**:
```python
偏移 = $close - 中点
     = $close - ($high + $low) / 2
     = (2*$close - $high - $low) / 2

# 标准化（除以开盘价或振幅）
KSFT = (2*$close - $high - $low) / $open
```

---

## 9. 常用函数和操作符

### Qlib表达式函数

基于 `qlib/data/ops.py` 中的操作符定义：

#### 基础运算

| 函数 | 说明 | 示例 |
|------|------|------|
| `$field` | 取字段值 | `$close`, `$open` |
| `+` `-` `*` `/` | 四则运算 | `$close - $open` |
| `Greater(a, b)` | 取较大值 | `Greater($open, $close)` |
| `Less(a, b)` | 取较小值 | `Less($open, $close)` |
| `Abs(x)` | 绝对值 | `Abs($close - $open)` |
| `Sign(x)` | 符号函数 | `Sign($close - $open)` |
| `Log(x)` | 自然对数 | `Log($volume + 1)` |
| `Power(x, n)` | 幂运算 | `Power($close, 2)` |

#### 时间序列运算

| 函数 | 说明 | 示例 |
|------|------|------|
| `Ref(x, n)` | n天前的值 | `Ref($close, 1)` = 昨日收盘价 |
| `Delta(x, n)` | n天变化量 | `Delta($close, 1)` = 今日收盘-昨日收盘 |
| `Mean(x, n)` | n天均值 | `Mean($close, 5)` = 5日均价 |
| `Sum(x, n)` | n天求和 | `Sum($volume, 5)` = 5日成交量和 |
| `Std(x, n)` | n天标准差 | `Std($close, 20)` = 20日波动率 |
| `Var(x, n)` | n天方差 | `Var($close, 10)` |
| `Max(x, n)` | n天最大值 | `Max($high, 20)` = 20日最高价 |
| `Min(x, n)` | n天最小值 | `Min($low, 20)` = 20日最低价 |
| `Med(x, n)` | n天中位数 | `Med($close, 10)` |
| `Mad(x, n)` | n天平均绝对偏差 | `Mad($close, 10)` |
| `Rank(x, n)` | n天排名 | `Rank($close, 10)` |
| `Quantile(x, n, q)` | n天分位数 | `Quantile($close, 20, 0.8)` |

#### 回归和相关

| 函数 | 说明 | 示例 |
|------|------|------|
| `Slope(x, n)` | n天线性回归斜率 | `Slope($close, 10)` |
| `Rsquare(x, n)` | n天线性回归R² | `Rsquare($close, 20)` |
| `Resi(x, n)` | n天线性回归残差 | `Resi($close, 10)` |
| `Corr(x, y, n)` | n天相关系数 | `Corr($close, $volume, 20)` |
| `Cov(x, y, n)` | n天协方差 | `Cov($close, $volume, 10)` |

#### 技术指标

| 函数 | 说明 | 示例 |
|------|------|------|
| `WMA(x, n)` | 加权移动平均 | `WMA($close, 10)` |
| `EMA(x, n)` | 指数移动平均 | `EMA($close, 12)` |
| `IdxMax(x, n)` | 最大值位置索引 | `IdxMax($high, 20)` |
| `IdxMin(x, n)` | 最小值位置索引 | `IdxMin($low, 20)` |

#### 截面运算 (Cross-Sectional)

| 函数 | 说明 | 应用 |
|------|------|------|
| `CSRank(x)` | 当日所有股票排名 | 相对排名 |
| `CSZScore(x)` | 当日截面Z-Score | 标准化 |
| `CSMean(x)` | 当日截面均值 | 市场平均水平 |
| `CSStd(x)` | 当日截面标准差 | 市场波动 |

### 特殊值处理

```python
# 避免除零
($close - $open) / ($high - $low + 1e-12)

# 避免log(0)
Log($volume + 1)

# 避免除以接近0的数
$volume / ($volume + 1e-12)
```

---

## 附录A: K线形态识别速查表

### 看涨形态

| 形态 | 英文 | 位置 | 可靠度 |
|------|------|------|--------|
| 锤子线 | Hammer | 底部 | ★★★ |
| 倒锤线 | Inverted Hammer | 底部 | ★★ |
| 看涨吞没 | Bullish Engulfing | 底部 | ★★★★ |
| 曙光初现 | Piercing Pattern | 底部 | ★★★ |
| 启明星 | Morning Star | 底部 | ★★★★★ |
| 三只白兵 | Three White Soldiers | 上涨中 | ★★★★ |

### 看跌形态

| 形态 | 英文 | 位置 | 可靠度 |
|------|------|------|--------|
| 上吊线 | Hanging Man | 顶部 | ★★★ |
| 射击之星 | Shooting Star | 顶部 | ★★ |
| 看跌吞没 | Bearish Engulfing | 顶部 | ★★★★ |
| 乌云盖顶 | Dark Cloud Cover | 顶部 | ★★★ |
| 黄昏星 | Evening Star | 顶部 | ★★★★★ |
| 三只乌鸦 | Three Black Crows | 下跌中 | ★★★★ |

### 中性/持续形态

| 形态 | 英文 | 含义 |
|------|------|------|
| 十字星 | Doji | 趋势犹豫 |
| 纺锤线 | Spinning Top | 方向不明 |
| 孕线 | Harami | 趋势减弱 |

---

## 附录B: 价格归一化方法

在Qlib和技术分析中，常见的价格归一化方法：

### 方法1: 除以当前收盘价

```python
normalized_price = $field / $close
```

**优点**: 去除价格水平影响，不同价格股票可比
**用于**: Alpha158的大部分特征

**示例**:
```python
MA5 = Mean($close, 5) / $close
ROC10 = Ref($close, 10) / $close
```

### 方法2: 除以前一日收盘价

```python
normalized_return = $field / Ref($close, 1)
```

**优点**: 表示相对于昨日的变化
**用于**: 计算收益率

**示例**:
```python
daily_return = $close / Ref($close, 1) - 1
```

### 方法3: 除以开盘价

```python
normalized_price = $field / $open
```

**优点**: 表示日内变化
**用于**: K线特征(KMID, KLEN等)

### 方法4: 除以振幅

```python
normalized_position = $field / ($high - $low + 1e-12)
```

**优点**: 表示在当日范围内的相对位置
**用于**: K线相对位置特征(KMID2, KUP2等)

### 方法5: 除以成交量

```python
normalized_volume = $volume / ($volume + 1e-12)
```

**优点**: 自归一化，值为1
**用于**: 成交量特征，避免量纲影响

---

## 附录C: 参考资源

### 书籍

- **《日本蜡烛图技术》** - Steve Nison (Candlestick Bible)
- **《技术分析》** - John Murphy
- **《股票作手回忆录》** - Edwin Lefèvre

### 在线资源

- **Investopedia**: https://www.investopedia.com/
- **StockCharts**: https://stockcharts.com/school/
- **TradingView**: https://www.tradingview.com/
- **Qlib Documentation**: https://qlib.readthedocs.io/

### Qlib相关

- **Alpha158源码**: `qlib/contrib/data/loader.py`
- **K线特征**: `qlib/contrib/data/handler.py`
- **数据操作符**: `qlib/data/ops.py`

---

**文档结束**

*本术语表旨在帮助量化研究人员理解K线图分析和Qlib框架中的相关术语。*
