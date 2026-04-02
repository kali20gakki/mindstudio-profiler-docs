# Profiler 子线程采集

在推理场景中，单进程多线程调用 torch 算子的用法较常见。由于 Profiler 默认无法感知用户自行创建的子线程，因此需要在子线程中显式调用接口注册采集回调。

## 示例

```python
import threading
import torch
import torch_npu


def infer(device, child_thread):
    torch.npu.set_device(device)

    if child_thread:
        torch_npu.profiler.profile.enable_profiler_in_child_thread(with_modules=True)

    for _ in range(5):
        outputs = model(input_data)

    if child_thread:
        torch_npu.profiler.profile.disable_profiler_in_child_thread()


if __name__ == "__main__":
    prof = torch_npu.profiler.profile(
        activities=[torch_npu.profiler.ProfilerActivity.CPU, torch_npu.profiler.ProfilerActivity.NPU],
        on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
        record_shapes=True,
        profile_memory=True,
        with_modules=True,
    )

    prof.start()

    threads = []
    for i in range(1, 3):
        t = threading.Thread(target=infer, args=(i, True))
        t.start()
        threads.append(t)

    infer(0, False)

    for t in threads:
        t.join()

    prof.stop()
```

## 说明

- `enable_profiler_in_child_thread()` 与 `disable_profiler_in_child_thread()` 必须配对使用。
- 子线程采集主要补充框架侧算子信息，不替代主线程 Profiler 的常规采集配置。
