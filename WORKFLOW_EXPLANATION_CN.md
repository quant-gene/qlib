# Qlib 工作流配置详解

> 基于: `examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml`
> 生成时间: 2025-12-15
> 目的: 详细解释Qlib工作流配置中使用的各个组件

## 目录

- [1. 工作流概览](#1-工作流概览)
- [2. 配置文件结构](#2-配置文件结构)
- [3. Qlib初始化配置](#3-qlib初始化配置)
- [4. 数据处理器: Alpha158](#4-数据处理器-alpha158)
- [5. 数据集: DatasetH](#5-数据集-dataseth)
- [6. 模型: LGBModel](#6-模型-lgbmodel)
- [7. 策略: TopkDropoutStrategy](#7-策略-topkdropoutstrategy)
- [8. 回测配置](#8-回测配置)
- [9. 记录器配置](#9-记录器配置)
- [10. 完整工作流执行流程](#10-完整工作流执行流程)

---

## 1. 工作流概览

这个工作流展示了Qlib中一个完整的量化投资研究流程：

```
数据准备   →   特征工程   →   模型训练   →   策略回测   →   结果分析
   ↓             ↓            ↓             ↓            ↓
Qlib Init     Alpha158     LGBModel      TopkDrop      Records
```

### 主要组件

1. **数据层**: Alpha158特征工程 + DatasetH数据集
2. **模型层**: LightGBM梯度提升树模型
3. **策略层**: TopkDropoutStrategy选股策略
4. **回测层**: 交易所模拟 + 成本建模
5. **记录层**: 信号记录 + 分析记录 + 组合分析

---

## 2. 配置文件结构

```yaml
workflow_config_lightgbm_Alpha158.yaml
├── qlib_init              # Qlib初始化配置
├── market & benchmark     # 市场和基准设置
├── data_handler_config    # 数据处理器配置
├── port_analysis_config   # 组合分析配置
│   ├── strategy          # 交易策略
│   └── backtest          # 回测设置
└── task                   # 主任务配置
    ├── model             # 模型配置
    ├── dataset           # 数据集配置
    └── record            # 记录器配置
```

---

## 3. Qlib初始化配置

### 配置内容

```yaml
qlib_init:
    provider_uri: "~/.qlib/qlib_data/cn_data"
    region: cn
```

### 参数说明

| 参数 | 值 | 说明 |
|------|-----|------|
| `provider_uri` | `~/.qlib/qlib_data/cn_data` | 数据存储路径，指向本地Qlib数据目录 |
| `region` | `cn` | 市场区域标识，cn表示中国市场 |

### 功能说明

- **数据源配置**: 指定Qlib从哪里读取历史数据
- **市场配置**: 定义交易日历、交易规则等市场特性
- **支持的区域**:
  - `cn` - 中国A股市场
  - `us` - 美国股票市场
  - 其他自定义市场

### 使用示例

```python
import qlib
qlib.init(
    provider_uri="~/.qlib/qlib_data/cn_data",
    region="cn"
)
```

---

## 4. 数据处理器: Alpha158

### 配置内容

```yaml
market: &market csi300
benchmark: &benchmark SH000300
data_handler_config: &data_handler_config
    start_time: 2008-01-01
    end_time: 2020-08-01
    fit_start_time: 2008-01-01
    fit_end_time: 2014-12-31
    instruments: *market

task:
    dataset:
        kwargs:
            handler:
                class: Alpha158
                module_path: qlib.contrib.data.handler
                kwargs: *data_handler_config
```

### 什么是Alpha158?

> **详细文档**: 请参考 [ALPHA158_CN.md](ALPHA158_CN.md) 获取完整的Alpha158特征集说明

Alpha158是Qlib预定义的一个特征集，包含**158个技术指标特征**。

#### 特征组成简要概览

```
Alpha158 (158个特征)
├── K线特征 (9个)      - KMID, KLEN, KMID2, KUP, KUP2, KLOW, KLOW2, KSFT, KSFT2
├── 价格特征 (5个)      - OPEN0, HIGH0, LOW0, VWAP0, CLOSE0
└── 滚动窗口特征 (144个) - 基于5/10/20/30/60天窗口
    ├── 趋势指标: ROC, MA, BETA (15个)
    ├── 波动指标: STD, RSQR, RESI (15个)
    ├── 位置指标: MAX, MIN, QTLU, QTLD, RANK, RSV (30个)
    ├── 动量指标: IMAX, IMIN, IMXD (15个)
    ├── 量价指标: CORR, CORD, CNTP, CNTN, CNTD (25个)
    └── 成交量指标: SUMP, SUMN, SUMD, VMA, VSTD, WVMA (30个)
```

**设计理念**:
- 多维度: 价格、成交量、趋势、动量等
- 多时间尺度: 5、10、20、30、60天窗口
- 标准化: 消除价格水平影响
- 通用性: 适用于大多数股票市场

> 📖 **完整特征说明**: 每个特征的详细公式、含义、用途等，请查阅 [ALPHA158_CN.md](ALPHA158_CN.md)

#### 4.1 特征示例

**K线特征示例**:
- `KMID = ($close-$open)/$open` - K线实体涨跌幅
- `KLEN = ($high-$low)/$open` - K线振幅

**价格特征示例**:
- `OPEN0 = $open/$close` - 开盘价标准化
- `VWAP0 = $vwap/$close` - VWAP标准化

**滚动窗口特征示例**:
- `ROC20 = Ref($close, 20)/$close` - 20日动量
- `MA10 = Mean($close, 10)/$close` - 10日均线
- `CORR20 = Corr($close, Log($volume+1), 20)` - 20日量价相关性


### 标签定义

```python
label = "Ref($close, -2)/Ref($close, -1) - 1"  # 预测未来1天的收益率
```

**解释**:
- `Ref($close, -2)`: 未来第2天的收盘价
- `Ref($close, -1)`: 未来第1天的收盘价
- 计算未来第1天到第2天的收益率
- 这种设计避免了使用当天收盘价(可能无法交易)

### 数据预处理器

#### 学习阶段处理器 (learn_processors)

```python
_DEFAULT_LEARN_PROCESSORS = [
    {"class": "DropnaLabel"},        # 删除标签缺失的样本
    {"class": "CSZScoreNorm",        # 截面Z-Score标准化
     "kwargs": {"fields_group": "label"}}
]
```

**DropnaLabel**: 移除没有标签的样本(如停牌、退市股票)

**CSZScoreNorm**:
- CS = Cross-Sectional(截面)
- 每个时间点对所有股票的标签进行Z-Score标准化
- 公式: `(x - mean) / std`
- 目的: 消除市场整体涨跌影响，专注于相对表现

#### 推理阶段处理器 (infer_processors)

```python
_DEFAULT_INFER_PROCESSORS = [
    {"class": "ProcessInf"},    # 处理无穷值
    {"class": "ZScoreNorm"},    # Z-Score标准化
    {"class": "Fillna"}         # 填充缺失值
]
```

**ProcessInf**: 将无穷值替换为极大/极小有限值

**ZScoreNorm**: 时间序列Z-Score标准化(使用fit阶段统计量)

**Fillna**: 用0填充缺失值

### 时间范围配置

```yaml
start_time: 2008-01-01      # 全部数据起始时间
end_time: 2020-08-01        # 全部数据结束时间
fit_start_time: 2008-01-01  # 预处理器拟合起始时间
fit_end_time: 2014-12-31    # 预处理器拟合结束时间
```

**关键点**:
- `fit_*` 时间用于计算标准化参数(均值、标准差)
- 避免使用未来数据，保证真实性

---

## 5. 数据集: DatasetH

### 配置内容

```yaml
dataset:
    class: DatasetH
    module_path: qlib.data.dataset
    kwargs:
        handler:
            class: Alpha158
            module_path: qlib.contrib.data.handler
            kwargs: *data_handler_config
        segments:
            train: [2008-01-01, 2014-12-31]
            valid: [2015-01-01, 2016-12-31]
            test: [2017-01-01, 2020-08-01]
```

### DatasetH 是什么?

**DatasetH** = Dataset with **H**andler，是Qlib中带数据处理器的数据集类。

### 架构层次

```
DatasetH
    ├── Handler (Alpha158)
    │   ├── DataLoader
    │   │   └── 原始数据 ($close, $open, $high, $low, $volume)
    │   ├── Feature Calculation
    │   │   └── 158个技术指标
    │   └── Processors
    │       ├── infer_processors (推理预处理)
    │       └── learn_processors (学习预处理)
    └── Segments (数据分割)
        ├── train: 2008-2014 (训练集)
        ├── valid: 2015-2016 (验证集)
        └── test: 2017-2020 (测试集)
```

### 数据分割策略

#### 时间序列分割

```
Timeline: ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
          2008        2014  2016      2020
          ↓           ↓     ↓         ↓
          ├─ Train ──┤├Valid├─ Test ─┤
          (7年)      (2年)  (3.7年)
```

**为什么这样分割?**

1. **时间顺序**: 严格按时间顺序，避免未来信息泄露
2. **训练集较长**: 7年数据捕捉多种市场状态(牛市、熊市、震荡)
3. **验证集**: 用于超参数调优和早停
4. **测试集**: 模拟真实预测场景

#### 数据使用方式

```python
# 在模型训练时
dataset.prepare("train", col_set=["feature", "label"])
# 返回: 2008-2014年的特征和标签

# 在模型预测时
dataset.prepare("test", col_set="feature")
# 返回: 2017-2020年的特征(无标签)
```

### DataHandlerLP 处理类型

```python
process_type = DataHandlerLP.PTYPE_A  # "append" 模式
```

**PTYPE_A (Append)** 处理流程:

```
原始数据 (raw)
    ↓
shared_processors (共享处理器)
    ↓
infer_processors (推理处理器)
    ↓
推理数据 (infer) ← 用于预测
    ↓
learn_processors (学习处理器)
    ↓
学习数据 (learn) ← 用于训练
```

**对比 PTYPE_I (Independent)**:

```
原始数据 (raw)
    ↓
    ├→ infer_processors → 推理数据 (infer)
    └→ learn_processors → 学习数据 (learn)
```

**为什么用PTYPE_A?**
- 推理和学习共享基础处理
- 学习阶段可以额外处理标签(如删除缺失标签)
- 更高效，避免重复计算

---

## 6. 模型: LGBModel

### 配置内容

```yaml
model:
    class: LGBModel
    module_path: qlib.contrib.model.gbdt
    kwargs:
        loss: mse
        colsample_bytree: 0.8879
        learning_rate: 0.2
        subsample: 0.8789
        lambda_l1: 205.6999
        lambda_l2: 580.9768
        max_depth: 8
        num_leaves: 210
        num_threads: 20
```

### LGBModel 是什么?

**LGBModel** 是Qlib对 **LightGBM** (Light Gradient Boosting Machine) 的封装。

### LightGBM 算法原理

#### 梯度提升决策树 (GBDT)

```
预测值 = 树1 + 树2 + 树3 + ... + 树N

每棵树学习前面所有树的残差(误差)
```

**训练过程**:
1. 初始化预测值(通常为0或均值)
2. 计算当前预测的残差
3. 训练一棵新树拟合残差
4. 更新预测值 = 旧预测 + 学习率 × 新树预测
5. 重复步骤2-4，直到达到树的数量或收敛

#### LightGBM 的优势

| 特性 | 说明 | 优势 |
|------|------|------|
| **Leaf-wise 生长** | 每次分裂最大增益的叶子 | 更准确，收敛更快 |
| **直方图算法** | 离散化连续特征 | 训练速度快，内存占用低 |
| **GOSS** | 梯度采样 | 保留大梯度样本，减少计算 |
| **EFB** | 互斥特征捆绑 | 减少特征维度 |

### 超参数详解

#### 基础参数

| 参数 | 值 | 说明 | 调优建议 |
|------|-----|------|----------|
| `loss` | `mse` | 损失函数: 均方误差(回归任务) | 固定不变 |
| `num_boost_round` | 1000 | 最多训练1000棵树 | 配合早停使用 |
| `early_stopping_rounds` | 50 | 50轮无改进则停止 | 防止过拟合 |

#### 学习参数

| 参数 | 值 | 范围 | 说明 |
|------|-----|------|------|
| `learning_rate` | 0.2 | [0.01, 0.3] | 学习率，控制每棵树的贡献 |

**影响**:
- 过大: 训练不稳定，易过拟合
- 过小: 需要更多树，训练慢
- 0.2是中等偏高的设置

#### 树结构参数

| 参数 | 值 | 范围 | 说明 |
|------|-----|------|------|
| `max_depth` | 8 | [3, 10] | 树的最大深度 |
| `num_leaves` | 210 | [31, 255] | 叶子节点数量 |

**关系**: `num_leaves` ≤ 2^`max_depth`

**当前配置分析**:
- `num_leaves=210` 远小于 2^8=256
- 意味着每层不是完全分裂，更灵活

**影响**:
- 更大的`num_leaves`: 模型更复杂，表达能力强，但易过拟合
- `num_leaves=210`是较大的值，适合158维特征

#### 采样参数

| 参数 | 值 | 范围 | 说明 |
|------|-----|------|------|
| `subsample` | 0.8789 | (0, 1] | 每棵树使用87.89%的样本 |
| `colsample_bytree` | 0.8879 | (0, 1] | 每棵树使用88.79%的特征 |

**作用**:
- 增加随机性，防止过拟合
- 类似于随机森林的Bagging思想
- 提高模型泛化能力

#### 正则化参数

| 参数 | 值 | 范围 | 说明 |
|------|-----|------|------|
| `lambda_l1` | 205.6999 | [0, ∞) | L1正则化(Lasso) |
| `lambda_l2` | 580.9768 | [0, ∞) | L2正则化(Ridge) |

**作用**:
- 控制模型复杂度
- 防止特征权重过大
- **这里的值非常大**，表示强正则化

**影响分析**:
```python
损失函数 = MSE + lambda_l1 * |权重| + lambda_l2 * 权重²
```

- 大的正则化系数会压缩特征权重
- 使模型更保守，减少对单一特征的依赖
- 适合高维特征(158维)，避免过拟合

#### 性能参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `num_threads` | 20 | 使用20个CPU线程并行训练 |

### 模型训练流程

```python
# 1. 准备数据
train_data = dataset.prepare("train", col_set=["feature", "label"])
valid_data = dataset.prepare("valid", col_set=["feature", "label"])

# 2. 转换为LightGBM格式
dtrain = lgb.Dataset(train_data["feature"], label=train_data["label"])
dvalid = lgb.Dataset(valid_data["feature"], label=valid_data["label"])

# 3. 训练模型
model = lgb.train(
    params,                    # 超参数
    dtrain,                    # 训练集
    num_boost_round=1000,      # 最多1000棵树
    valid_sets=[dtrain, dvalid],  # 监控训练和验证集
    callbacks=[
        lgb.early_stopping(50),    # 早停
        lgb.log_evaluation(20)     # 每20轮打印一次
    ]
)

# 4. 预测
pred = model.predict(test_data["feature"])
```

### 模型输出

- **训练输出**: 每只股票在每个交易日的预测收益率
- **输出格式**: pandas.Series，MultiIndex(datetime, instrument)
- **数值范围**: 通常在[-0.1, 0.1]之间(归一化后的收益率)

---

## 7. 策略: TopkDropoutStrategy

### 配置内容

```yaml
strategy:
    class: TopkDropoutStrategy
    module_path: qlib.contrib.strategy
    kwargs:
        signal: <PRED>      # 模型预测信号
        topk: 50           # 持仓股票数量
        n_drop: 5          # 每期调仓数量
```

### TopkDropoutStrategy 是什么?

一个**动态选股+定期调仓**的投资组合策略，核心思想:
- 持有预测收益最高的Top K只股票
- 每个调仓日替换表现最差的N只股票

### 策略原理

#### 策略流程图

```
每个交易日:
    ↓
获取所有股票的预测信号
    ↓
┌─────────────────────────────┐
│ 当前持仓: 50只股票            │
│ [Stock1, Stock2, ..., Stock50] │
└─────────────────────────────┘
    ↓
识别表现最差的5只 (n_drop=5)
    ↓
从剩余股票中选择预测最高的5只
    ↓
卖出5只 → 买入5只
    ↓
新持仓: 50只股票
```

#### 详细步骤

**步骤1: 获取当前持仓**

```python
current_stocks = ["SH600000", "SH600004", ..., "SH600050"]  # 50只
```

**步骤2: 对持仓股票排序**

```python
# 按模型预测得分排序
last = pred_score.reindex(current_stocks).sort_values(ascending=False)
# 结果: [得分最高的股票, ..., 得分最低的股票]
```

**步骤3: 识别候选买入股票**

```python
# 方法1: method_buy="top" (默认)
# 从不在持仓中的股票里，选择得分最高的 topk+n_drop 只
new_candidates = pred_score[~pred_score.index.isin(last)] \
                    .sort_values(ascending=False) \
                    .head(topk + n_drop)

# 如果持仓不足50只，候选股可以更多
```

**步骤4: 组合排序**

```python
# 将当前持仓和候选股票合并，重新排序
combined = pred_score.reindex(last.union(new_candidates)) \
                     .sort_values(ascending=False)
```

**步骤5: 识别卖出股票**

```python
# method_sell="bottom" (默认)
# 找出在combined排名后n_drop名，且在当前持仓中的股票
sell_stocks = last[last.isin(combined.tail(n_drop))]
# 这些是持仓中得分最低的，需要卖出
```

**步骤6: 确定买入股票**

```python
# 买入数量 = 卖出数量 + (topk - 当前持仓数)
buy_count = len(sell_stocks) + (topk - len(last))
buy_stocks = new_candidates[:buy_count]
```

**步骤7: 生成交易订单**

```python
# 卖出订单
for stock in sell_stocks:
    if 可交易 and 持有天数 >= hold_thresh:
        创建卖单(stock, amount=持有数量)

# 买入订单
cash_per_stock = 可用资金 * risk_degree / len(buy_stocks)
for stock in buy_stocks:
    if 可交易:
        amount = cash_per_stock / 股价
        创建买单(stock, amount)
```

### 参数详解

#### topk: 50

**含义**: 目标持仓股票数量

**影响**:
- **分散化**: 50只股票较为分散，降低单股风险
- **集中度**: 不会太分散，保留了预测信号的价值
- **交易成本**: 股票数适中，不会产生过多小额交易

**常见设置**:
- 小盘股: 20-30只(流动性考虑)
- 大盘股: 50-100只(更分散)
- 沪深300成分: 50只约占1/6

#### n_drop: 5

**含义**: 每个调仓日最多替换5只股票

**影响**:
- **换手率**: n_drop/topk = 5/50 = 10%每期
- **交易成本**: 较低，减少频繁交易
- **信号捕捉**: 保留大部分持仓，只替换最差的

**调仓频率计算**:
```python
# 假设每日调仓
日均换手率 = 10%
年化换手率 = 10% × 252 = 2520%

# 实际可能是周度或月度调仓
周度调仓年化换手率 = 10% × 52 = 520%
月度调仓年化换手率 = 10% × 12 = 120%
```

#### method_sell: "bottom"

**含义**: 卖出得分最低的股票

**替代方案**:
- `"random"`: 随机卖出n_drop只 → 对照实验，验证选股能力

#### method_buy: "top"

**含义**: 买入得分最高的股票

**替代方案**:
- `"random"`: 从TopK候选中随机买入 → 减少过拟合风险

### 高级特性

#### hold_thresh: 1 (默认)

**含义**: 最小持有天数

```python
# 股票必须持有至少1天才能卖出
if current_position.get_stock_count(stock) < self.hold_thresh:
    continue  # 跳过，不卖出
```

**作用**: 防止日内频繁买卖同一只股票

#### only_tradable: False (默认)

**含义**: 是否只考虑可交易股票

```python
if only_tradable:
    # 过滤掉停牌、涨跌停、ST股票等
    tradable_stocks = [s for s in stocks if is_tradable(s)]
else:
    # 生成订单，由交易所判断是否可成交
    pass
```

**建议**: 回测时设为`False`(更真实)，实盘时设为`True`

#### forbid_all_trade_at_limit: True (默认)

**含义**: 涨跌停时禁止所有交易

**影响**:
- `True`: 涨停不卖出，跌停不买入(更保守)
- `False`: 涨停可以卖出，跌停可以买入(更激进)

### 风险管理

#### risk_degree: 0.95 (默认)

**含义**: 使用95%的可用资金

```python
每只股票投资金额 = 总资金 × 0.95 / 50 = 总资金 × 1.9%
```

**作用**:
- 保留5%现金应对极端情况
- 避免因资金不足导致无法调仓

### 策略示例

假设某日:
- 当前持仓: 50只股票(A1-A50)
- 预测得分: A1(0.05), A2(0.04), ..., A48(-0.02), A49(-0.03), A50(-0.04)
- 候选股票: B1(0.06), B2(0.055), B3(0.045), ..., B100(-0.05)

**执行步骤**:

1. **识别卖出股票**:
   - 持仓中得分最低5只: A46, A47, A48, A49, A50
   - 但还要与候选股比较...

2. **组合排序**:
   - 合并持仓和候选: [B1(0.06), B2(0.055), A1(0.05), ...]
   - 排序后最低5只可能是: A48, A49, A50, B98, B99
   - 实际卖出: A48, A49, A50 (只卖持仓内的)

3. **买入股票**:
   - 买入B1, B2, B3 (得分最高的3只候选股)

4. **新持仓**:
   - 保留: A1-A47 (47只)
   - 新增: B1, B2, B3 (3只)
   - 共50只

---

## 8. 回测配置

### 配置内容

```yaml
backtest:
    start_time: 2017-01-01
    end_time: 2020-08-01
    account: 100000000          # 初始资金1亿
    benchmark: *benchmark       # SH000300 (沪深300指数)
    exchange_kwargs:
        limit_threshold: 0.095  # 涨跌停阈值9.5%
        deal_price: close       # 成交价格为收盘价
        open_cost: 0.0005       # 买入费率0.05%
        close_cost: 0.0015      # 卖出费率0.15%
        min_cost: 5             # 最小交易费用5元
```

### 回测时间范围

```yaml
start_time: 2017-01-01
end_time: 2020-08-01
```

**为什么是2017-2020?**
- 与测试集(test segment)保持一致
- 模型未见过这段时间的数据
- 模拟真实的"训练后预测"场景

**回测时间线**:
```
2008────2014│2015─2016│2017────────2020
  训练集(7年)  验证集    回测/测试集(3.7年)
                         ↑
                    策略在这里运行
```

### 初始资金

```yaml
account: 100000000  # 1亿人民币
```

**为什么选1亿?**
- 机构级别资金规模
- 足够大以忽略小额交易限制(最小100股)
- 能够均匀分配到50只股票

**资金分配**:
```python
每只股票 = 1亿 × 0.95 / 50 = 190万元
假设股价30元/股: 约63,333股 → 向下取整633手
```

### 基准指数

```yaml
benchmark: SH000300  # 沪深300指数
```

**作用**:
- 评估策略是否跑赢市场
- 计算超额收益 (Alpha)
- 风险调整指标参考

**为什么用沪深300?**
- 股票池是`csi300`(沪深300成分股)
- 基准应与股票池匹配
- 沪深300代表A股大盘蓝筹表现

### 交易所设置

#### limit_threshold: 0.095

**含义**: 涨跌停阈值为9.5%

**中国A股规则**:
- 主板/中小板: ±10%
- 科创板/创业板: ±20%
- ST股票: ±5%

**9.5%的设计原因**:
- 略小于10%的理论涨跌停
- 考虑到实际可能在9.98%就封涨停
- 给出缓冲空间

**影响**:
```python
if abs(今日收益率) > 0.095:
    该股票不可交易  # 涨跌停，无法成交
```

#### deal_price: close

**含义**: 以收盘价成交

**回测假设**:
- 收到信号后，以当日收盘价交易
- **理想化假设**: 实际可能无法保证以收盘价成交

**其他选项**:
- `open`: 开盘价(更保守，可操作性强)
- `vwap`: 成交量加权均价(更真实)
- `close`: 收盘价(信号与执行同一天)

**实际建议**:
- 日频策略: 用第二天`open`更真实
- 这里用`close`可能高估了策略表现

#### 交易成本

| 费用类型 | 参数 | 费率 | 说明 |
|---------|------|------|------|
| 买入佣金 | `open_cost` | 0.05% | 券商佣金+交易所费用 |
| 卖出佣金 | `close_cost` | 0.15% | 佣金+印花税(0.1%)+其他 |
| 最小费用 | `min_cost` | 5元 | 单笔交易最低收费 |

**中国A股实际成本**:
- 买入: 佣金(万2.5-万3) + 过户费(万0.2) ≈ 0.03%
- 卖出: 佣金 + 印花税(0.1%) + 过户费 ≈ 0.13%
- 这里略微高估了成本(更保守)

**成本计算示例**:
```python
# 买入100万元股票
买入成本 = 1,000,000 × 0.0005 = 500元
实际扣费 = max(500, 5) = 500元

# 卖出100万元股票
卖出成本 = 1,000,000 × 0.0015 = 1,500元
实际扣费 = max(1,500, 5) = 1,500元

# 一买一卖总成本
总成本 = 500 + 1,500 = 2,000元
成本率 = 2,000 / 1,000,000 = 0.2%
```

**对策略的影响**:
```python
# 假设年化换手率500%
年化交易成本 = 500% × 0.2% = 1%
# 会显著降低策略收益
```

### 回测引擎

Qlib使用事件驱动的回测引擎:

```python
for trade_date in trading_calendar:
    # 1. 获取市场数据
    market_data = exchange.get_data(trade_date)

    # 2. 策略生成交易决策
    orders = strategy.generate_trade_decision()

    # 3. 交易所撮合成交
    for order in orders:
        if exchange.check_order(order):  # 检查涨跌停、停牌等
            trade_val, trade_cost, trade_price = exchange.deal_order(order)
            position.update(order, trade_val, trade_cost)

    # 4. 更新持仓市值
    position.update_value(market_data)

    # 5. 记录净值
    account.record(date=trade_date, value=position.total_value)
```

---

## 9. 记录器配置

### 配置内容

```yaml
record:
    - class: SignalRecord           # 信号记录器
      module_path: qlib.workflow.record_temp
      kwargs:
          model: <MODEL>
          dataset: <DATASET>

    - class: SigAnaRecord           # 信号分析记录器
      module_path: qlib.workflow.record_temp
      kwargs:
          ana_long_short: False
          ann_scaler: 252

    - class: PortAnaRecord          # 组合分析记录器
      module_path: qlib.workflow.record_temp
      kwargs:
          config: *port_analysis_config
```

### 记录器架构

```
Experiment (实验)
    ├── Recorder (记录器)
    │   ├── Parameters (参数)
    │   ├── Metrics (指标)
    │   └── Artifacts (产物)
    │       ├── SignalRecord
    │       │   ├── pred.pkl (预测信号)
    │       │   └── label.pkl (真实标签)
    │       ├── SigAnaRecord
    │       │   ├── IC分析
    │       │   ├── 信息比率
    │       │   └── 分组收益
    │       └── PortAnaRecord
    │           ├── backtest结果
    │           ├── 净值曲线
    │           └── 风险指标
    └── MLflow Integration
```

### 9.1 SignalRecord - 信号记录器

**功能**: 保存模型的预测信号和真实标签

#### 记录内容

```python
# pred.pkl - 预测信号
                              score
datetime   instrument
2017-01-03 SH600000         0.0123
           SH600004         0.0045
           SH600005        -0.0032
           ...
2020-08-01 SH603999         0.0087

# label.pkl - 真实标签
                              LABEL0
datetime   instrument
2017-01-03 SH600000         0.0156
           SH600004         0.0012
           SH600005        -0.0089
           ...
2020-08-01 SH603999         0.0104
```

#### 使用场景

```python
# 加载记录
from qlib.workflow import R

recorder = R.get_recorder()
pred = recorder.load_object("pred.pkl")
label = recorder.load_object("label.pkl")

# 自定义分析
correlation = pred.corrwith(label)
print(f"平均相关性: {correlation.mean()}")
```

### 9.2 SigAnaRecord - 信号分析记录器

**功能**: 分析预测信号的质量

#### 参数说明

```yaml
ana_long_short: False   # 不分析多空组合(纯多头策略)
ann_scaler: 252         # 年化因子(252个交易日)
```

#### 核心指标

##### IC (Information Coefficient) - 信息系数

**定义**: 预测值与真实值的相关系数

```python
IC_t = corr(pred_t, label_t)  # 每个时间点t的IC

# 示例计算
某日IC = corr(
    [0.02, 0.01, -0.01, 0.03, ...],  # 预测收益率
    [0.03, 0.005, -0.02, 0.025, ...]  # 实际收益率
)
```

**IC值含义**:
- IC > 0: 预测方向正确
- IC = 0: 预测无效
- IC < 0: 预测反向(!)
- |IC| > 0.05: 优秀
- |IC| > 0.03: 良好
- |IC| < 0.02: 较弱

##### ICIR (IC Information Ratio) - IC信息比率

**定义**: IC的均值除以IC的标准差

```python
ICIR = mean(IC) / std(IC)
```

**含义**: 预测的稳定性
- ICIR > 2: 非常稳定
- ICIR > 1: 较稳定
- ICIR < 0.5: 不稳定

**为什么重要?**
- 高IC但不稳定 → 难以实现
- 稳定的IC → 可靠的策略

##### Rank IC - 排序信息系数

**定义**: 基于排名的相关性(Spearman)

```python
Rank_IC = spearman_corr(rank(pred), rank(label))
```

**优势**:
- 对异常值不敏感
- 更关注相对排序而非绝对值
- 适合选股策略

#### 分组回测

将股票按预测分成N组(通常5或10组):

```
第1组(预测最高) → 平均收益: 2%
第2组           → 平均收益: 1%
第3组           → 平均收益: 0%
第4组           → 平均收益: -0.5%
第5组(预测最低) → 平均收益: -1.5%
```

**分组收益差**(Group 1 - Group 5):
```python
long_short_return = return_group1 - return_group5
                  = 2% - (-1.5%) = 3.5%
```

**含义**: 做多最高分组，做空最低分组的收益

#### 累计IC曲线

```python
cumulative_IC = IC.cumsum()

# 绘制
plt.plot(cumulative_IC)
plt.title("Cumulative IC")
plt.xlabel("Time")
plt.ylabel("Cumulative IC")
```

**理想形态**: 单调上升

### 9.3 PortAnaRecord - 组合分析记录器

**功能**: 回测策略并分析投资组合表现

#### 记录内容

##### 1. 回测报告 (report_normal.pkl)

```python
{
    'total_return': 0.456,      # 总收益率45.6%
    'annual_return': 0.121,     # 年化收益率12.1%
    'max_drawdown': -0.235,     # 最大回撤23.5%
    'sharpe_ratio': 1.34,       # 夏普比率
    'information_ratio': 0.87,  # 信息比率
    'win_rate': 0.523,          # 胜率52.3%
    ...
}
```

##### 2. 持仓明细 (positions_normal.pkl)

```python
                              amount    price      value   weight
datetime   instrument
2017-01-03 SH600000           10000    15.23     152300    0.019
           SH600004           12000    25.67     308040    0.038
           ...
2017-01-04 SH600000           10000    15.45     154500    0.019
           ...
```

##### 3. 风险分析 (portfolio_analysis.pkl)

```python
{
    'annual_volatility': 0.18,     # 年化波动率18%
    'downside_risk': 0.12,         # 下行风险
    'calmar_ratio': 0.51,          # 卡玛比率
    'sortino_ratio': 1.67,         # 索提诺比率
    'max_consecutive_loss': 5,     # 最长连续亏损天数
    ...
}
```

#### 关键绩效指标详解

##### 收益指标

| 指标 | 公式 | 说明 |
|------|------|------|
| 总收益率 | (期末-期初)/期初 | 整个回测期收益 |
| 年化收益率 | (1+总收益)^(252/天数) - 1 | 折算为年化 |
| 超额收益 | 策略收益 - 基准收益 | 相对基准的Alpha |

##### 风险指标

| 指标 | 公式 | 说明 |
|------|------|------|
| 最大回撤 | max(peak - trough) / peak | 最大亏损幅度 |
| 波动率 | std(daily_return) × √252 | 年化标准差 |
| 下行波动 | std(负收益) × √252 | 只考虑亏损时的波动 |

##### 风险调整收益

| 指标 | 公式 | 优秀阈值 | 说明 |
|------|------|---------|------|
| 夏普比率 | (年化收益-无风险利率)/年化波动 | >1 | 单位风险收益 |
| 信息比率 | 超额收益/跟踪误差 | >0.5 | Alpha质量 |
| 卡玛比率 | 年化收益/最大回撤 | >0.5 | 收益回撤比 |
| 索提诺比率 | (年化收益-无风险利率)/下行波动 | >1 | 只惩罚下行风险 |

##### 其他指标

| 指标 | 说明 |
|------|------|
| 胜率 | 盈利天数/总交易天数 |
| 盈亏比 | 平均盈利/平均亏损 |
| 换手率 | 年化交易金额/平均资产 |
| 持仓集中度 | 前N只股票权重之和 |

#### 可视化输出

记录器会生成以下图表:

1. **净值曲线**
```
净值
  ↑
  │    ╱─╲    ╱──╲
  │   ╱   ╲  ╱    ╲  ╱
  │  ╱     ╲╱      ╲╱
  └─────────────────→ 时间
   2017           2020
```

2. **回撤曲线**
```
回撤%
  0├────╲    ╱────────
    │     ╲  ╱
-10 │      ╲╱ ← 最大回撤
    │
-20 │
    └──────────────→ 时间
```

3. **滚动收益**
```
月度收益
  ↑
5%│ █  █ █
  │ █ ██ █ █
0 │─█─██─█─█──
  │  █  █ █
-5│     █
  └──────────────→ 月份
```

### 记录器执行顺序

```python
# 1. SignalRecord
pred = model.predict(dataset, segment="test")
label = dataset.prepare("test", col_set="label")
recorder.save_objects(pred=pred, label=label)

# 2. SigAnaRecord (依赖SignalRecord)
pred = recorder.load_object("pred.pkl")
label = recorder.load_object("label.pkl")
ic_analysis = calculate_ic(pred, label)
recorder.save_objects(sig_analysis=ic_analysis)

# 3. PortAnaRecord (依赖SignalRecord)
pred = recorder.load_object("pred.pkl")
backtest_result = run_backtest(pred, strategy, exchange)
recorder.save_objects(portfolio_analysis=backtest_result)
```

---

## 10. 完整工作流执行流程

### 10.1 命令行执行

```bash
# 基本用法
qrun workflow_config_lightgbm_Alpha158.yaml

# 指定实验名称
qrun workflow_config_lightgbm_Alpha158.yaml --experiment_name "LGB_Alpha158_v1"

# 指定MLflow tracking URI
qrun workflow_config_lightgbm_Alpha158.yaml --uri "file:///path/to/mlruns"
```

### 10.2 Python脚本执行

```python
import qlib
from qlib.workflow import R
from qlib.workflow.record_temp import SignalRecord, SigAnaRecord, PortAnaRecord
from qlib.utils import init_instance_by_config
import yaml

# 1. 初始化Qlib
qlib.init(provider_uri="~/.qlib/qlib_data/cn_data", region="cn")

# 2. 加载配置
with open("workflow_config_lightgbm_Alpha158.yaml") as f:
    config = yaml.safe_load(f)

# 3. 创建数据集
dataset = init_instance_by_config(config["task"]["dataset"])

# 4. 创建并训练模型
model = init_instance_by_config(config["task"]["model"])
model.fit(dataset)

# 5. 开始记录实验
with R.start(experiment_name="LGB_Alpha158"):
    R.log_params(**config["task"]["model"]["kwargs"])  # 记录超参数

    # 6. SignalRecord
    recorder = R.get_recorder()
    sr = SignalRecord(model=model, dataset=dataset, recorder=recorder)
    sr.generate()

    # 7. SigAnaRecord
    sar = SigAnaRecord(recorder=recorder, ana_long_short=False, ann_scaler=252)
    sar.generate()

    # 8. PortAnaRecord
    par = PortAnaRecord(recorder=recorder, config=config["port_analysis_config"])
    par.generate()

    # 9. 记录指标
    metrics = recorder.load_object("portfolio_analysis/report_normal.pkl")
    R.log_metrics(**metrics)

print("实验完成! 结果保存在MLflow中")
```

### 10.3 详细执行流程

#### 阶段1: 数据准备 (约1-5分钟)

```
[1/8] 初始化Qlib环境
  ├─ 加载配置: ~/.qlib/qlib_data/cn_data
  ├─ 读取交易日历: 2008-01-01 to 2020-08-01
  └─ 检查数据完整性: ✓

[2/8] 加载数据处理器
  ├─ 类: Alpha158
  ├─ 股票池: csi300 (约300只股票)
  ├─ 时间范围: 2008-01-01 to 2020-08-01
  └─ 加载原始数据: $close, $open, $high, $low, $volume, $vwap

[3/8] 计算特征
  ├─ K线特征: 9个
  ├─ 价格特征: 5个
  ├─ 滚动特征: 144个
  ├─ 总特征数: 158个
  └─ 数据形状: (约900,000行 × 158列)
     # 300只股票 × 3000交易日 ≈ 900,000

[4/8] 数据预处理
  ├─ 处理无穷值: ProcessInf()
  ├─ 标准化: ZScoreNorm()
  │   ├─ 训练期统计: 2008-2014
  │   └─ 应用到全部数据
  ├─ 填充缺失值: Fillna()
  └─ 标签处理:
      ├─ DropnaLabel()
      └─ CSZScoreNorm()
```

#### 阶段2: 模型训练 (约5-30分钟)

```
[5/8] 准备训练数据
  ├─ 训练集: 2008-2014 (约540,000样本)
  ├─ 验证集: 2015-2016 (约150,000样本)
  └─ 测试集: 2017-2020 (约280,000样本)

[6/8] 训练LightGBM
  ├─ 超参数:
  │   ├─ learning_rate: 0.2
  │   ├─ max_depth: 8
  │   ├─ num_leaves: 210
  │   └─ ...
  ├─ 训练进度:
  │   [20]  train: 0.0156  valid: 0.0189
  │   [40]  train: 0.0134  valid: 0.0178
  │   [60]  train: 0.0121  valid: 0.0172
  │   ...
  │   [420] train: 0.0089  valid: 0.0169
  └─ 早停: 第420轮, 最佳验证: 0.0165

[7/8] 模型预测
  ├─ 测试集预测: 280,000个预测值
  └─ 预测范围: [-0.08, 0.12]
```

#### 阶段3: 策略回测 (约2-10分钟)

```
[8/8] 回测与分析

SignalRecord (1/3):
  ├─ 保存预测: pred.pkl (280,000行)
  └─ 保存标签: label.pkl (280,000行)

SigAnaRecord (2/3):
  ├─ 计算IC:
  │   ├─ IC均值: 0.0342
  │   ├─ IC标准差: 0.156
  │   ├─ ICIR: 0.219
  │   └─ Rank IC: 0.0389
  ├─ 分组回测:
  │   ├─ Group 1 (Top 20%): 年化收益 18.3%
  │   ├─ Group 2: 年化收益 12.1%
  │   ├─ Group 3: 年化收益 7.8%
  │   ├─ Group 4: 年化收益 3.2%
  │   └─ Group 5 (Bottom 20%): 年化收益 -5.6%
  └─ 多空组合: 18.3% - (-5.6%) = 23.9%

PortAnaRecord (3/3):
  ├─ 策略参数:
  │   ├─ topk: 50
  │   ├─ n_drop: 5
  │   └─ 初始资金: 1亿
  ├─ 回测执行:
  │   ├─ 交易天数: 928天
  │   ├─ 总交易笔数: 4,640笔
  │   └─ 换手率: 520% (年化)
  ├─ 绩效指标:
  │   ├─ 总收益: 45.6%
  │   ├─ 年化收益: 12.1%
  │   ├─ 基准收益(沪深300): 8.7%
  │   ├─ 超额收益: 3.4%
  │   ├─ 最大回撤: -23.5%
  │   ├─ 夏普比率: 1.34
  │   ├─ 信息比率: 0.87
  │   └─ 卡玛比率: 0.51
  └─ 保存报告:
      ├─ report_normal.pkl
      ├─ positions_normal.pkl
      └─ portfolio_analysis.pkl

实验完成! ✓
  ├─ 实验ID: 20231215_143022_abc123
  ├─ 记录路径: ~/mlruns/1/abc123
  └─ MLflow UI: mlflow ui --port 5000
```

### 10.4 结果查看

#### 命令行查看

```bash
# 启动MLflow UI
mlflow ui --port 5000

# 浏览器访问
# http://localhost:5000
```

#### Python查看

```python
from qlib.workflow import R

# 列出所有实验
experiments = R.list_experiments()

# 获取特定记录器
recorder = R.get_recorder(recorder_id="abc123")

# 加载结果
pred = recorder.load_object("pred.pkl")
report = recorder.load_object("portfolio_analysis/report_normal.pkl")

# 打印关键指标
print(f"年化收益: {report['annual_return']:.2%}")
print(f"夏普比率: {report['sharpe_ratio']:.2f}")
print(f"最大回撤: {report['max_drawdown']:.2%}")

# 可视化
import matplotlib.pyplot as plt
positions = recorder.load_object("positions_normal.pkl")
positions['value'].sum(level='datetime').plot()
plt.title("Portfolio Value")
plt.show()
```

### 10.5 超参数调优工作流

```python
from qlib.workflow import R
from qlib.contrib.tuner import OptunaSearcher

# 定义搜索空间
search_space = {
    "learning_rate": {"type": "float", "low": 0.01, "high": 0.3},
    "num_leaves": {"type": "int", "low": 31, "high": 255},
    "max_depth": {"type": "int", "low": 3, "high": 15},
    "lambda_l1": {"type": "float", "low": 0, "high": 500},
    "lambda_l2": {"type": "float", "low": 0, "high": 1000},
}

# 定义目标函数
def objective(params):
    # 更新配置
    config["task"]["model"]["kwargs"].update(params)

    # 训练模型
    model = init_instance_by_config(config["task"]["model"])
    model.fit(dataset)

    # 评估
    pred = model.predict(dataset, segment="valid")
    label = dataset.prepare("valid", col_set="label", data_key="learn")
    ic = pred.corr(label)

    return ic  # 优化目标: 最大化IC

# 运行优化
searcher = OptunaSearcher(
    search_space=search_space,
    objective=objective,
    n_trials=100,
    direction="maximize"
)

best_params = searcher.search()
print(f"最佳参数: {best_params}")
```

---

## 总结

### 工作流亮点

1. **完整性**: 涵盖数据→特征→模型→策略→回测→分析全流程
2. **可配置**: YAML配置文件,易于调整参数
3. **可重现**: MLflow记录所有参数和结果
4. **模块化**: 各组件独立,易于替换和扩展

### 关键组件总结

| 组件 | 作用 | 输入 | 输出 |
|------|------|------|------|
| Alpha158 | 特征工程 | 原始价量数据 | 158维特征 |
| DatasetH | 数据管理 | 特征+标签 | 训练/验证/测试集 |
| LGBModel | 预测模型 | 训练数据 | 收益率预测 |
| TopkDropout | 投资组合策略 | 预测信号 | 交易订单 |
| Backtest | 回测引擎 | 订单+市场数据 | 净值曲线 |
| Records | 实验记录 | 预测+回测结果 | 分析报告 |

### 性能基准

基于历史经验,这个配置通常能达到:

| 指标 | 典型范围 |
|------|----------|
| IC均值 | 0.02 - 0.05 |
| ICIR | 0.1 - 0.5 |
| 年化收益 | 10% - 20% |
| 夏普比率 | 1.0 - 2.0 |
| 最大回撤 | 15% - 30% |
| 信息比率 | 0.5 - 1.5 |

### 改进方向

1. **特征增强**:
   - 添加基本面因子(市盈率、ROE等)
   - 情绪因子(新闻、社交媒体)
   - 另类数据(卫星图像、信用卡数据)

2. **模型升级**:
   - 尝试深度学习(LSTM、Transformer)
   - 集成学习(多模型融合)
   - 元学习(市场状态自适应)

3. **策略优化**:
   - 动态调整topk和n_drop
   - 加入风险约束(波动率、因子暴露)
   - 多周期组合(日度+周度)

4. **执行改进**:
   - 更真实的滑点模型
   - 算法交易(VWAP、TWAP)
   - 订单拆分和隐藏

### 参考资源

- **Qlib文档**: https://qlib.readthedocs.io/
- **Alpha158论文**: https://arxiv.org/abs/2009.11189
- **LightGBM文档**: https://lightgbm.readthedocs.io/
- **MLflow文档**: https://mlflow.org/docs/latest/index.html

---

**文档结束**

*本文档详细解释了Qlib工作流配置的各个组件,希望能帮助您理解和使用Qlib进行量化投资研究。*
