# 快速开始

推荐优先使用 `torch_npu.profiler.profile` 完成第一次采集。该方式能力最完整，也最适合作为后续调优的基础用法。

## 示例

```python
import torch
import torch_npu


experimental_config = torch_npu.profiler._ExperimentalConfig(
    export_type=[torch_npu.profiler.ExportType.Text],
    profiler_level=torch_npu.profiler.ProfilerLevel.Level0,
    aic_metrics=torch_npu.profiler.AiCMetrics.AiCoreNone,
    l2_cache=False,
    op_attr=False,
    data_simplification=False,
    record_op_args=False,
    gc_detect_threshold=None,
    host_sys=[],
    sys_io=False,
    sys_interconnection=False,
)

with torch_npu.profiler.profile(
    activities=[
        torch_npu.profiler.ProfilerActivity.CPU,
        torch_npu.profiler.ProfilerActivity.NPU,
    ],
    schedule=torch_npu.profiler.schedule(
        wait=0, warmup=0, active=1, repeat=1, skip_first=1
    ),
    on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
    record_shapes=False,
    profile_memory=False,
    with_stack=False,
    with_modules=False,
    with_flops=False,
    experimental_config=experimental_config,
) as prof:
    for step in range(steps):
        train_one_step(step, steps, train_loader, model, optimizer, criterion)
        prof.step()
```

## 采集后做什么

1. 运行训练或在线推理任务。
2. 在 `./result` 下查看生成的性能数据目录。
3. 使用 MindStudio Insight 可视化查看，或继续使用 [`msprof-analyze`](../../msprof-analyze/) 进行辅助分析。

## 下一步

- 需要理解参数和调用方式时，继续阅读 [profile 接口采集](../user_guide/profile_api.md)。
- 需要手动解析结果时，阅读 [离线解析](../user_guide/offline_analyse.md)。
