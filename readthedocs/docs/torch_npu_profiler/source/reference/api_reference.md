# 接口参考

## `torch_npu.profiler.profile` 主要参数

| 参数名称 | 说明 | 是否必选 |
| --- | --- | --- |
| `activities` | CPU、NPU 事件采集列表。默认同时开启 CPU 和 NPU。 | 否 |
| `schedule` | 设置不同 step 的采集行为。`_KinetoProfile` 不支持。 | 否 |
| `on_trace_ready` | 采集结束时自动执行的回调，常用为 `tensorboard_trace_handler`。 | 否 |
| `record_shapes` | 是否采集算子 `InputShapes` 和 `InputTypes`。 | 否 |
| `profile_memory` | 是否采集显存占用信息。 | 否 |
| `with_stack` | 是否采集调用栈。 | 否 |
| `with_modules` | 是否采集 modules 层级的 Python 调用栈。 | 否 |
| `with_flops` | 是否采集浮点操作信息。当前暂不支持解析。 | 否 |
| `experimental_config` | 扩展采集配置。 | 否 |
| `use_cuda` | 昇腾环境不支持。 | 否 |

## 主要方法

| 方法名 | 说明 |
| --- | --- |
| `step()` | 划分不同迭代。 |
| `export_chrome_trace(path)` | 导出 `.json` trace 文件。 |
| `export_stacks(path, metric)` | 导出堆栈信息，用于 FlameGraph 等工具。 |
| `export_memory_timeline(output_path, device)` | 导出显存时间线文件。 |
| `start()` | 设置采集开始位置。 |
| `stop()` | 设置采集结束位置。 |
| `enable_profiler_in_child_thread()` | 注册子线程采集回调。 |
| `disable_profiler_in_child_thread()` | 注销子线程采集回调。 |

## 其他常用接口

| 类或函数 | 说明 |
| --- | --- |
| `torch_npu.profiler.schedule` | 设置不同 step 的采集行为。 |
| `torch_npu.profiler.tensorboard_trace_handler` | 导出并可自动解析性能数据。 |
| `torch_npu.profiler.ProfilerAction` | Profiler 状态枚举。 |
| `torch_npu.profiler._ExperimentalConfig` | 扩展采集配置。 |
| `torch_npu.profiler.supported_activities` | 查询当前支持的 activities。 |
| `torch_npu.profiler.supported_profiler_level` | 查询支持的 `profiler_level`。 |
| `torch_npu.profiler.supported_ai_core_metrics` | 查询支持的 AI Core 指标。 |
| `torch_npu.profiler.supported_export_type` | 查询支持的导出类型。 |

## 补充说明

- 使用共享存储的多卡大集群场景中，若直接通过 `on_trace_ready` 落盘，可能导致额外性能膨胀，可根据实际情况改用离线解析。
- `profile_memory` 在部分 `glibc < 2.34` 环境中可能触发已知问题，必要时建议升级 `glibc`。
