# Qlib Codebase Structure Analysis

> Generated: 2025-12-14
> Project Type: AI-Oriented Quantitative Investment Platform
> Developer: Microsoft

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Root Directory Structure](#2-root-directory-structure)
- [3. Core Modules Deep Dive](#3-core-modules-deep-dive)
- [4. Configuration Files](#4-configuration-files)
- [5. Documentation Directory](#5-documentation-directory)
- [6. Test Directory](#6-test-directory)
- [7. Examples & Benchmarks](#7-examples--benchmarks)
- [8. Core Functional Areas](#8-core-functional-areas)
- [9. Data Collection & Preparation](#9-data-collection--preparation)
- [10. Code Statistics](#10-code-statistics)

---

## 1. Project Overview

**Qlib** is an AI-oriented quantitative investment platform developed by Microsoft. It is a Python library (PyPI package name: `pyqlib`) that provides a complete framework for quantitative trading research and production environments.

### Key Features

- Complete ML pipeline: Data → Features → Models → Backtest → Analysis
- Full investment chain: Alpha discovery → Risk modeling → Portfolio optimization → Order execution
- Multiple paradigm support: Supervised learning, meta-learning, reinforcement learning
- Production-grade features: Online serving, experiment tracking, high-frequency trading support
- Extensible architecture: Loosely-coupled modules, community contribution system
- High-performance optimization: Cython optimization, efficient caching, parallel processing

---

## 2. Root Directory Structure

```
/Users/admin/work/qlib/
├── qlib/                    # Main package source code (core implementation)
├── examples/                # Example workflows and benchmarks
├── tests/                   # Test suite
├── docs/                    # Documentation (Sphinx-based)
├── scripts/                 # Utility scripts (data collection, etc.)
├── .github/                 # GitHub workflows and CI/CD
├── setup.py                 # Build configuration (Cython extensions)
├── pyproject.toml          # Modern Python project configuration
├── README.md               # English documentation
├── README_CN.md            # Chinese documentation
└── Other config files      # .pylintrc, .mypy.ini, Dockerfile, etc.
```

### Main Configuration Files

| File | Purpose |
|------|---------|
| `pyproject.toml` | Project metadata, dependency management, CLI entry point |
| `setup.py` | Cython extension compilation configuration |
| `.pylintrc` | Code linting rules |
| `.mypy.ini` | Type checking configuration |
| `.pre-commit-config.yaml` | Pre-commit hooks |
| `Dockerfile` | Containerized deployment |

---

## 3. Core Modules Deep Dive

### 3.1 Data Layer (`qlib/data/`)

**Purpose**: Data handling, storage, and preprocessing

#### Core Files

| File | Size | Function |
|------|------|----------|
| `data.py` | 49KB | Core data access interface |
| `cache.py` | 47KB | Performance caching mechanism |
| `ops.py` | 45KB | Data operations and transformations |
| `filter.py` | - | Data filtering utilities |
| `pit.py` | - | Point-in-Time data handling |

#### Subdirectory Structure

- **`dataset/`** - Dataset handlers, loaders, processors, weights
  - `handler.py` - Data handler base class
  - `loader.py` - Data loaders
  - `processor.py` - Data preprocessing
  - `weight.py` - Sample weights

- **`storage/`** - File-based and abstract storage implementations
  - `file_storage.py` - File system storage
  - `storage.py` - Storage abstraction layer

- **`_libs/`** - Cython-optimized operations
  - `rolling.pyx` - Rolling window computations
  - `expanding.pyx` - Expanding window computations

### 3.2 Model Layer (`qlib/model/`)

**Purpose**: Machine learning model interfaces and implementations

#### Core Files

| File | Size | Function |
|------|------|----------|
| `base.py` | - | Model base class interface |
| `trainer.py` | 22KB | Model training logic |

#### Subdirectory Structure

- **`ens/`** - Ensemble methods
  - `ensemble.py` - Ensemble models
  - `group.py` - Group ensemble

- **`interpret/`** - Model interpretation tools
  - Feature importance analysis
  - Model interpretability

- **`meta/`** - Meta-learning components
  - `dataset.py` - Meta-learning datasets
  - `model.py` - Meta-learning models
  - `task.py` - Meta-learning tasks

- **`riskmodel/`** - Risk modeling
  - POET model
  - Shrinkage estimation
  - Structured covariance matrix

### 3.3 Contrib Layer (`qlib/contrib/`)

**Purpose**: Community-contributed model, strategy, and tool implementations

#### 36+ Model Implementations (`contrib/model/`)

##### Deep Learning Models
- **LSTM** - Long Short-Term Memory network
- **GRU** - Gated Recurrent Unit
- **Transformer** - Attention mechanism model
- **TabNet** - Tabular data neural network
- **TCN** - Temporal Convolutional Network
- **HIST** - Hierarchical Shared Transformer
- **TRA** - Temporal Routing Adaptor
- **ADD** - Adaptive Deep Learning
- **ADARNN** - Adaptive Recurrent Neural Network
- **Localformer** - Localized Transformer
- **TFT** - Temporal Fusion Transformer
- **TCTS** - Time-Conditional Transformer Sequence

##### Tree-based Models
- **LightGBM** - Light Gradient Boosting Machine
- **XGBoost** - Extreme Gradient Boosting
- **CatBoost** - Categorical Boosting

##### Specialized Models
- **DoubleEnsemble** - Double ensemble
- **IGMTF** - Information Gain Multi-Task Fusion
- **KRNN** - Knowledge-enhanced RNN
- **Sandwich** - Sandwich network
- **SFM** - Stochastic Factor Model
- **GATs** - Graph Attention Networks

#### Other Subdirectories

- **`strategy/`** - Trading strategies
  - `signal_strategy.py` - Signal-based strategies
  - `rule_strategy.py` - Rule-based strategies
  - `order_generator.py` - Order generators

- **`data/`** - High-frequency data handlers
  - `handler.py` - High-frequency data processing

- **`report/`** - Analysis and reporting tools
  - Performance analysis
  - Visualization reports

- **`online/`** - Online serving components
  - Online model deployment
  - Real-time prediction

- **`rolling/`** - Rolling window implementations
  - DDG-DA - Dynamic Distribution-Guided Domain Adaptation

- **`tuner/`** - Hyperparameter tuning
  - Automatic tuning framework

### 3.4 Backtest Layer (`qlib/backtest/`)

**Purpose**: Backtesting framework for strategy evaluation

#### Core Files

| File | Size | Function |
|------|------|----------|
| `exchange.py` | 44KB | Exchange simulation |
| `executor.py` | 26KB | Order execution |
| `position.py` | 20KB | Position management |
| `account.py` | 17KB | Account tracking |
| `decision.py` | 21KB | Decision making |
| `report.py` | 27KB | Performance reporting |
| `profit_attribution.py` | - | Attribution analysis |
| `high_performance_ds.py` | - | High-performance data structures |

#### Functional Features

- Realistic exchange simulation
- Order execution strategies
- Position management
- Transaction cost modeling
- High-frequency backtesting
- Nested decision execution framework

### 3.5 Workflow Layer (`qlib/workflow/`)

**Purpose**: Experiment management and workflow orchestration

#### Core Files

| File | Size | Function |
|------|------|----------|
| `__init__.py` | 25KB | Main workflow interface |
| `recorder.py` | 18KB | Experiment recording |
| `expm.py` | 17KB | Experiment manager (MLflow integration) |
| `exp.py` | 15KB | Experiment definitions |
| `record_temp.py` | 27KB | Record templates |

#### Subdirectory Structure

- **`task/`** - Task management
  - `collect.py` - Task collection
  - `generate.py` - Task generation
  - `manage.py` - Task management

- **`online/`** - Online model management
  - Model updates
  - Online serving

#### Integration Features

- MLflow experiment tracking
- Recorder pattern (reproducibility)
- Task scheduling
- Online model serving and rolling updates

### 3.6 Strategy Layer (`qlib/strategy/`)

**Purpose**: Trading strategy implementations

#### Core Files

| File | Size | Function |
|------|------|----------|
| `base.py` | 11KB | Strategy base class |

### 3.7 Reinforcement Learning Layer (`qlib/rl/`)

**Purpose**: RL-based trading and order execution

#### Core Files

| File | Function |
|------|----------|
| `interpreter.py` | RL interpreter |
| `reward.py` | Reward functions |
| `simulator.py` | Market simulation |

#### Subdirectory Structure

- **`order_execution/`** - RL-based order execution
  - `policy.py` - Policies
  - `network.py` - Neural networks
  - `state.py` - State representation
  - `strategy.py` - Execution strategies

- **`trainer/`** - RL training framework
  - `callbacks.py` - Callback functions
  - `vessel.py` - Vessel
  - `trainer.py` - Trainer

- **`contrib/`** - Contributed RL implementations
  - `backtest.py` - RL backtesting
  - `train_onpolicy.py` - On-policy training

- **`data/`** - RL data handling
- **`strategy/`** - RL strategies
- **`utils/`** - RL utilities

#### Integration Features

- Tianshou library integration
- Continuous decision-making framework
- Order execution optimization
- Custom environments and simulators

### 3.8 Utilities Layer (`qlib/utils/`)

**Purpose**: Common utilities and helper functions

#### Core Files

| File | Size | Function |
|------|------|----------|
| `__init__.py` | 30KB | Core utility functions |
| `index_data.py` | 22KB | Index data handling |
| `time.py` | 11KB | Time utilities |
| `paral.py` | 10KB | Parallel processing |
| `resam.py` | - | Resampling utilities |
| `serial.py` | - | Serialization |
| `mod.py` | - | Module loading |
| `objm.py` | - | Object management |

### 3.9 CLI Layer (`qlib/cli/`)

**Purpose**: Command-line interface

#### Core Files

| File | Function |
|------|----------|
| `run.py` | Main CLI runner (qrun command) |
| `data.py` | Data management CLI |

### 3.10 Core Configuration (qlib root)

| File | Size | Function |
|------|------|----------|
| `__init__.py` | - | Package initialization (init() and auto_init()) |
| `config.py` | 18KB | Configuration management |
| `constant.py` | - | Constant definitions (regions, etc.) |
| `log.py` | - | Logging configuration |
| `typehint.py` | - | Type hints |

---

## 4. Configuration Files

### 4.1 Primary Configuration (`pyproject.toml`)

```toml
[project]
name = "pyqlib"
# Package metadata
# Dependencies: pandas, numpy, mlflow, lightgbm, etc.

[project.optional-dependencies]
rl = [...]         # Reinforcement learning dependencies
dev = [...]        # Development dependencies
lint = [...]       # Code linting dependencies
docs = [...]       # Documentation building dependencies
test = [...]       # Testing dependencies
analysis = [...]   # Analysis tool dependencies

[project.scripts]
qrun = "qlib.cli.run:run"  # CLI entry point
```

### 4.2 Build Configuration (`setup.py`)

Cython extension compilation:
- `qlib.data._libs.rolling` - Rolling window operations
- `qlib.data._libs.expanding` - Expanding window operations

### 4.3 Other Configuration Files

| File | Purpose |
|------|---------|
| `.pylintrc` | Code linting rules |
| `.mypy.ini` | Type checking configuration |
| `.pre-commit-config.yaml` | Pre-commit hooks |
| `.deepsource.toml` | Code quality checks |
| `.readthedocs.yaml` | Documentation building configuration |

---

## 5. Documentation Directory

### `docs/` Structure

```
docs/
├── _static/           # Static assets (images, logos)
├── start/            # Getting started guides
├── component/        # Component documentation
│   ├── data/        # Data components
│   ├── model/       # Model components
│   ├── backtest/    # Backtest components
│   ├── workflow/    # Workflow components
│   └── rl/          # Reinforcement learning components
├── introduction/     # Introduction and framework overview
├── advanced/         # Advanced topics
├── developer/        # Developer guides
├── reference/        # API reference
├── FAQ/             # Frequently asked questions
├── changelog/        # Version history
└── conf.py          # Sphinx configuration
```

### Documentation Features

- Sphinx-based
- Bilingual support (English and Chinese)
- Auto-generated API
- Rich code examples
- Hosted on ReadTheDocs

---

## 6. Test Directory

### `tests/` Structure

```
tests/
├── 10+ test files (root level)
├── backtest/              # Backtest tests
├── data_mid_layer_tests/  # Data layer tests
├── dataset_tests/         # Dataset tests
├── model/                # Model tests
├── rl/                   # Reinforcement learning tests
├── ops/                  # Operations tests
├── storage_tests/        # Storage tests
├── rolling_tests/        # Rolling window tests
├── dependency_tests/     # Dependency tests
├── misc/                 # Miscellaneous tests
├── conftest.py           # Pytest configuration
└── pytest.ini            # Pytest settings
```

### Test Infrastructure

- Pytest framework
- Test fixtures
- Parameterized tests
- Continuous integration

---

## 7. Examples & Benchmarks

### 7.1 Benchmarks (`examples/benchmarks/`)

#### 27+ Model Benchmark Directories

##### Tree-based Models
- **LightGBM** - Gradient Boosting Decision Tree
- **XGBoost** - Extreme Gradient Boosting
- **CatBoost** - Categorical Boosting

##### Neural Networks
- **LSTM** - Long Short-Term Memory network
- **GRU** - Gated Recurrent Unit
- **TCN** - Temporal Convolutional Network
- **Transformer** - Attention mechanism
- **MLP** - Multi-Layer Perceptron

##### Advanced Models
- **ALSTM** - Attention LSTM
- **GATs** - Graph Attention Networks
- **TRA** - Temporal Routing Adaptor
- **HIST** - Hierarchical Shared Transformer
- **IGMTF** - Information Gain Multi-Task Fusion
- **KRNN** - Knowledge-enhanced RNN
- **Sandwich** - Sandwich network
- **ADD** - Adaptive Deep Learning
- **ADARNN** - Adaptive RNN

##### Specialized Models
- **TabNet** - Tabular network
- **TFT** - Temporal Fusion Transformer
- **Localformer** - Local Transformer
- **TCTS** - Time-Conditional Transformer Sequence
- **SFM** - Stochastic Factor Model
- **DoubleEnsemble** - Double ensemble

Each benchmark contains:
- Configuration files (YAML)
- README documentation
- Training scripts

### 7.2 Other Examples

```
examples/
├── tutorial/                    # Tutorial notebooks
├── benchmarks_dynamic/          # Dynamic adaptation benchmarks
│   ├── baseline/               # Baseline methods
│   └── DDG-DA/                # Dynamic Distribution-Guided Domain Adaptation
├── highfreq/                   # High-frequency trading examples
├── rl_order_execution/         # RL order execution examples
├── nested_decision_execution/  # Nested decision framework
├── online_srv/                 # Online serving examples
├── portfolio/                  # Portfolio optimization examples
├── model_rolling/              # Rolling model examples
├── model_interpreter/          # Model interpretation examples
├── hyperparameter/             # Hyperparameter tuning examples
├── orderbook_data/             # Order book data examples
├── data_demo/                  # Data handling demos
├── workflow_by_code.py         # Code-based workflow example
└── run_all_model.py            # Batch model execution
```

---

## 8. Core Functional Areas

### 8.1 Data Management

#### Features
- Multi-frequency data support (1d, 1min, 5min)
- Point-in-Time (PIT) database (avoiding look-ahead bias)
- High-frequency data processing
- Multiple data source support
- Efficient storage (Cython-optimized)
- Caching mechanisms

#### Supported Data Sources
- **Yahoo Finance** - Global stock data
- **Baostock** - Chinese market data
- **Cryptocurrency** - Digital asset data
- **Fund data** - Fund NAV data
- **Custom data sources** - Extensible interface

### 8.2 Feature Engineering

#### Predefined Feature Sets
- **Alpha158** - 158 technical indicators
- **Alpha360** - 360 features

#### Custom Operators
- Rolling window operations
- Expanding window operations
- Cross-sectional operations
- High-frequency operators

### 8.3 Model Development

#### Model Paradigms
- **Supervised learning** - GBDT, neural networks
- **Time series** - LSTM, GRU, Transformer
- **Graph neural networks** - GATs
- **Attention mechanisms** - TRA, HIST
- **Meta-learning** - Market adaptation

#### Model Features
- 36+ pre-implemented models
- Unified model interface
- Easy to extend
- Community contributions

### 8.4 Backtesting & Execution

#### Backtesting Features
- Realistic exchange simulation
- Order execution strategies
- Position management
- Transaction cost modeling
- High-frequency backtesting
- Nested decision framework

#### Order Execution
- Market orders
- Limit orders
- TWAP/VWAP
- RL-optimized execution

### 8.5 Portfolio Management

#### Functions
- **Portfolio optimization** - cvxpy-based
- **Risk modeling** - Factor models, covariance estimation
- **Enhanced indexing** - Index tracking and enhancement
- **Position analysis** - Attribution analysis

### 8.6 Reinforcement Learning

#### Framework Features
- RL continuous decision-making framework
- Order execution optimization
- Tianshou integration
- Custom environments and simulators

#### Application Scenarios
- Order execution
- Dynamic position adjustment
- Market making

### 8.7 Experiment Management

#### Functions
- **MLflow integration** - Experiment tracking
- **Recorder pattern** - Reproducibility
- **Task management** - Task scheduling
- **Online serving** - Model deployment and rolling updates

### 8.8 Analysis & Reporting

#### Analysis Tools
- Comprehensive performance metrics
- Profit attribution analysis
- Model interpretation
- Visualization tools (Plotly, Matplotlib)

#### Report Types
- Backtest reports
- Performance analysis
- Risk analysis
- Attribution analysis

---

## 9. Data Collection & Preparation

### `scripts/` Structure

```
scripts/
├── data_collector/          # Data collection tools
│   ├── cn_index/           # Chinese stock market
│   ├── us_index/           # US stock market
│   ├── br_index/           # Brazilian stock market (Ibovespa)
│   ├── yahoo/              # Yahoo Finance
│   ├── crypto/             # Cryptocurrency
│   ├── fund/               # Fund data
│   ├── pit/                # Point-in-time data
│   ├── baostock_5min/      # 5-minute data
│   └── crowd_source/       # Community contributions
├── dump_bin.py             # Binary data dumping (21KB)
├── dump_pit.py             # PIT data dumping (11KB)
└── get_data.py             # Data retrieval script
```

### Data Workflow

1. **Data collection** - Download data from various sources
2. **Data cleaning** - Handle missing values, outliers
3. **Data dumping** - Convert to efficient binary format
4. **PIT processing** - Build point-in-time database
5. **Feature calculation** - Generate technical indicators

---

## 10. Code Statistics

### Code Volume

- **Total Python LOC**: ~56,000 lines (qlib package only)
- **Language Distribution**:
  - Python (primary)
  - Cython (performance-critical parts)
  - C++ (compiled extensions)

### Supported Environments

- **Python versions**: 3.8, 3.9, 3.10, 3.11, 3.12
- **Platforms**: Linux, Windows, macOS

### Dependencies

#### Core Dependencies
- **Data processing**: pandas, numpy
- **Machine learning**: scikit-learn, scipy
- **Deep learning**: PyTorch
- **Tree models**: LightGBM, XGBoost, CatBoost
- **Experiment tracking**: MLflow
- **Reinforcement learning**: Tianshou

#### Optional Dependencies
- **RL extensions**: gym, tensorboard
- **Development tools**: pytest, black, flake8
- **Documentation**: Sphinx, sphinx-rtd-theme
- **Analysis**: matplotlib, plotly, seaborn

---

## Architecture Summary

### Design Principles

1. **Loosely-coupled architecture** - Low coupling between modules, easy to extend
2. **Layered design** - Data → Model → Strategy → Backtest → Analysis
3. **Pluggable components** - Components at each layer can be independently replaced
4. **Performance optimization** - Cython optimization on critical paths
5. **Production-ready** - Online serving, monitoring, logging

### Extension Mechanisms

1. **Custom data sources** - Implement Data Handler interface
2. **Custom models** - Inherit from Model base class
3. **Custom strategies** - Implement Strategy interface
4. **Custom executors** - Inherit from Executor base class
5. **Community contributions** - contrib directory mechanism

### Best Practices

1. **Use workflows** - Manage experiments through workflow
2. **Record tracking** - Use Recorder to record all experiments
3. **Modular development** - Follow single responsibility principle
4. **Test-driven** - Write unit tests and integration tests
5. **Documentation first** - Write documentation for new features

---

## Appendix

### A. Key Interfaces

#### Data Handler
```python
class DataHandler:
    def fetch()
    def load()
    def process()
```

#### Model
```python
class Model:
    def fit()
    def predict()
    def save()
    def load()
```

#### Strategy
```python
class Strategy:
    def generate_trade_decision()
```

#### Executor
```python
class Executor:
    def execute()
```

### B. Configuration Examples

#### Data Configuration
```yaml
provider_uri: ~/.qlib/qlib_data/cn_data
region: cn
```

#### Model Configuration
```yaml
model:
  class: LGBMModel
  module_path: qlib.contrib.model.gbdt
  kwargs:
    loss: mse
    num_leaves: 31
```

#### Strategy Configuration
```yaml
strategy:
  class: TopkDropoutStrategy
  module_path: qlib.contrib.strategy.signal_strategy
  kwargs:
    topk: 50
    n_drop: 5
```

### C. CLI Commands

```bash
# Run workflow
qrun workflow.yaml

# Download data
python scripts/get_data.py qlib_data --target_dir ~/.qlib/qlib_data/cn_data --region cn

# Dump data
python scripts/dump_bin.py dump_all --csv_path data --qlib_dir ~/.qlib/qlib_data/cn_data
```

### D. Reference Resources

- **Official Documentation**: https://qlib.readthedocs.io/
- **GitHub Repository**: https://github.com/microsoft/qlib
- **Paper**: [Qlib: An AI-oriented Quantitative Investment Platform](https://arxiv.org/abs/2009.11189)
- **Community**: GitHub Discussions

---

**End of Document**
