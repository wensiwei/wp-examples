# Benchmark 用例指南

benchmark 目录收录了基于 `benchmark/benchmark_common.sh` 的性能测试用例。测试用例按数据源类型组织为多个 case 目录，每个 case 下包含不同的处理场景。本文档说明整体结构、通用参数与各测试场景的用途。

## 前置准备

1. 所有脚本默认在 release profile 下运行，并依赖 `wparse`/`wpgen`/`wproj`，确保它们位于 PATH 中。
2. 从 benchmark 目录或具体测试目录运行脚本。

## 目录结构

```
benchmark/
├── benchmark_common.sh      # 公共函数库（参数解析、环境初始化等）
├── check_run.sh            # 批量测试脚本（使用 -m 参数运行所有测试）
├── models/                 # 共享的模型文件
│   ├── wpl/               # WPL 规则集（nginx、sysmon、apt、aws 等）
│   └── oml/               # OML 转换模型
├── sinks/                 # 共享的 sink 配置
│   ├── parse_to_blackhole/
│   ├── parse_to_file/
│   ├── trans_to_blackhole/
│   └── trans_to_file/
├── case_tcp/              # TCP 数据源测试场景
│   ├── sources/           # TCP 源配置
│   ├── parse_to_blackhole/
│   ├── parse_to_file/
│   ├── trans_to_blackhole/
│   └── trans_to_file/
├── case_file/             # File 数据源测试场景
│   ├── sources/           # File 源配置
│   ├── parse_to_blackhole/
│   ├── parse_to_file/
│   ├── trans_to_blackhole/
│   └── trans_to_file/
├── case_syslog/           # Syslog 数据源测试场景
│   ├── sources/           # Syslog 源配置
│   ├── parse_to_blackhole/
│   └── trans_to_blackhole/
└── wpgen_test/            # wpgen 性能测试
```

### 测试场景说明

**处理模式：**
- `parse`: 解析模式 - 使用 WPL 规则对日志进行解析和转换
- `trans`: 透传模式 - 不进行解析，直接转发原始数据

**输出目标：**
- `blackhole`: 黑洞输出 - 丢弃数据，用于测试纯解析/转发性能
- `file`: 文件输出 - 输出到文件，测试完整的处理链路

**示例：**
- `parse_to_blackhole`: 解析后丢弃，测试解析性能
- `parse_to_file`: 解析后写入文件，测试完整解析链路
- `trans_to_blackhole`: 透传后丢弃，测试转发性能
- `trans_to_file`: 透传后写入文件，测试完整转发链路

## 通用选项

所有测试脚本共享以下参数（由 `benchmark_common.sh` 中的 `benchmark_parse_args` 解析）：

- **`-m`**: 使用中等规模数据集（LINE_CNT=200,000 行）；默认使用大规模数据集（20,000,000 行）
- **`-f`**: 强制重新生成输入数据，即使 `./data/in_dat/*.dat` 已存在（部分脚本支持）
- **`-c <cnt>`**: 指定数据条数，与 `-m` 互斥，优先级更高
- **`-w <cnt>`**: 指定 wparse worker 数量
  - daemon 模式默认 6 worker
  - batch/blackhole 模式默认 10 worker
- **`wpl_dir`**: 指定 WPL 规则集目录（位置参数）
  - 可选值：`nginx`、`sysmon`、`apt`、`aws` 等
  - 默认值：`nginx`
  - 路径相对于 `benchmark/models/wpl/`
- **`speed`**: 样本生成限速（行/秒），0 表示不限速（位置参数，默认 0）

执行 `./run.sh -h` 可查看某个测试脚本支持的选项组合。

## 快速开始

### 1. 运行单个测试

```bash
cd benchmark

# 使用默认配置（nginx 规则，大规模数据集）
./case_tcp/parse_to_blackhole/run.sh

# 使用中等规模数据集
./case_tcp/parse_to_blackhole/run.sh -m

# 使用 sysmon 规则，12 个 worker，限速 1M 行/秒
./case_tcp/parse_to_blackhole/run.sh -w 12 sysmon 1000000

# 自定义数据量和 worker 数
./case_file/parse_to_file/run.sh -c 500000 -w 8
```

### 2. 批量测试所有场景

使用 `check_run.sh` 脚本自动运行所有测试用例（使用 `-m` 参数进行小数据测试）：

```bash
cd benchmark
./check_run.sh
```

**输出示例：**
```
========================================
Benchmark Check - Small Data Test
========================================

检查 case_tcp ...
  → 运行 case_tcp/parse_to_blackhole ...
    ✓ case_tcp/parse_to_blackhole 通过

  → 运行 case_tcp/parse_to_file ...
    ✓ case_tcp/parse_to_file 通过
  ...

========================================
测试总结
========================================
总测试数: 10
通过: 10
失败: 0

所有测试通过！
```

## 测试场景清单

### TCP 数据源测试（case_tcp）

