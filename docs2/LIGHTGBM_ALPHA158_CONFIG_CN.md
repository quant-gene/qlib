# LightGBM + Alpha158 配置详解

> 配置文件：`examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml`
> 本文档结合源码，重点说明**模型的输入(input)与输出(output)分别是什么**。

---

## 一、配置文件整体结构

该 YAML 描述了一个完整的 qlib 工作流(workflow)，分为四大块：

| 块 | 作用 |
|----|------|
| `qlib_init` | 初始化 qlib，指定数据源 |
| `market` / `benchmark` | 股票池与基准 |
| `data_handler_config` | 数据处理时间范围(锚点) |
| `port_analysis_config` | 回测/组合分析配置(锚点) |
| `task` | 核心：模型 + 数据集 + 记录器 |

`&xxx` 是 YAML 锚点(anchor)，`*xxx` 是引用(alias)，用于复用配置。

---

## 二、逐项解释

### 1. qlib_init —— 初始化

```yaml
qlib_init:
    provider_uri: "~/.qlib/qlib_data/cn_data"   # 本地行情数据目录
    region: cn                                   # 中国市场(影响交易日历、涨跌停规则等)
```

### 2. market / benchmark —— 股票池与基准

```yaml
market: &market csi300        # 沪深300成分股作为股票池
benchmark: &benchmark SH000300 # 沪深300指数作为业绩基准
```

### 3. data_handler_config —— 数据时间范围(锚点)

```yaml
data_handler_config: &data_handler_config
    start_time: 2008-01-01     # 数据整体起始
    end_time: 2020-08-01       # 数据整体结束
    fit_start_time: 2008-01-01 # 预处理器(如归一化)拟合统计量的起始
    fit_end_time: 2014-12-31   # 预处理器拟合统计量的结束(防止未来信息泄露)
    instruments: *market       # = csi300
```

> `fit_start_time` / `fit_end_time` 很关键：归一化(ZScoreNorm)的均值/方差只在这段历史上计算，
> 然后应用到全部数据，避免用到测试期的统计信息(数据泄露)。

### 4. port_analysis_config —— 回测组合分析(锚点)

```yaml
port_analysis_config: &port_analysis_config
    strategy:
        class: TopkDropoutStrategy        # 选股策略：买入预测分最高的topk只
        kwargs:
            signal: <PRED>                # 用模型预测结果作为选股信号
            topk: 50                      # 持有50只
            n_drop: 5                     # 每期最多换出5只(降低换手率)
    backtest:
        start_time: 2017-01-01            # 回测区间(=测试集)
        end_time: 2020-08-01
        account: 100000000                # 初始资金1亿
        benchmark: *benchmark
        exchange_kwargs:
            limit_threshold: 0.095        # 涨跌停阈值(±9.5%不可成交)
            deal_price: close             # 以收盘价成交
            open_cost: 0.0005             # 买入手续费 0.05%
            close_cost: 0.0015            # 卖出手续费 0.15%(含印花税)
            min_cost: 5                   # 单笔最低费用5元
```

---

## 三、task —— 核心配置

### 3.1 model —— LightGBM 模型

```yaml
model:
    class: LGBModel
    module_path: qlib.contrib.model.gbdt
    kwargs:
        loss: mse              # 损失函数：均方误差(回归任务)
        colsample_bytree: 0.8879  # 每棵树随机采样的特征比例
        learning_rate: 0.2        # 学习率
        subsample: 0.8789         # 每棵树随机采样的样本比例
        lambda_l1: 205.6999       # L1正则
        lambda_l2: 580.9768       # L2正则
        max_depth: 8              # 树最大深度
        num_leaves: 210           # 最大叶子数
        num_threads: 20           # 并行线程数
```

对应源码 `qlib/contrib/model/gbdt.py`：

```python
class LGBModel(ModelFT, LightGBMFInt):
    def fit(self, dataset: DatasetH, ...):
        # 从 dataset 取出训练/验证集，调用 lgb.train() 训练
    def predict(self, dataset, segment="test") -> pd.Series:
        # 对指定 segment 推理，返回每只股票每个交易日的预测分
```

- `loss: mse` 表明这是**回归任务**：预测一个连续的收益率数值，而不是分类。

### 3.2 dataset —— 数据集与特征处理器

