# `experimental_config` 参考

`experimental_config` 用于配置扩展采集项。为了避免混淆，建议按使用场景理解：

- Python API 场景：通过 `torch_npu.profiler._ExperimentalConfig(...)` 配置。
- `dynamic_profile` 场景：通过 `profiler_config.json` 中的 `experimental_config` 字段配置。

## 主要参数

| 参数 | 说明 |
| --- | --- |
| `export_type` | 导出结果格式，支持 `Text`/`Db` 或配置文件中的 `text`/`db`。 |
| `profiler_level` | 采集等级，支持 `Level_none`、`Level0`、`Level1`、`Level2`。 |
| `aic_metrics` | AI Core 性能指标采集项。 |
| `l2_cache` | 是否采集 L2 Cache 数据。 |
| `op_attr` | 是否采集算子属性信息。 |
| `gc_detect_threshold` | GC 检测阈值，单位 ms。 |
| `data_simplification` | 是否开启数据精简模式。 |
| `record_op_args` | 是否输出算子信息统计结果。 |
| `mstx` / `msprof_tx` | 是否开启自定义打点。 |
| `mstx_domain_include` | 只输出指定 domain 的 mstx 数据。 |
| `mstx_domain_exclude` | 过滤指定 domain 的 mstx 数据。 |
| `host_sys` | 是否采集 Host 侧 CPU、内存、磁盘、网络、OSRT 等系统数据。 |
| `sys_io` | 是否采集 NIC、ROCE、MAC 数据。 |
| `sys_interconnection` | 是否采集集合通信带宽、PCIe、片间传输带宽信息。 |

## `profiler_level` 建议

- `Level0`：默认值，适合大多数常规采集场景。
- `Level1`：在 `Level0` 基础上增加 AscendCL 和 AI Core 指标，适合进一步分析计算与通信细节。
- `Level2`：在 `Level1` 基础上增加 Runtime 和 AI CPU 数据，适合更深入的问题定位。
- `Level_none`：关闭层级数据采集，常与 mstx 场景配合使用。

## 注意事项

- `mstx_domain_include` 与 `mstx_domain_exclude` 互斥。
- `record_op_args` 主要面向 AOE 调优场景，不建议与其他性能采集接口同时开启。
- `host_sys` 中的部分采集项依赖第三方工具，如 `iotop`、`perf`、`ltrace`，并需要额外权限配置。
