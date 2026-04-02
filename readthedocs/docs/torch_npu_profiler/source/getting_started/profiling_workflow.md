# 性能采集完整流程

`torch_npu.profiler` 的典型使用流程如下：

1. 完成环境准备，确认版本和约束条件。
2. 选择采集方式：
   - 常规采集：`torch_npu.profiler.profile`
   - 动态采集：`dynamic_profile`
   - 基础采集：`torch_npu.profiler._KinetoProfile`
3. 配置采集参数并运行训练或在线推理任务。
4. 自动解析或离线解析采集结果。
5. 在 MindStudio Insight 中可视化分析，或结合 `msprof-analyze` 继续定位问题。

## 自动解析与离线解析

- 使用 `tensorboard_trace_handler` 时，可以在采集结束后自动解析结果。
- 当数据量较大、共享存储压力较高，或任务执行中断只保留了原始数据时，建议关闭自动解析并改用 [离线解析](../user_guide/offline_analyse.md)。

## 结果查看

- 结果文件结构与字段介绍可参考 [结果文件与查看方式](../user_guide/result_files_and_view.md)。
- 若需要数据库格式结果配合 Insight 查看，可在 `experimental_config.export_type` 中启用 `Db`。
