# 使用 `_KinetoProfile` 采集

`torch_npu.profiler._KinetoProfile` 提供基础的性能数据采集能力，适合只需要较简化 trace 导出的场景。

## 示例

```python
import torch
import torch_npu


prof = torch_npu.profiler._KinetoProfile(
    activities=None,
    record_shapes=False,
    profile_memory=False,
    with_stack=False,
    with_flops=False,
    with_modules=False,
    experimental_config=None,
)

for epoch in range(epochs):
    train_model_step()
    if epoch == 0:
        prof.start()
    if epoch == 1:
        prof.stop()

prof.export_chrome_trace("result_dir/trace.json")
```

## 限制

- 不支持 `schedule`。
- 不支持 `tensorboard_trace_handler`。
- 不支持 `step()`、`enable_profiler_in_child_thread()`、`disable_profiler_in_child_thread()`。

## 解析与查看

- 该方式通常通过 `export_chrome_trace()` 直接导出 trace。
- 结果查看方式与其他采集方式一致，详见 [结果文件与查看方式](./result_files_and_view.md)。
