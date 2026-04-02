# 自定义元数据

你可以在采集过程中写入自定义字符串或 JSON 元数据，便于在结果目录中附加模型超参、配置、实验标签等信息。

## 示例

```python
with torch_npu.profiler.profile(...) as prof:
    prof.add_metadata(key, value)
```

```python
with torch_npu.profiler._KinetoProfile(...) as prof:
    prof.add_metadata_json(key, value)
```

## 说明

- `add_metadata` 用于写入字符串键值对。
- `add_metadata_json` 用于写入 JSON 格式字符串。
- 这两个接口都应在 Profiler 初始化之后、`finalize` 之前调用。
- 结果会写入采集结果根目录下的 `profiler_metadata.json`。
