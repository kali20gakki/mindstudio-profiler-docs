# 前提条件与约束

## 前提条件

- 请先完成 [使用前准备](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0002.html#ZH-CN_TOPIC_0000002504358330)。
- 准备好基于 PyTorch 2.1.0 或更高版本开发的训练模型及配套数据集。
- 按照《[PyTorch 训练模型迁移调优指南](https://www.hiascend.com/document/detail/zh/Pytorch/730/ptmoddevg/trainingmigrguide/PT_LMTMOG_0002.html)》完成模型向昇腾环境的迁移。

## 通用约束

- `torch_npu.profiler` 支持多种采集方式，但不同采集方式不可同时开启。
- Profiler 接口的调用必须与被采集业务流程位于同一进程内。
- 性能数据可能占用较大磁盘空间，请提前确认结果目录有充足可用空间。

## 进程与 Device 关系

- 多进程多 Device 场景：支持每个 Device 单独设置一个采集进程。
- 单进程多 Device 场景：支持，但需配套 PyTorch `2.1.0post14`、`2.5.1post2`、`2.6.0` 及之后版本。
- 多进程单 Device 场景：要求多个采集动作串行执行，不能同时开始，并且每次采集都要包含完整的启动和停止过程。

## 选型建议

- 常规训练或在线推理采集，优先使用 `torch_npu.profiler.profile`。
- 任务运行中需要随时触发采集，使用 `dynamic_profile`。
- 只需要较基础的 trace 导出能力，可以使用 `_KinetoProfile`。
