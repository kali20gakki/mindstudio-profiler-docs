# `schedule` 参考

`torch_npu.profiler.schedule` 用于设置不同 step 的采集行为，接口原型如下：

```python
torch_npu.profiler.schedule(wait, active, warmup=0, repeat=0, skip_first=0)
```

## 参数说明

| 参数 | 说明 |
| --- | --- |
| `wait` | 每次重复执行采集前跳过的 step 数。 |
| `active` | 实际采集的 step 数。 |
| `warmup` | 预热 step 数，默认 `0`，建议设置 `1`。 |
| `repeat` | 重复执行 `wait + warmup + active` 的次数，默认 `0`。 |
| `skip_first` | 正式采集前先跳过的 step 数，默认 `0`。 |

## 使用建议

- 动态 Shape 场景建议跳过前 10 轮，保证性能数据更稳定。
- 使用集群分析工具或 MindStudio Insight 时，建议配置 `repeat=1`，避免同一目录下生成多份性能数据。
- 建议满足公式：`step 总数 >= skip_first + (wait + warmup + active) * repeat`。

## 示例

```python
with torch_npu.profiler.profile(
    activities=[
        torch_npu.profiler.ProfilerActivity.CPU,
        torch_npu.profiler.ProfilerActivity.NPU,
    ],
    schedule=torch_npu.profiler.schedule(
        wait=1,
        warmup=1,
        active=2,
        repeat=2,
        skip_first=1,
    ),
    on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
) as prof:
    for _ in range(9):
        train_one_step()
        prof.step()
```
