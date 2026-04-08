# mindstudio-profiler-docs

MindStudio Profiling Tools 文档托管仓库。

这个仓库用于聚合 Ascend 性能工具相关文档，并通过 MkDocs 统一对外提供站点。当前主要聚合的子工具包括：

- `msprof`：性能数据采集、导出与基础解析
- `ascend-pytorch-profiler`：PyTorch / `torch_npu` 场景的框架侧 Profiling
- `mspti`：Profiling C API / Python API 接入
- `msmonitor`：在线监控、动态采集与运行时观测
- `msprof-analyze`：基于 Profiling 数据的分析、对比与调优定位

## 目录说明

```text
.
├── readthedocs/
│   ├── docs/                 # MkDocs 文档根目录
│   ├── mkdocs.yml            # MkDocs 配置
│   ├── requirements.txt      # 文档依赖
│   └── scripts/build_docs.py # 预构建脚本：同步子工具文档并生成导航
├── .readthedocs.yaml         # Read the Docs 构建配置
└── .gitmodules               # 子模块配置
```

## 本地准备

建议使用 Python 3.10+。

先拉取仓库和子模块：

```bash
git clone --recurse-submodules <repo-url>
cd mindstudio-profiler-docs
git submodule sync --recursive
git submodule update --init --recursive
git submodule update --remote --recursive
```

安装文档依赖：

```bash
python3 -m pip install -r readthedocs/requirements.txt
```

## 本地启动服务

本地预览前需要先执行一次预构建脚本。这个脚本会：

- 从各子工具仓库同步文档
- 过滤需要展示的目录与页面
- 生成工具导航与首页入口
- 重写部分站内链接

执行命令：

```bash
python3 readthedocs/scripts/build_docs.py
```

然后启动本地服务：

```bash
mkdocs serve -f readthedocs/mkdocs.yml -a 127.0.0.1:8000
```

启动后浏览器访问：

```text
http://127.0.0.1:8000
```

如果 `8000` 端口已被占用，可以改成别的端口，例如：

```bash
mkdocs serve -f readthedocs/mkdocs.yml -a 127.0.0.1:8001
```

## 本地构建

如果只想检查能否成功构建静态站点，可以执行：

```bash
python3 readthedocs/scripts/build_docs.py
mkdocs build -f readthedocs/mkdocs.yml --strict
```

构建产物默认输出到：

```text
readthedocs/site/
```

## 与 RTD 的一致性

Read the Docs 上的构建流程定义在 [.readthedocs.yaml](/home/fanglanyue/workspace/mindstudio-profiler-docs/.readthedocs.yaml)，主要步骤是：

1. 同步并更新子模块
2. 将子工具文档切到指定分支
3. 执行 `python readthedocs/scripts/build_docs.py`
4. 使用 `readthedocs/mkdocs.yml` 构建站点

本地调试时，建议尽量复用同样的顺序。

## 常见问题

### 1. `mkdocs serve` 提示 `Address already in use`

说明默认端口被占用了。可以：

- 结束占用 `8000` 端口的进程
- 或者直接换端口启动，例如 `8001`

### 2. 构建时出现锚点告警

例如：

- `contains a link '#xxx', but there is no such anchor on this page`
- `does not contain an anchor '#xxx'`

这类通常是文档内部链接与标题锚点不一致导致的，不一定会阻塞站点启动，但建议逐步修复。

### 3. 为什么修改了脚本但页面没变化

因为导航和聚合内容不是纯手写的，很多页面是在预构建阶段生成的。修改脚本后需要重新执行：

```bash
python3 readthedocs/scripts/build_docs.py
```

再运行 `mkdocs serve` 或 `mkdocs build`。
