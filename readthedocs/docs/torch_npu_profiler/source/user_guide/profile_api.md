# 使用 `torch_npu.profiler.profile` 采集

`torch_npu.profiler.profile` 是最推荐的采集方式，支持 `schedule`、`prof.step()`、自动解析、显存时间线、mstx、子线程采集等完整能力。

## 方式一：使用 `with` 语句

该方式会自动创建和释放 Profiler，适合最常见的训练和推理采集场景。

```python
import torch
import torch_npu


experimental_config = torch_npu.profiler._ExperimentalConfig(
    export_type=[torch_npu.profiler.ExportType.Text],
    profiler_level=torch_npu.profiler.ProfilerLevel.Level0,
    mstx=False,
    mstx_domain_include=[],
    mstx_domain_exclude=[],
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
    schedule=torch_npu.profiler.schedule(wait=0, warmup=0, active=1, repeat=1, skip_first=1),
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

## 方式二：手动控制 `start()` 和 `stop()`

该方式适合需要精确控制采集起止位置的场景。

```python
import torch
import torch_npu


experimental_config = torch_npu.profiler._ExperimentalConfig(
    export_type=[torch_npu.profiler.ExportType.Text],
    profiler_level=torch_npu.profiler.ProfilerLevel.Level0,
    mstx=False,
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

prof = torch_npu.profiler.profile(
    activities=[
        torch_npu.profiler.ProfilerActivity.CPU,
        torch_npu.profiler.ProfilerActivity.NPU,
    ],
    schedule=torch_npu.profiler.schedule(wait=0, warmup=0, active=1, repeat=1, skip_first=1),
    on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
    record_shapes=False,
    profile_memory=False,
    with_stack=False,
    with_modules=False,
    with_flops=False,
    experimental_config=experimental_config,
)

prof.start()
for step in range(steps):
    train_one_step()
    prof.step()
prof.stop()
```

## 导出单个 trace 文件

如果只需要导出单个 `chrome_trace_{pid}.json` 文件，可以在采集结束后调用 `export_chrome_trace()`：

```python
with torch_npu.profiler.profile() as prof:
    for step in range(steps):
        train_one_step(step, steps, train_loader, model, optimizer, criterion)

prof.export_chrome_trace("./chrome_trace_14.json")
```

`tensorboard_trace_handler` 导出的结果包含更完整的数据，通常优先推荐使用该方式。

## 解析与查看

- 自动解析：通过 `tensorboard_trace_handler` 或 `export_chrome_trace` 直接输出解析结果。
- 离线解析：详见 [离线解析](./offline_analyse.md)。
- 结果查看：详见 [结果文件与查看方式](./result_files_and_view.md)。

## 使用建议

- 建议显式配置 `schedule`，并配合 `prof.step()` 使用，以便采集更稳定的 step 数据。
- 多卡大集群且使用共享存储时，直接通过 `on_trace_ready` 落盘可能带来额外性能膨胀，应根据场景评估是否改用离线解析。
- 参数字典与 `schedule` 说明请参考 [接口参考](../reference/api_reference.md) 和 [schedule 参考](../reference/schedule_reference.md)。
