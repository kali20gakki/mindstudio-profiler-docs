# 显存时间线

显存时间线用于在模型训练过程中展示不同类别数据对显存的占用情况，并导出为可视化 HTML 文件。

## 使用条件

- Python 环境中需要安装 `matplotlib`。
- `record_shapes=True`
- `profile_memory=True`
- `with_stack=True` 或 `with_modules=True`

## 示例

```python
import torch
import torch_npu


def trace_handler(prof: torch_npu.profiler.profile):
    prof.export_memory_timeline(output_path="./memory_timeline.html", device="npu:0")


with torch_npu.profiler.profile(
    activities=[
        torch_npu.profiler.ProfilerActivity.CPU,
        torch_npu.profiler.ProfilerActivity.NPU,
    ],
    schedule=torch_npu.profiler.schedule(wait=0, warmup=0, active=4, repeat=1, skip_first=0),
    on_trace_ready=trace_handler,
    record_shapes=True,
    profile_memory=True,
    with_stack=True,
    with_modules=True,
) as prof:
    for _ in range(steps):
        prof.step()
```

## 图中常见类别

- `PARAMETER`：模型参数、模型权重。
- `OPTIMIZER_STATE`：优化器状态。
- `INPUT`：输入数据。
- `TEMPORARY`：算子执行中的临时内存。
- `ACTIVATION`：前向激活值。
- `GRADIENT`：梯度。
- `AUTOGRAD_DETAIL`：反向计算过程中产生的内存占用。
- `UNKNOWN`：未知类型。