```yaml
dataset:
    class: DatasetH
    kwargs:
        handler:
            class: Alpha158               # 关键：特征工程方案
            module_path: qlib.contrib.data.handler
            kwargs: *data_handler_config
        segments:                          # 数据集切分(时间序列不可乱切)
            train: [2008-01-01, 2014-12-31]  # 训练集
            valid: [2015-01-01, 2016-12-31]  # 验证集(早停用)
            test:  [2017-01-01, 2020-08-01]  # 测试集(回测用)
```

`Alpha158` 是整个流程中决定**输入**的部分，详见第四节。

### 3.3 record —— 结果记录与评估

```yaml
record:
    - class: SignalRecord    # 1) 跑预测，保存 pred.pkl(预测分) + label.pkl(真实标签)
    - class: SigAnaRecord    # 2) 信号分析：计算 IC / ICIR / Rank IC / Rank ICIR
      kwargs:
        ana_long_short: False
        ann_scaler: 252      # 年化系数(一年约252个交易日)
    - class: PortAnaRecord   # 3) 组合回测：按策略选股，输出收益/夏普/最大回撤等
      kwargs:
        config: *port_analysis_config
```

---

## 四、【重点】模型的输入与输出

### 4.1 输入(Input / Features)= Alpha158 的 158 个因子

源码 `qlib/contrib/data/handler.py` 中 `Alpha158.get_feature_config()`：

```python
def get_feature_config(self):
    conf = {
        "kbar": {},                                    # K线形态类
        "price": {"windows": [0],
                  "feature": ["OPEN", "HIGH", "LOW", "VWAP"]},  # 当日价格类
        "rolling": {},                                 # 滚动窗口技术指标类
    }
    return Alpha158DL.get_feature_config(conf)
```

实际特征由 `qlib/contrib/data/loader.py` 的 `Alpha158DL.get_feature_config()` 生成。
所有特征都是 qlib 表达式，最终在原始行情字段(`$open/$high/$low/$close/$volume/$vwap`)上计算得出。

**输入特征三大类：**

#### (1) KBAR —— K线形态特征(9个)

刻画单根K线的形态，全部用当日 OHLC 计算，已归一化(除以 open 或振幅)：

| 名称 | 表达式 | 含义 |
|------|--------|------|
| KMID | `($close-$open)/$open` | 实体涨跌幅 |
| KLEN | `($high-$low)/$open` | K线全长(振幅) |
| KMID2| `($close-$open)/($high-$low+1e-12)` | 实体占振幅比例 |
| KUP  | `($high-Greater($open,$close))/$open` | 上影线 |
| KUP2 | `($high-Greater($open,$close))/($high-$low+1e-12)` | 上影线占振幅比例 |
| KLOW | `(Less($open,$close)-$low)/$open` | 下影线 |
| KLOW2| `(Less($open,$close)-$low)/($high-$low+1e-12)` | 下影线占振幅比例 |
| KSFT | `(2*$close-$high-$low)/$open` | 收盘价在区间中的偏移 |
| KSFT2| `(2*$close-$high-$low)/($high-$low+1e-12)` | 偏移占振幅比例 |

#### (2) Price —— 当日价格特征(4个)

`windows=[0]` 表示只取当日，除以 `$close` 做归一化：

| 名称 | 表达式 | 含义 |
|------|--------|------|
| OPEN0 | `$open/$close` | 开盘价/收盘价 |
| HIGH0 | `$high/$close` | 最高价/收盘价 |
| LOW0  | `$low/$close`  | 最低价/收盘价 |
| VWAP0 | `$vwap/$close` | 成交均价/收盘价 |

#### (3) Rolling —— 滚动窗口技术指标(约145个)

`"rolling": {}` 使用默认配置：窗口 `windows=[5,10,20,30,60]`，启用全部算子。
每个算子 × 5 个窗口生成多个特征，主要包括：

| 算子组 | 名称示例 | 含义 |
|--------|----------|------|
| 趋势 | ROC/MA/STD/BETA/RSQR/RESI | 变化率、均线、波动率、斜率、回归拟合度、残差 |
| 极值 | MAX/MIN/QTLU/QTLD | 区间高低点、上下分位数 |
| 位置 | RANK/RSV | 当前价在历史中的分位、随机指标 |
| 动量 | IMAX/IMIN/IMXD | Aroon类，距离最高/最低价的天数 |
| 量价相关 | CORR/CORD | 价量相关性、价量变化率相关性 |
| 涨跌统计 | CNTP/CNTN/CNTD | 上涨/下跌天数占比及其差 |
| 类RSI | SUMP/SUMN/SUMD | 累计涨幅/跌幅占比及其差 |
| 成交量 | VMA/VSTD/WVMA/VSUMP/VSUMN/VSUMD | 量均线、量波动、量加权波动、量涨跌统计 |

