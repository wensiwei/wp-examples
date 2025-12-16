# WP-Examples 

WarpParse 日志解析引擎的示例项目，展示如何使用 WarpParse进行高性能日志解析和处理。

## 提纲

- **core**： 核心配置示例
- **benchmark**：性能测试
- **extensions**： 扩展示例
- **enterprise**：  企业版本示例


## 🛠️ 快速开始

### 要求

- Linux/macOS
- 下载WarpParse 

## 运行环境设置 
### mac 
* 放置到%{HOME}/bin， 并在 PATH 里可以访问到达
* 安全许可运行

### 运行示例

1. **快速入门示例**
   ```bash
   cd core/getting_started
   ./run.sh
   ```

2. **性能基准测试**
   ```bash
   # TCP 到 Blackhole 测试
   cd benchmark/tcp_blackhole
   ./run.sh nginx 300000 
   ./run.sh aws 300000 
   ```
