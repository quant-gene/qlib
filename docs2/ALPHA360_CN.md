# Alpha360 特征集详解

> 版本: Qlib 预定义特征集
> 特征数量: 360个
> 适用市场: 股票市场（日频数据）
> 设计理念: 原始价格序列 + 标准化

## 目录

- [1. Alpha360 概览](#1-alpha360-概览)
- [2. 与Alpha158的对比](#2-与alpha158的对比)
- [3. 特征构成详解](#3-特征构成详解)
- [4. 标准化方法](#4-标准化方法)
- [5. 标签定义](#5-标签定义)
- [6. 使用场景](#6-使用场景)
- [7. 使用示例](#7-使用示例)
- [8. 优缺点分析](#8-优缺点分析)

---

## 1. Alpha360 概览

### 什么是 Alpha360?

**Alpha360** 是Qlib提供的另一个预定义特征集，包含 **360个特征**，主要特点是提供**原始价格序列数据**而不是计算复杂的技术指标。

### 设计哲学

```
Alpha158: 手工设计的158个技术指标 (Feature Engineering)
    ↓
Alpha360: 60天原始价量数据 (Let Model Learn)
    ↓
让深度学习模型自己学习特征组合和模式
```

**核心思想**:
- 不对数据进行过多预处理
- 保留原始价格时间序列信息
- 让模型（尤其是深度学习模型）自动学习特征
- 更适合LSTM、Transformer等时序模型

### 特征组成

```
Alpha360 (360个特征)
├── CLOSE序列 (60个) - 过去60天收盘价
├── OPEN序列 (60个)  - 过去60天开盘价
├── HIGH序列 (60个)  - 过去60天最高价
├── LOW序列 (60个)   - 过去60天最低价
├── VWAP序列 (60个)  - 过去60天VWAP
└── VOLUME序列 (60个) - 过去60天成交量

总计: 6 × 60 = 360个特征
```

---

## 2. 与Alpha158的对比

### 2.1 设计理念对比

| 维度 | Alpha158 | Alpha360 |
|------|----------|----------|
| **特征数量** | 158个 | 360个 |
| **设计方式** | 手工特征工程 | 原始序列数据 |
| **技术指标** | 丰富(趋势、波动、动量等) | 无（原始数据） |
| **时间窗口** | 多窗口(5/10/20/30/60天) | 固定60天 |
| **适用模型** | 树模型(GBDT)、线性模型 | 深度学习(LSTM/Transformer) |
| **可解释性** | 高（每个特征有明确含义） | 低（需模型学习） |
| **计算开销** | 中等（需计算技术指标） | 低（直接使用原始数据） |

### 2.2 特征对比示例

#### Alpha158的方式

```python
# 需要计算各种技术指标
ROC5 = Ref($close, 5) / $close          # 5日动量
MA10 = Mean($close, 10) / $close        # 10日均线
STD20 = Std($close, 20) / $close        # 20日波动率
CORR30 = Corr($close, Log($volume+1), 30)  # 30日量价相关
...
# 共158个这样的特征
```

#### Alpha360的方式

```python
# 直接提供原始价格序列
CLOSE59 = Ref($close, 59) / $close      # 59天前收盘价
CLOSE58 = Ref($close, 58) / $close      # 58天前收盘价
...
CLOSE1 = Ref($close, 1) / $close        # 1天前收盘价
CLOSE0 = $close / $close = 1            # 当天收盘价(基准)

# OPEN、HIGH、LOW、VWAP、VOLUME同理
# 总共360个特征
```

### 2.3 优势对比

#### Alpha158的优势
- ✅ 特征有明确金融含义
- ✅ 适合传统机器学习模型
- ✅ 训练速度快
- ✅ 可解释性强
- ✅ 数据量需求小

#### Alpha360的优势
- ✅ 保留完整时序信息
- ✅ 适合深度学习模型
- ✅ 无需特征工程
- ✅ 模型可以学习复杂模式
- ✅ 计算简单直接

---

## 3. 特征构成详解

### 3.1 收盘价序列 (60个)

```python
# 特征命名: CLOSE59, CLOSE58, ..., CLOSE1, CLOSE0

for i in range(59, 0, -1):
    fields += [f"Ref($close, {i})/$close"]
    names += [f"CLOSE{i}"]
fields += ["$close/$close"]
names += ["CLOSE0"]
```

**特征列表**:
- `CLOSE59`: 59天前的收盘价 / 当天收盘价
- `CLOSE58`: 58天前的收盘价 / 当天收盘价
- ...
- `CLOSE1`: 昨天的收盘价 / 当天收盘价
- `CLOSE0`: 当天的收盘价 / 当天收盘价 = 1

**含义解释**:
```python
# 示例: 当前股价 = 100元

CLOSE0 = 100/100 = 1.00   # 今天(基准)
CLOSE1 = 99/100 = 0.99    # 昨天下跌1%
CLOSE2 = 98/100 = 0.98    # 前天又跌1%
CLOSE5 = 105/100 = 1.05   # 5天前高5%
...
CLOSE59 = 90/100 = 0.90   # 59天前低10%
```

**时间序列视角**:
```
[CLOSE59, CLOSE58, ..., CLOSE1, CLOSE0]
   59天前  58天前  ...   昨天    今天
     ↓       ↓     ...    ↓      ↓
    0.90    0.91  ...   0.99    1.00

这形成了一个长度为60的价格序列
```

### 3.2 开盘价序列 (60个)

```python
# 特征命名: OPEN59, OPEN58, ..., OPEN1, OPEN0

for i in range(59, 0, -1):
    fields += [f"Ref($open, {i})/$close"]
    names += [f"OPEN{i}"]
fields += ["$open/$close"]
names += ["OPEN0"]
```

**注意**:
- 标准化基准是当天的**收盘价**，不是开盘价
- 这保证了所有特征使用统一的标准化基准

**示例**:
```python
# 当天: 开10, 收11
OPEN0 = 10/11 = 0.909   # 当天开盘价低于收盘价(阳线)

# 昨天: 开10.5, 收11
OPEN1 = 10.5/11 = 0.955 # 昨天开盘价
```

### 3.3 最高价序列 (60个)

```python
# 特征命名: HIGH59, HIGH58, ..., HIGH1, HIGH0

for i in range(59, 0, -1):
    fields += [f"Ref($high, {i})/$close"]
    names += [f"HIGH{i}"]
fields += ["$high/$close"]
names += ["HIGH0"]
```

**含义**: 每天的最高价相对于当天收盘价

### 3.4 最低价序列 (60个)

```python
# 特征命名: LOW59, LOW58, ..., LOW1, LOW0

for i in range(59, 0, -1):
    fields += [f"Ref($low, {i})/$close"]
    names += [f"LOW{i}"]
fields += ["$low/$close"]
names += ["LOW0"]
```

**含义**: 每天的最低价相对于当天收盘价

### 3.5 VWAP序列 (60个)

```python
# 特征命名: VWAP59, VWAP58, ..., VWAP1, VWAP0

for i in range(59, 0, -1):
    fields += [f"Ref($vwap, {i})/$close"]
    names += [f"VWAP{i}"]
fields += ["$vwap/$close"]
names += ["VWAP0"]
```

**VWAP** (Volume Weighted Average Price): 成交量加权平均价

### 3.6 成交量序列 (60个)

```python
# 特征命名: VOLUME59, VOLUME58, ..., VOLUME1, VOLUME0

for i in range(59, 0, -1):
    fields += [f"Ref($volume, {i})/($volume+1e-12)"]
    names += [f"VOLUME{i}"]
fields += ["$volume/($volume+1e-12)"]
names += ["VOLUME0"]
```

**注意**:
- 成交量是自标准化：除以当天成交量
- 添加`1e-12`避免除零
- `VOLUME0 = 1` (当天成交量/当天成交量)

**示例**:
```python
# 当天成交量 = 1000万股
VOLUME0 = 1000万/1000万 = 1.0   # 今天(基准)
VOLUME1 = 800万/1000万 = 0.8    # 昨天缩量
VOLUME5 = 1500万/1000万 = 1.5   # 5天前放量
```

---

## 4. 标准化方法

### 4.1 价格标准化

**所有价格字段(CLOSE, OPEN, HIGH, LOW, VWAP)使用当天收盘价标准化**:

```python
normalized_price = $price / $close
```

**优点**:
1. **统一基准**: 所有价格特征用同一基准，保持一致性
2. **相对价格**: 反映相对变化而非绝对价格
3. **当天基准为1**: `CLOSE0 = 1`，其他天数相对于1波动
4. **可比性**: 不同价格水平的股票特征可比

**示例对比**:

```python
# 10元股票
开10, 高11, 低9.5, 收10.5
OPEN0 = 10/10.5 = 0.952
HIGH0 = 11/10.5 = 1.048
LOW0 = 9.5/10.5 = 0.905

# 100元股票
开100, 高110, 低95, 收105
OPEN0 = 100/105 = 0.952
HIGH0 = 110/105 = 1.048
LOW0 = 95/105 = 0.905

# 标准化后的特征完全相同！
```

### 4.2 成交量标准化

**成交量使用当天成交量自标准化**:

```python
normalized_volume = $volume / ($volume + 1e-12)
```

**为什么不用收盘价标准化?**
- 成交量和价格是不同量纲
- 自标准化能反映成交量的相对变化

### 4.3 进一步标准化

在数据加载后，通常还会应用：

```python
# Z-Score标准化
features = (features - mean) / std

# 使得所有特征：
# - 均值接近0
# - 标准差为1
```

---

## 5. 标签定义

### 默认标签

```python
# Alpha360 默认标签
label = "Ref($close, -2)/Ref($close, -1) - 1"
```

**含义**: 预测未来1天(T+1到T+2)的收益率

### Alpha360vwap 变体

```python
# 使用VWAP而非收盘价
class Alpha360vwap(Alpha360):
    def get_label_config(self):
        return ["Ref($vwap, -2)/Ref($vwap, -1) - 1"], ["LABEL0"]
```

**用途**:
- VWAP更能反映真实成交价格
- 适合大资金、机构投资者
- 减少收盘价可能的异常波动影响

---

## 6. 使用场景

### 6.1 适合使用Alpha360的情况

✅ **使用深度学习模型**
```python
# LSTM模型
model = LSTM(input_size=360, hidden_size=64, num_layers=2)

# Transformer模型
model = Transformer(d_model=360, nhead=8)

# 这些模型可以处理时序数据，自动学习特征
```

✅ **需要捕捉复杂时序模式**
- 价格趋势的非线性变化
- 多时间尺度的相互作用
- 隐含的周期性模式

✅ **数据量充足**
- 深度学习需要大量数据
- Alpha360特征维度高(360维)
- 需要足够样本避免过拟合

✅ **不确定哪些特征重要**
- 让模型自己学习
- 避免人为选择带来的偏差

### 6.2 不适合使用Alpha360的情况

❌ **使用传统机器学习模型**
```python
# GBDT/Random Forest等
# 这些模型不能很好地利用时序结构
# Alpha158会更合适
```

❌ **数据量不足**
- 360维特征需要更多数据
- 小样本情况下容易过拟合

❌ **需要可解释性**
- Alpha360特征不直观
- 难以解释模型决策

❌ **计算资源有限**
- 深度学习训练慢
- 需要GPU加速

---

## 7. 使用示例

### 7.1 配置文件

```yaml
# workflow_config_alpha360.yaml
task:
    dataset:
        class: DatasetH
        module_path: qlib.data.dataset
        kwargs:
            handler:
                class: Alpha360
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

### 7.2 Python代码

```python
import qlib
from qlib.contrib.data.handler import Alpha360
from qlib.data.dataset import DatasetH
import numpy as np

# 初始化Qlib
qlib.init(provider_uri="~/.qlib/qlib_data/cn_data", region="cn")

# 创建Alpha360处理器
handler = Alpha360(
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
X_train = train_data["feature"].values  # (N, 360)
y_train = train_data["label"].values    # (N, 1)

print(f"特征维度: {X_train.shape}")
print(f"标签维度: {y_train.shape}")

# 输出:
# 特征维度: (540000, 360)
# 标签维度: (540000, 1)
```

### 7.3 与LSTM模型配合

```python
import torch
import torch.nn as nn

class LSTMModel(nn.Module):
    def __init__(self):
        super().__init__()
        # 360个特征可以reshape为 (60, 6)
        # 60个时间步，每步6个特征(OHLCV+VWAP)
        self.lstm = nn.LSTM(
            input_size=6,      # 每个时间步6个特征
            hidden_size=64,
            num_layers=2,
            batch_first=True
        )
        self.fc = nn.Linear(64, 1)

    def forward(self, x):
        # x shape: (batch, 360)
        # reshape to (batch, 60, 6)
        x = x.view(-1, 60, 6)
        out, _ = self.lstm(x)
        out = self.fc(out[:, -1, :])  # 取最后一个时间步
        return out

# 使用
model = LSTMModel()
```

### 7.4 特征重组技巧

```python
def reshape_alpha360(features):
    """
    将360个特征重组为时序格式
    Input: (N, 360)
    Output: (N, 60, 6)  # 60天 × 6个字段
    """
    N = features.shape[0]
    # Alpha360顺序: CLOSE(60) + OPEN(60) + HIGH(60) + LOW(60) + VWAP(60) + VOLUME(60)
    features_reshaped = features.reshape(N, 6, 60)
    # 转置为 (N, 60, 6)
    features_reshaped = features_reshaped.transpose(0, 2, 1)
    return features_reshaped

# 使用
X_train_seq = reshape_alpha360(X_train)  # (540000, 60, 6)
```

---

## 8. 优缺点分析

### 8.1 优点

#### ✅ 保留完整时序信息
```python
# Alpha158: 只保留5/10/20/30/60天的统计量
MA20  # 丢失了这20天的具体变化过程

# Alpha360: 保留完整60天序列
[CLOSE59, CLOSE58, ..., CLOSE1, CLOSE0]
# 可以看到每一天的价格变化
```

#### ✅ 适合深度学习
- LSTM可以捕捉长短期依赖
- Transformer可以学习全局模式
- CNN可以识别局部模式

#### ✅ 无需特征工程
- 不需要选择技术指标
- 不需要调整窗口大小
- 模型自动学习

#### ✅ 灵活性高
```python
# 可以很容易调整回溯天数
Alpha360_30天: 30 × 6 = 180个特征
Alpha360_90天: 90 × 6 = 540个特征
```

### 8.2 缺点

#### ❌ 维度高
- 360维 >> 158维
- 需要更多数据
- 训练时间更长

#### ❌ 可解释性差
```python
# Alpha158: "BETA20表示20日趋势强度"
# Alpha360: "CLOSE37是什么意思？" 不直观
```

#### ❌ 不适合传统模型
```python
# GBDT/LR不能很好利用时序结构
# 可能不如Alpha158表现好
```

#### ❌ 冗余信息多
```python
# 相邻天数的价格高度相关
CLOSE1 ≈ CLOSE2 ≈ CLOSE3  # 相关性>0.99
# 对非深度学习模型是噪音
```

### 8.3 性能对比（经验值）

| 模型类型 | Alpha158 | Alpha360 | 说明 |
|---------|----------|----------|------|
| **LightGBM** | ★★★★★ | ★★★☆☆ | Alpha158更好 |
| **XGBoost** | ★★★★★ | ★★★☆☆ | Alpha158更好 |
| **MLP** | ★★★☆☆ | ★★★★☆ | Alpha360略好 |
| **LSTM** | ★★★☆☆ | ★★★★★ | Alpha360显著更好 |
| **Transformer** | ★★☆☆☆ | ★★★★★ | Alpha360显著更好 |
| **训练速度** | 快 | 慢 | Alpha158快3-5倍 |
| **可解释性** | 高 | 低 | Alpha158容易解释 |

---

## 附录A: Alpha360 完整特征列表

### 特征命名规则

```
<字段名><天数>

字段名: CLOSE, OPEN, HIGH, LOW, VWAP, VOLUME
天数: 59, 58, ..., 1, 0
```

### 收盘价特征 (60个)
```
CLOSE59, CLOSE58, CLOSE57, ..., CLOSE2, CLOSE1, CLOSE0
```

### 开盘价特征 (60个)
```
OPEN59, OPEN58, OPEN57, ..., OPEN2, OPEN1, OPEN0
```

### 最高价特征 (60个)
```
HIGH59, HIGH58, HIGH57, ..., HIGH2, HIGH1, HIGH0
```

### 最低价特征 (60个)
```
LOW59, LOW58, LOW57, ..., LOW2, LOW1, LOW0
```

### VWAP特征 (60个)
```
VWAP59, VWAP58, VWAP57, ..., VWAP2, VWAP1, VWAP0
```

### 成交量特征 (60个)
```
VOLUME59, VOLUME58, VOLUME57, ..., VOLUME2, VOLUME1, VOLUME0
```

**总计**: 60 × 6 = **360个特征**

---

## 附录B: 特征可视化

### 单只股票的Alpha360特征

```python
import matplotlib.pyplot as plt
import numpy as np

# 假设已有features (360,)
features = features.reshape(6, 60).T  # (60, 6)

fig, axes = plt.subplots(3, 2, figsize=(15, 12))

field_names = ['CLOSE', 'OPEN', 'HIGH', 'LOW', 'VWAP', 'VOLUME']
for idx, (ax, name) in enumerate(zip(axes.flat, field_names)):
    data = features[:, idx]
    ax.plot(range(60), data)
    ax.set_title(f'{name} - 过去60天')
    ax.set_xlabel('天数(0=今天, 59=59天前)')
    ax.set_ylabel('标准化值')
    ax.grid(True)
    ax.axhline(y=1, color='r', linestyle='--', alpha=0.3)

plt.tight_layout()
plt.show()
```

---

## 附录C: 相关资源

### Qlib文档
- **Alpha360源码**: `qlib/contrib/data/handler.py`
- **特征配置**: `qlib/contrib/data/loader.py` → `Alpha360DL.get_feature_config()`
- **Alpha158对比**: [ALPHA158_CN.md](ALPHA158_CN.md)

### 相关论文
- **Qlib论文**: [Qlib: An AI-oriented Quantitative Investment Platform](https://arxiv.org/abs/2009.11189)
- **LSTM for股票预测**: 多篇相关研究

### 推荐阅读
- **时间序列预测**: LSTM、GRU、Transformer应用
- **深度学习调参**: Learning rate、batch size、dropout等
- **特征工程 vs 深度学习**: 何时使用哪种方法

---

## 总结

### Alpha360的定位

```
手工特征工程 ←──────────────→ 原始数据
    (Alpha158)                  (Alpha360)

    适合树模型                   适合深度学习
    特征少但精                   特征多保留信息
    训练快可解释                 训练慢自动学习
```

### 选择建议

**使用Alpha158如果**:
- 使用GBDT/XGBoost等树模型
- 需要快速实验和迭代
- 需要可解释性
- 数据量有限

**使用Alpha360如果**:
- 使用LSTM/Transformer等深度学习
- 有充足的计算资源和数据
- 追求极致性能
- 愿意花时间调参

**最佳实践**:
1. 先用Alpha158+LightGBM建立baseline
2. 再尝试Alpha360+深度学习模型
3. 对比两者性能
4. 也可以尝试Alpha158+Alpha360组合

---

**文档结束**

*本文档详细介绍了Qlib中Alpha360特征集的设计、使用和与Alpha158的对比，是深度学习量化投资研究的重要参考。*
