[![Python 版本](https://img.shields.io/pypi/pyversions/pyqlib.svg?logo=python&logoColor=white)](https://pypi.org/project/pyqlib/#files)
[![平台](https://img.shields.io/badge/platform-linux%20%7C%20windows%20%7C%20macos-lightgrey)](https://pypi.org/project/pyqlib/#files)
[![PyPI 版本](https://img.shields.io/pypi/v/pyqlib)](https://pypi.org/project/pyqlib/#history)
[![上传 Python 包](https://github.com/microsoft/qlib/workflows/Upload%20Python%20Package/badge.svg)](https://pypi.org/project/pyqlib/)
[![Github Actions 测试状态](https://github.com/microsoft/qlib/workflows/Test/badge.svg?branch=main)](https://github.com/microsoft/qlib/actions)
[![文档状态](https://readthedocs.org/projects/qlib/badge/?version=latest)](https://qlib.readthedocs.io/en/latest/?badge=latest)
[![许可证](https://img.shields.io/pypi/l/pyqlib)](LICENSE)
[![加入聊天 https://gitter.im/Microsoft/qlib](https://badges.gitter.im/Microsoft/qlib.svg)](https://gitter.im/Microsoft/qlib?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## :newspaper: **最新消息!** &nbsp;   :sparkling_heart:

近期发布的功能

### 介绍 <a href="https://github.com/microsoft/RD-Agent"><img src="docs/_static/img/rdagent_logo.png" alt="RD_Agent" style="height: 2em"></a>：基于大语言模型的工业数据驱动研发自主演化智能体

我们很高兴地宣布发布 **RD-Agent**📢，这是一个强大的工具，支持量化投资研发中的自动化因子挖掘和模型优化。

RD-Agent 现已在 [GitHub](https://github.com/microsoft/RD-Agent) 上发布，欢迎您的 star🌟！

要了解更多，请访问我们的 [♾️演示页面](https://rdagent.azurewebsites.net/)。在这里，您可以找到英文和中文的演示视频，帮助您更好地了解 RD-Agent 的场景和使用。

我们为您准备了几个演示视频：
| 场景 | 演示视频 (英文) | 演示视频 (中文) |
| --                      | ------    | ------    |
| 量化因子挖掘 | [链接](https://rdagent.azurewebsites.net/factor_loop?lang=en) | [链接](https://rdagent.azurewebsites.net/factor_loop?lang=zh) |
| 从报告中挖掘量化因子 | [链接](https://rdagent.azurewebsites.net/report_factor?lang=en) | [链接](https://rdagent.azurewebsites.net/report_factor?lang=zh) |
| 量化模型优化 | [链接](https://rdagent.azurewebsites.net/model_loop?lang=en) | [链接](https://rdagent.azurewebsites.net/model_loop?lang=zh) |

- 📃**论文**: [R&D-Agent-Quant: A Multi-Agent Framework for Data-Centric Factors and Model Joint Optimization](https://arxiv.org/abs/2505.15155)
- 👾**代码**: https://github.com/microsoft/RD-Agent/
```BibTeX
@misc{li2025rdagentquant,
    title={R\&D-Agent-Quant: A Multi-Agent Framework for Data-Centric Factors and Model Joint Optimization},
    author={Yuante Li and Xu Yang and Xiao Yang and Minrui Xu and Xisen Wang and Weiqing Liu and Jiang Bian},
    year={2025},
    eprint={2505.15155},
    archivePrefix={arXiv},
    primaryClass={cs.AI}
}
```
![image](https://github.com/user-attachments/assets/3198bc10-47ba-4ee0-8a8e-46d5ce44f45d)

***

| 特性 | 状态 |
| --                      | ------    |
| [R&D-Agent-Quant](https://arxiv.org/abs/2505.15155) 已发布 | 将 R&D-Agent 应用于 Qlib 进行量化交易 |
| BPQP 用于端到端学习 | 📈即将到来!([正在审核中](https://github.com/microsoft/qlib/pull/1863)) |
| 🔥LLM驱动的自动量化工厂🔥 | 🚀 于2024年8月8日在 [♾️RD-Agent](https://github.com/microsoft/RD-Agent) 发布 |
| KRNN 和 Sandwich 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/1414/) 于 2023年5月26日 |
| 发布 Qlib v0.9.0 | :octocat: [已发布](https://github.com/microsoft/qlib/releases/tag/v0.9.0) 于 2022年12月9日 |
| 强化学习框架 | :hammer: :chart_with_upwards_trend: 发布于 2022年11月10日。 [#1332](https://github.com/microsoft/qlib/pull/1332), [#1322](https://github.com/microsoft/qlib/pull/1322), [#1316](https://github.com/microsoft/qlib/pull/1316),[#1299](https://github.com/microsoft/qlib/pull/1299),[#1263](https://github.com/microsoft/qlib/pull/1263), [#1244](https://github.com/microsoft/qlib/pull/1244), [#1169](https://github.com/microsoft/qlib/pull/1169), [#1125](https://github.com/microsoft/qlib/pull/1125), [#1076](https://github.com/microsoft/qlib/pull/1076)|
| HIST 和 IGMTF 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/1040) 于 2022年4月10日 |
| Qlib [笔记本教程](https://github.com/microsoft/qlib/tree/main/examples/tutorial) | 📖 [已发布](https://github.com/microsoft/qlib/pull/1037) 于 2022年4月7日 |
| Ibovespa 指数数据 | :rice: [已发布](https://github.com/microsoft/qlib/pull/990) 于 2022年4月6日 |
| 时点数据库 | :hammer: [已发布](https://github.com/microsoft/qlib/pull/343) 于 2022年3月10日 |
| Arctic 提供者后端 & 订单簿数据示例 | :hammer: [已发布](https://github.com/microsoft/qlib/pull/744) 于 2022年1月17日 |
| 基于元学习的框架 & DDG-DA  | :chart_with_upwards_trend:  :hammer: [已发布](https://github.com/microsoft/qlib/pull/743) 于 2022年1月10日 |
| 基于规划的投资组合优化 | :hammer: [已发布](https://github.com/microsoft/qlib/pull/754) 于 2021年12月28日 |
| 发布 Qlib v0.8.0 | :octocat: [已发布](https://github.com/microsoft/qlib/releases/tag/v0.8.0) 于 2021年12月8日 |
| ADD 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/704) 于 2021年11月22日 |
| ADARNN 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/689) 于 2021年11月14日 |
| TCN 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/668) 于 2021年11月4日 |
| 嵌套决策框架 | :hammer: [已发布](https://github.com/microsoft/qlib/pull/438) 于 2021年10月1日。 [示例](https://github.com/microsoft/qlib/blob/main/examples/nested_decision_execution/workflow.py) 和 [文档](https://qlib.readthedocs.io/en/latest/component/highfreq.html) |
| 时序路由适配器 (TRA) | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/531) 于 2021年7月30日 |
| Transformer & Localformer | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/508) 于 2021年7月22日 |
| 发布 Qlib v0.7.0 | :octocat: [已发布](https://github.com/microsoft/qlib/releases/tag/v0.7.0) 于 2021年7月12日 |
| TCTS 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/491) 于 2021年7月1日 |
| 在线服务和自动模型滚动 | :hammer:  [已发布](https://github.com/microsoft/qlib/pull/290) 于 2021年5月17日 |
| DoubleEnsemble 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/286) 于 2021年3月2日 |
| 高频数据处理示例 | :hammer: [已发布](https://github.com/microsoft/qlib/pull/257) 于 2021年2月5日  |
| 高频交易示例 | :chart_with_upwards_trend: [部分代码已发布](https://github.com/microsoft/qlib/pull/227) 于 2021年1月28日  |
| 高频数据(1分钟) | :rice: [已发布](https://github.com/microsoft/qlib/pull/221) 于 2021年1月27日 |
| Tabnet 模型 | :chart_with_upwards_trend: [已发布](https://github.com/microsoft/qlib/pull/205) 于 2021年1月22日 |

2021年之前发布的特性未在此列出。

<p align="center">
  <img src="docs/_static/img/logo/1.png" />
</p>

Qlib 是一个开源的、面向AI的量化投资平台，旨在利用AI技术实现潜力、赋能研究并创造价值，从探索想法到实现生产。Qlib支持多样化的机器学习建模范式，包括监督学习、市场动态建模和强化学习。

越来越多的在不同范式中的 SOTA 量化研究工作/论文正在 Qlib 中发布，以协作解决量化投资中的关键挑战。例如，1) 使用监督学习从丰富的异构金融数据中挖掘市场的复杂非线性模式，2) 使用自适应概念漂移技术建模金融市场的动态特性，3) 使用强化学习建模连续投资决策并协助投资者优化其交易策略。

它包含数据处理、模型训练、回测的完整ML流水线；涵盖了量化投资的整个链条：alpha寻优、风险建模、投资组合优化和订单执行。
欲了解更多信息，请参阅我们的论文 ["Qlib: An AI-oriented Quantitative Investment Platform"](https://arxiv.org/abs/2009.11189)。

<table>
  <tbody>
    <tr>
      <th>框架、教程、数据 & DevOps</th>
      <th>量化研究中的主要挑战与解决方案</th>
    </tr>
    <tr>
      <td>
        <li><a href="#plans"><strong>计划</strong></a></li>
        <li><a href="#framework-of-qlib">Qlib框架</a></li>
        <li><a href="#quick-start">快速开始</a></li>
          <ul dir="auto">
            <li type="circle"><a href="#installation">安装</a> </li>
            <li type="circle"><a href="#data-preparation">数据准备</a></li>
            <li type="circle"><a href="#auto-quant-research-workflow">自动量化研究工作流</a></li>
            <li type="circle"><a href="#building-customized-quant-research-workflow-by-code">通过代码构建自定义量化研究工作流</a></li></ul>
        <li><a href="#quant-dataset-zoo"><strong>量化数据集动物园</strong></a></li>
        <li><a href="#learning-framework">学习框架</a></li>
        <li><a href="#more-about-qlib">更多关于Qlib</a></li>
        <li><a href="#offline-mode-and-online-mode">离线模式和在线模式</a>
        <ul>
          <li type="circle"><a href="#performance-of-qlib-data-server">Qlib数据服务器性能</a></li></ul>
        <li><a href="#related-reports">相关报告</a></li>
        <li><a href="#contact-us">联系我们</a></li>
        <li><a href="#contributing">贡献</a></li>
      </td>
      <td valign="baseline">
        <li><a href="#main-challenges--solutions-in-quant-research">量化研究中的主要挑战与解决方案</a>
          <ul>
            <li type="circle"><a href="#forecasting-finding-valuable-signalspatterns">预测：寻找有价值信号/模式</a>
              <ul>
                <li type="disc"><a href="#quant-model-paper-zoo"><strong>量化模型（论文）动物园</strong></a>
                  <ul>
                    <li type="circle"><a href="#run-a-single-model">运行单个模型</a></li>
                    <li type="circle"><a href="#run-multiple-models">运行多个模型</a></li>
                  </ul>
                </li>
              </ul>
            </li>
          <li type="circle"><a href="#adapting-to-market-dynamics">适应市场动态</a></li>
          <li type="circle"><a href="#reinforcement-learning-modeling-continuous-decisions">强化学习：建模连续决策</a></li>
          </ul>
        </li>
      </td>
    </tr>
  </tbody>
</table>

# 计划
正在开发的新特性（按预估发布时间排序）。
关于这些特性的反馈对我们非常重要。
<!-- | 特性                        | 状态      | -->
<!-- | --                      | ------    | -->

# Qlib框架

<div style="align: center">
<img src="docs/_static/img/framework-abstract.jpg" />
</div>

上面是 Qlib 的高级框架（用户可以在深入了解时找到 Qlib 设计的[详细框架](https://qlib.readthedocs.io/en/latest/introduction/introduction.html#framework)）。
组件被设计为松耦合模块，每个组件都可以独立使用。

Qlib 提供了强大的基础设施支持量化研究。[数据](https://qlib.readthedocs.io/en/latest/component/data.html)始终是重要部分。
设计了一个强大的学习框架，以支持不同层次的多样化学习范式（如[强化学习](https://qlib.readthedocs.io/en/latest/component/rl.html)、[监督学习](https://qlib.readthedocs.io/en/latest/component/workflow.html#model-section)）和模式（如[市场动态建模](https://qlib.readthedocs.io/en/latest/component/meta.html)）。
通过建模市场，[交易策略](https://qlib.readthedocs.io/en/latest/component/strategy.html)将生成交易决策并执行。不同层级或粒度的多种交易策略和执行器可以[嵌套在一起进行优化和运行](https://qlib.readthedocs.io/en/latest/component/highfreq.html)。
最后，将提供全面的[分析](https://qlib.readthedocs.io/en/latest/component/report.html)，并且模型可以[在线服务](https://qlib.readthedocs.io/en/latest/component/online.html)且成本较低。

# 快速开始

本快速入门指南试图演示：
1. 使用 _Qlib_ 构建完整的量化研究工作流并尝试您的想法是非常容易的。
2. 即使使用*公开数据*和*简单模型*，机器学习技术在实际量化投资中也**表现得非常出色**。

这里有一个快速的**[演示](https://terminalizer.com/view/3f24561a4470)**展示如何安装 ``Qlib``，并使用 ``qrun`` 运行 LightGBM。**但是**，请确保您已按照[说明](#data-preparation)准备了数据。

## 安装

此表格展示了 `Qlib` 支持的 Python 版本：
|               | 使用 pip 安装      | 从源码安装  |        绘图        |
| ------------- |:---------------------:|:--------------------:|:------------------:|
| Python 3.8    | :heavy_check_mark:    | :heavy_check_mark:   | :heavy_check_mark: |
| Python 3.9    | :heavy_check_mark:    | :heavy_check_mark:   | :heavy_check_mark: |
| Python 3.10   | :heavy_check_mark:    | :heavy_check_mark:   | :heavy_check_mark: |
| Python 3.11   | :heavy_check_mark:    | :heavy_check_mark:   | :heavy_check_mark: |
| Python 3.12   | :heavy_check_mark:    | :heavy_check_mark:   | :heavy_check_mark: |

**注意**:
1. **Conda** 被建议用于管理 Python 环境。在某些情况下，在 `conda` 环境外使用 Python 可能导致缺少头文件，导致某些包安装失败。
2. 请注意，在 Python 3.6 中安装 cython 在从源码安装 ``Qlib`` 时会引发一些错误。如果用户在机器上使用 Python 3.6，建议*升级* Python 到 3.8 或更高版本，或使用 `conda` 的 Python 从源码安装 ``Qlib``。

### 使用 pip 安装
用户可以根据以下命令使用 pip 轻松安装 ``Qlib``。

```bash
  pip install pyqlib
```

**注意**: pip 将安装最新稳定版的 qlib。然而，qlib 的 main 分支正在积极开发中。如果您想测试 main 分支中的最新脚本或功能。请使用以下方法安装 qlib。

### 从源码安装
此外，用户可以根据以下步骤通过源码安装最新开发版 ``Qlib``：

* 在从源码安装 ``Qlib`` 之前，用户需要安装一些依赖：

  ```bash
  pip install numpy
  pip install --upgrade cython
  ```

* 克隆仓库并安装 ``Qlib`` 如下。
    ```bash
    git clone https://github.com/microsoft/qlib.git && cd qlib
    pip install .  # `pip install -e .[dev]` 被推荐用于开发。详情请查看 docs/developer/code_standard_and_dev_guide.rst
    ```

**提示**: 如果您在环境中安装 `Qlib` 失败或无法运行示例，请比较您的步骤和 [CI 工作流](.github/workflows/test_qlib_from_source.yml)，这可能有助于您找到问题。

**Mac 提示**: 如果您在使用 M1 的 Mac 上，您可能会在构建 LightGBM 的轮子时遇到问题，这是由于 OpenMP 的依赖项缺失。要解决这个问题，请先用 ``brew install libomp`` 安装 openmp，然后运行 ``pip install .`` 来成功构建。

## 数据准备
❗ 由于更严格的数据安全政策，官方数据集暂时停用。您可以尝试社区贡献的[这个数据源](https://github.com/chenditc/investment_data/releases)。
这里是一个下载最新数据的示例。
```bash
wget https://github.com/chenditc/investment_data/releases/latest/download/qlib_bin.tar.gz
mkdir -p ~/.qlib/qlib_data/cn_data
tar -zxvf qlib_bin.tar.gz -C ~/.qlib/qlib_data/cn_data --strip-components=1
rm -f qlib_bin.tar.gz
```

下面的官方数据集将在不久的将来恢复。

----

通过运行以下代码来加载和准备数据：

### 通过模块获取
  ```bash
  # 获取1天数据
  python -m qlib.cli.data qlib_data --target_dir ~/.qlib/qlib_data/cn_data --region cn

  # 获取1分钟数据
  python -m qlib.cli.data qlib_data --target_dir ~/.qlib/qlib_data/cn_data_1min --region cn --interval 1min

  ```

### 从源码获取

  ```bash
  # 获取1天数据
  python scripts/get_data.py qlib_data --target_dir ~/.qlib/qlib_data/cn_data --region cn

  # 获取1分钟数据
  python scripts/get_data.py qlib_data --target_dir ~/.qlib/qlib_data/cn_data_1min --region cn --interval 1min

  ```

此数据集是通过收集的公开数据创建的，[爬虫脚本](scripts/data_collector/)已在同个仓库中发布。
用户可以使用它创建相同的数据集。[数据集说明](https://github.com/microsoft/qlib/tree/main/scripts/data_collector#description-of-dataset)

*请注意**注意**数据是从[Yahoo Finance](https://finance.yahoo.com/lookup)收集的，数据可能并不完美。
我们建议用户如果他们有高质量数据集，准备自己的数据。欲了解更多信息，用户可以参考[相关文档](https://qlib.readthedocs.io/en/latest/component/data.html#converting-csv-format-into-qlib-format)*。

### 日频数据的自动更新（来自 yahoo finance）
  > 如果用户只想在历史数据上尝试他们的模型和策略，此步骤是*可选的*。
  >
  > 建议用户手动更新一次数据（--trading_date 2021-05-25），然后将其设置为自动更新。
  >
  > **注意**: 用户无法基于 Qlib 提供的离线数据增量更新数据（某些字段被移除以减少数据大小）。用户应使用[yahoo collector](https://github.com/microsoft/qlib/tree/main/scripts/data_collector/yahoo#automatic-update-of-daily-frequency-datafrom-yahoo-finance)从头开始下载 Yahoo 数据，然后增量更新它。
  >
  > 有关更多信息，请参考: [yahoo collector](https://github.com/microsoft/qlib/tree/main/scripts/data_collector/yahoo#automatic-update-of-daily-frequency-datafrom-yahoo-finance)

  * 每个交易日将数据自动更新到 "qlib" 目录（Linux）
      * 使用 *crontab*: `crontab -e`
      * 设置定时任务：

        ```
        * * * * 1-5 python <script path> update_data_to_bin --qlib_data_1d_dir <user data dir>
        ```
        * **script path**: *scripts/data_collector/yahoo/collector.py*

  * 手动更新数据
      ```
      python scripts/data_collector/yahoo/collector.py update_data_to_bin --qlib_data_1d_dir <user data dir> --trading_date <start date> --end_date <end date>
      ```
      * *trading_date*: 交易日开始
      * *end_date*: 交易日结束（不包含）

### 检查数据健康状况
  * 我们提供了一个脚本来检查数据健康状况，您可以运行以下命令来检查数据是否健康。
    ```
    python scripts/check_data_health.py check_data --qlib_dir ~/.qlib/qlib_data/cn_data
    ```
  * 当然，您也可以添加一些参数来调整测试结果，例如这样。
    ```
    python scripts/check_data_health.py check_data --qlib_dir ~/.qlib/qlib_data/cn_data --missing_data_num 30055 --large_step_threshold_volume 94485 --large_step_threshold_price 20
    ```
  * 如果您想了解更多关于 `check_data_health` 的信息，请参考[文档](https://qlib.readthedocs.io/en/latest/component/data.html#checking-the-health-of-the-data)。

<!--
- 运行初始化代码并获取股票数据：

  ```python
  import qlib
  from qlib.data import D
  from qlib.constant import REG_CN

  # 初始化
  mount_path = "~/.qlib/qlib_data/cn_data"  # target_dir
  qlib.init(mount_path=mount_path, region=REG_CN)

  # 通过 Qlib 获取股票数据
  # 加载给定时间范围和频率的交易日历
  print(D.calendar(start_time='2010-01-01', end_time='2017-12-31', freq='day')[:2])

  # 将给定的市场名称解析为股票池配置
  instruments = D.instruments('csi500')
  print(D.list_instruments(instruments=instruments, start_time='2010-01-01', end_time='2017-12-31', as_list=True)[:6])

  # 在给定时间范围内加载某些股票的特征
  instruments = ['SH600000']
  fields = ['$close', '$volume', 'Ref($close, 1)', 'Mean($close, 3)', '$high-$low']
  print(D.features(instruments, fields, start_time='2010-01-01', end_time='2017-12-31', freq='day').head())
  ```
 -->

## Docker 镜像
1. 从 docker hub 仓库拉取 docker 镜像
    ```bash
    docker pull pyqlib/qlib_image_stable:stable
    ```
2. 启动一个新的 Docker 容器
    ```bash
    docker run -it --name <container name> -v <Mounted local directory>:/app pyqlib/qlib_image_stable:stable
    ```
3. 此时您在 docker 环境中，可以运行 qlib 脚本。示例：
    ```bash
    >>> python scripts/get_data.py qlib_data --name qlib_data_simple --target_dir ~/.qlib/qlib_data/cn_data --interval 1d --region cn
    >>> python qlib/cli/run.py examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml
    ```
4. 退出容器
    ```bash
    >>> exit
    ```
5. 重启容器
    ```bash
    docker start -i -a <container name>
    ```
6. 停止容器
    ```bash
    docker stop <container name>
    ```
7. 删除容器
    ```bash
    docker rm <container name>
    ```
8. 如果您想了解更多信息，请参考[文档](https://qlib.readthedocs.io/en/latest/developer/how_to_build_image.html)。

## 自动量化研究工作流
Qlib 提供了一个名为 `qrun` 的工具，可自动运行整个工作流（包括构建数据集、训练模型、回测和评估）。您可以根据以下步骤启动自动量化研究工作流并获得图形化报告分析：

1. 量化研究工作流：使用 lightgbm 工作流配置运行 `qrun` ([workflow_config_lightgbm_Alpha158.yaml](examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml) 如下所示。
    ```bash
      cd examples  # 避免在包含 `qlib` 的目录下运行程序
      qrun benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml
    ```
    如果用户想在调试模式下使用 `qrun`，请使用以下命令：
    ```bash
    python -m pdb qlib/cli/run.py examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml
    ```
    `qrun` 的结果如下所示，有关结果的更多解释请参考[文档](https://qlib.readthedocs.io/en/latest/component/strategy.html#result)。

    ```bash

    '以下是无成本超额收益的分析结果。'
                           risk
    mean               0.000708
    std                0.005626
    annualized_return  0.178316
    information_ratio  1.996555
    max_drawdown      -0.081806
    '以下是有成本超额收益的分析结果。'
                           risk
    mean               0.000512
    std                0.005626
    annualized_return  0.128982
    information_ratio  1.444287
    max_drawdown      -0.091078
    ```
    这里是关于 `qrun` 和[工作流](https://qlib.readthedocs.io/en/latest/component/workflow.html)的详细文档。

2. 图形化报告分析：首先，运行 `python -m pip install .[analysis]` 安装所需的依赖项。然后用 `jupyter notebook` 运行 `examples/workflow_by_code.ipynb` 获取图形化报告。
    - 预测信号（模型预测）分析
      - 组累计收益
      ![Cumulative Return](https://github.com/microsoft/qlib/blob/main/docs/_static/img/analysis/analysis_model_cumulative_return.png)
      - 收益分布
      ![long_short](https://github.com/microsoft/qlib/blob/main/docs/_static/img/analysis/analysis_model_long_short.png)
      - 信息系数 (IC)
      ![Information Coefficient](https://github.com/microsoft/qlib/blob/main/docs/_static/img/analysis/analysis_model_IC.png)
      ![Monthly IC](https://github.com/microsoft/qlib/blob/main/docs/_static/img/analysis/analysis_model_monthly_IC.png)
      ![IC](https://github.com/microsoft/qlib/blob/main/docs/_static/img/analysis/analysis_... [truncated]