| 测试场景 | 说明 | 配置文件 |
| --- | --- | --- |
| `parse_to_blackhole` | TCP 数据解析后丢弃，测试 TCP 接收 + 解析性能 | wpgen.toml, wparse.toml |
| `parse_to_file` | TCP 数据解析后写入文件 | wpgen.toml, wparse.toml |
| `trans_to_blackhole` | TCP 数据透传后丢弃，测试 TCP 接收 + 转发性能 | wpgen.toml, wparse.toml |
| `trans_to_file` | TCP 数据透传后写入文件 | wpgen.toml, wparse.toml |

**数据源：** wpgen 通过 TCP 连接（默认端口 19001）发送样本数据

### File 数据源测试（case_file）

| 测试场景 | 说明 | 配置文件 |
| --- | --- | --- |
| `parse_to_blackhole` | 文件数据解析后丢弃，测试文件读取 + 解析性能 | wpgen.toml, wparse.toml |
| `parse_to_file` | 文件数据解析后写入文件，测试完整 file-to-file 链路 | wpgen.toml, wparse.toml |
| `trans_to_blackhole` | 文件数据透传后丢弃 | wpgen.toml, wparse.toml |
| `trans_to_file` | 文件数据透传后写入文件 | wpgen.toml, wparse.toml |

**数据源：** wpgen 预先生成数据文件到 `./data/in_dat/`

### Syslog 数据源测试（case_syslog）

| 测试场景 | 说明 | 配置文件 |
| --- | --- | --- |
| `parse_to_blackhole` | Syslog UDP 数据解析后丢弃 | wpgen.toml, wparse.toml |
| `trans_to_blackhole` | Syslog UDP 数据透传后丢弃 | wpgen.toml, wparse.toml |

**数据源：** wpgen 通过 Syslog UDP 协议发送样本数据

### wpgen 性能测试（wpgen_test）

专门测试 wpgen 的样本数据生成能力，不启动 wparse。

## 配置文件说明

每个测试场景目录下通常包含：

```
<test_scenario>/
├── conf/
│   ├── wparse.toml    # wparse 引擎配置
│   └── wpgen.toml     # wpgen 数据生成配置
├── data/              # 运行时数据目录（自动创建）
│   ├── in_dat/        # 输入数据
│   ├── out_dat/       # 输出数据
│   ├── logs/          # 日志文件
│   └── rescue/        # 异常数据
├── .run/              # 运行时文件（PID 等）
└── run.sh             # 测试脚本
```

### wparse.toml 配置要点

```toml
[models]
wpl = "../../models/wpl"    # WPL 规则路径（相对于测试目录）
oml = "../../models/oml"    # OML 模型路径

[topology]
sources = "../sources"      # 数据源配置路径
sinks = "../../sinks/xxx"   # Sink 配置路径

[performance]
parse_workers = 6           # 解析 worker 数量
rate_limit_rps = 0          # 速率限制（0 表示不限制）
```

### wpgen.toml 配置要点

```toml
[generator]
mode = "sample"             # 生成模式
count = 1000                # 每批次数量
speed = 0                   # 生成速度限制
parallel = 4                # 并行度

[output]
connect = "tcp_sink"        # 连接器类型（tcp_sink/file_raw_sink 等）
params = { port = 19001 }   # 连接器参数
```

## 性能调优建议

### 确定最佳 Worker 数

1. 从 CPU 核心数开始测试
2. 使用不同 worker 数运行同一测试：
   ```bash
   ./case_tcp/parse_to_blackhole/run.sh -m -w 2
   ./case_tcp/parse_to_blackhole/run.sh -m -w 4
   ./case_tcp/parse_to_blackhole/run.sh -m -w 6
   ./case_tcp/parse_to_blackhole/run.sh -m -w 8
   ```
3. 对比吞吐量，找到性价比最优点

### 数据规模选择

- **开发/调试**：使用 `-m` 参数（20 万行），快速验证
- **性能测试**：使用默认规模（2000 万行）或自定义 `-c` 参数
- **压力测试**：使用 `-c` 指定更大数据量

### 避免 I/O 瓶颈

- `blackhole` 测试：测试纯计算性能，无 I/O 影响
- `file` 测试：注意磁盘 I/O 可能成为瓶颈
- 使用 SSD 或 RAM disk 可提升 I/O 性能

## 输出与校验

每个脚本会自动调用以下命令显示结果：

- `wproj data stat`: 打印数据统计信息（输入/输出条数、耗时等）
- `wproj validate sink-file`: 校验输出文件（仅限 file 输出场景）

若遇到数据残留导致统计不准，可手动执行：
```bash
wproj data clean   # 清理 wparse 数据
wpgen data clean   # 清理 wpgen 数据
```

或使用 `-f` 参数强制重新生成数据（部分脚本支持）。

## 常见问题

###  WPL 模型加载失败

检查 wparse.toml 中的 `wpl` 路径是否正确，应相对于测试目录：
```toml
[models]
wpl = "../../models/wpl"  # 正确
# wpl = "../../../models/wpl"  # 错误（旧配置）
```

###  数据生成失败

- 检查磁盘空间是否充足
- 确认 wpgen.toml 配置正确
- 查看 `./data/logs/` 下的日志文件

###  测试运行缓慢

- 使用 `-m` 参数减小数据规模
- 调整 `-w` 参数优化 worker 数
- 检查系统资源使用情况（CPU、内存、磁盘 I/O）
