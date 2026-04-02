# 概览

Ascend PyTorch Profiler 用于在 PyTorch 训练和在线推理过程中采集性能数据，帮助定位计算、通信、调度、显存等方面的性能问题。工具支持框架侧与 NPU 侧联合采集，也支持采集后自动解析或离线解析。

## 能力概览

- 采集 PyTorch 层算子信息、CANN 层信息和底层 NPU 算子信息。
- 支持 `profile`、`dynamic_profile`、`_KinetoProfile` 三种采集方式。
- 支持 mstx 打点、显存时间线、环境变量信息、自定义元数据、子线程采集等扩展能力。
- 支持生成文本结果、数据库结果，并可对接 MindStudio Insight 与 `msprof-analyze`。

## 阅读建议

- 新用户先看 [快速开始](./getting_started/quick_start.md)。
- 日常采集和调优优先看 [profile 接口采集](./user_guide/profile_api.md)。
- 长任务运行中触发采集时看 [dynamic_profile 动态采集](./user_guide/dynamic_profile.md)。
- 需要查参数字典时看 [参考说明](./reference/index.md)。
