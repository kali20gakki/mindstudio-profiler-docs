# `profiler_config.json` 参考

`profiler_config.json` 主要用于 `dynamic_profile` 场景。以下是默认配置示例：

```json
{
  "activities": ["CPU", "NPU"],
  "prof_dir": "./",
  "analyse": false,
  "record_shapes": false,
  "profile_memory": false,
  "with_stack": false,
  "with_flops": false,
  "with_modules": false,
  "active": 1,
  "warmup": 0,
  "start_step": 0,
  "is_rank": false,
  "rank_list": [],
  "experimental_config": {
    "profiler_level": "Level0",
    "aic_metrics": "AiCoreNone",
    "l2_cache": false,
    "op_attr": false,
    "gc_detect_threshold": null,
    "data_simplification": true,
    "record_op_args": false,
    "export_type": ["text"],
    "mstx": false,
    "mstx_domain_include": [],
    "mstx_domain_exclude": [],
    "host_sys": [],
    "sys_io": false,
    "sys_interconnection": false
  }
}
```

## 主要字段

| 参数 | 说明 |
| --- | --- |
| `start_step` | 开始采集的 step。`0` 表示不采集，`-1` 表示保存配置后的下个 step 开始采集。 |
| `activities` | 采集 CPU、NPU 事件列表。 |
| `prof_dir` | 采集结果落盘目录。 |
| `analyse` | 是否自动解析。 |
| `record_shapes` | 是否采集输入 shape 和 type。 |
| `profile_memory` | 是否采集显存占用。 |
| `with_stack` | 是否采集调用栈。 |
| `with_flops` | 是否采集浮点操作信息。 |
| `with_modules` | 是否采集 modules 层级 Python 调用栈。 |
| `active` | 实际采集的 step 数。 |
| `warmup` | 预热 step 数。 |
| `is_rank` | 是否开启指定 Rank 采集。 |
| `rank_list` | 需要采集的 Rank ID 列表。 |
| `async_mode` | 是否开启异步解析。 |
| `experimental_config` | 扩展采集配置。 |
| `metadata` | 自定义元数据。 |

## dynamic_profile 维测日志

启用 `dynamic_profile` 后，会在 `profiler_config_path` 下自动生成日志目录，典型结构如下：

```text
profiler_config_path/
├── log
│   ├── dp_ubuntu_xxxxxx_rank_*.log
│   ├── dp_ubuntu_xxxxxx_rank_*.log.1
│   ├── monitor_dp_ubuntu_xxxxxx_rank_*.log
│   ├── monitor_dp_ubuntu_xxxxxx_rank_*.log.1
├── profiler_config.json
└── shm
```

- `dp_*.log`：记录动态采集执行过程中的 INFO、WARNING、ERROR。
- `monitor_dp_*.log`：记录 `profiler_config.json` 每次修改是否生效，以及 dynamic_profile 进程的结束。
- `shm`：Python 3.7 环境下用于共享内存映射的目录。程序异常终止时，需要手动清理残留文件。