> 合计约 **158 个特征**(故名 Alpha**158**)。每个样本 = (某只股票, 某个交易日)，
> 输入是该样本对应的 158 维特征向量。

**输入数据的形状：** `DataFrame`，行索引为 `(datetime, instrument)` 多级索引，
列为 158 个特征列；模型 `fit/predict` 时取其 `.values` 作为特征矩阵 `X`。

### 4.2 输出(Output / Label)= 未来一日收益率

源码 `Alpha158.get_label_config()`：

```python
def get_label_config(self):
    return ["Ref($close, -2)/Ref($close, -1) - 1"], ["LABEL0"]
```

- **标签(训练目标)：** `Ref($close, -2)/Ref($close, -1) - 1`，列名 `LABEL0`
- `Ref($close, -1)` = 下一个交易日(T+1)收盘价
- `Ref($close, -2)` = 后第二个交易日(T+2)收盘价
- 含义：**T+1 买入、T+2 卖出的隔日收益率**(即"次日到次次日"的涨跌幅)。

> 为什么用 T+1→T+2 而不是 T→T+1？
> 因为决策发生在 T 日收盘后，最早只能在 T+1 开盘/收盘建仓，
> 这样设计可避免使用决策时刻尚不可得的价格(防止未来信息泄露)。

#### 常见误区：这个模型预测的是"第二天的收盘价"吗？

**不是。** 模型预测的不是某个绝对价格，而是**隔日收益率**。

站在决策日 T，把标签 `Ref($close, -2)/Ref($close, -1) - 1` 拆开：

- `Ref($close, -1)` = T+1 收盘价
- `Ref($close, -2)` = T+2 收盘价
- `LABEL0 = (T+2收盘价 / T+1收盘价) - 1`

所以它预测的是 **"T+1 收盘到 T+2 收盘的涨跌幅"**(一个百分比收益率)，
值是类似 `+0.02`(涨2%)、`-0.01`(跌1%)这样的比率。

需要澄清的三点：

1. **是收益率，不是价格。** 收益率量纲无关、跨股票可比，便于横截面排序选股；
   直接预测收盘价的绝对值既难训练，也无法在不同股票间横向比较。

2. **不是 T→T+1，而是 T+1→T+2。** 决策在 T 日收盘后做出，最早 T+1 才能建仓，
   用 T+1→T+2 的收益可避免使用决策时尚不可得的价格(防止未来信息泄露)。

3. **训练时标签还会被横截面归一化。** `learn_processors` 默认包含
   `CSZScoreNorm`(对 label 按交易日做横截面 ZScore)，因此模型实际学习的是
   "该股票当期收益在全市场中的相对强弱排名"。预测分越高代表越看好，
   正好用于 `TopkDropoutStrategy` 挑选预测分最高的 50 只股票。

> 一句话：模型输出是"未来隔日的相对收益强弱分"，用于排序选股，而非预测具体股价。

### 4.3 模型预测产出(Prediction)

- `model.predict()` 返回一个 `pd.Series`，索引同样是 `(datetime, instrument)`，
  值为对每只股票未来收益率的**预测分**(连续值，越大表示越看好)。
- 该预测分被保存为 `pred.pkl`，并作为 `<PRED>` 信号传给 `TopkDropoutStrategy`：
  每个交易日买入预测分最高的 50 只股票，从而完成"特征 → 预测 → 选股 → 回测"的闭环。

---

## 五、端到端数据流总结

```
原始行情($open/$high/$low/$close/$volume/$vwap)
        │
        ▼  Alpha158 特征工程
输入 X：158维因子(KBAR + Price + Rolling)        ←── 模型输入
        │
        ▼  LGBModel.fit (train+valid, mse回归)
训练好的 LightGBM 模型
        │
        ▼  LGBModel.predict (test)
输出 ŷ：每股每日的预测收益分(pred.pkl)            ←── 模型输出
        │
        ├─ 对照真实标签 LABEL0(T+1→T+2收益率) → SigAnaRecord 算 IC/ICIR
        └─ 作为信号 → TopkDropoutStrategy 选股 → PortAnaRecord 回测收益
```

**一句话总结：**
- **输入** = Alpha158 的 158 个量价技术因子(每只股票每个交易日一个 158 维向量)；
- **输出** = 对 `LABEL0`(T+1 买入、T+2 卖出的隔日收益率)的回归预测值，用作选股信号。
