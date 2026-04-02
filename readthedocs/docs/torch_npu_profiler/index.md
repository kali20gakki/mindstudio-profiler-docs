# torch_npu.profiler

`torch_npu.profiler` 是面向 PyTorch 训练与在线推理场景的性能数据采集工具，支持采集框架侧算子信息、CANN 软件栈数据、底层 NPU 算子信息以及显存占用信息，并可衔接自动解析、离线解析、MindStudio Insight 和 `msprof-analyze` 进行后续分析。

## 文档导读

- 初次使用，建议先阅读 [快速开始](./source/getting_started/quick_start.md)。
- 需要完成常规性能采集，优先阅读 [使用 `torch_npu.profiler.profile` 采集](./source/user_guide/profile_api.md)。
- 需要在训练或推理过程中按需触发采集，阅读 [dynamic_profile 动态采集](./source/user_guide/dynamic_profile.md)。
- 采集完成后需要手动解析数据，阅读 [离线解析](./source/user_guide/offline_analyse.md)。
- 需要查参数、配置文件和扩展能力，阅读 [参考说明](./source/reference/index.md)。

## 采集方式选型

| 方式 | 适用场景 | 特点 |
| --- | --- | --- |
| `torch_npu.profiler.profile` | 常规训练、在线推理、单次定点采集 | 功能最完整，支持 `schedule`、自动解析、显存分析、子线程采集等，推荐优先使用。 |
| `dynamic_profile` | 训练/推理任务运行过程中按需启动采集 | 支持运行中修改配置触发采集，适合长任务和不方便频繁改代码的场景。 |
| `torch_npu.profiler._KinetoProfile` | 基础采集或兼容性场景 | 能力较精简，不支持 `schedule` 和 `tensorboard_trace_handler`。 |

## 典型流程

1. 完成环境准备并选择采集方式。
2. 在脚本中接入 `torch_npu.profiler.profile`、`dynamic_profile` 或 `_KinetoProfile`。
3. 执行训练或在线推理任务，生成性能数据。
4. 选择自动解析或离线解析。
5. 使用 MindStudio Insight 或 `msprof-analyze` 查看和分析结果。

## 目录结构

- [入门](./source/getting_started/index.md)：前提条件、快速开始、完整使用流程。
- [使用指南](./source/user_guide/index.md)：三种采集方式、离线解析、结果查看。
- [高级特性](./source/advanced_features/index.md)：mstx、自定义元数据、显存时间线、子线程采集等。
- [参考说明](./source/reference/index.md)：接口参数、`schedule`、`profiler_config.json`、`experimental_config`。
