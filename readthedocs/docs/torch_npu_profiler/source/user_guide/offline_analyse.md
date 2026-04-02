# 离线解析

当采集数据较大、当前环境不适合直接自动解析，或采集过程中仅保留了原始数据时，建议使用离线解析。

## 使用方法

1. 创建一个 Python 文件，例如 `analyse_profiler.py`。
2. 写入如下代码：

```python
from torch_npu.profiler.profiler import analyse


if __name__ == "__main__":
    analyse(
        profiler_path="./result_data",
        max_process_number=max_process_number,
        export_type=export_type,
    )
```

3. 执行：

```bash
python3 analyse_profiler.py
```

## 参数说明

| 参数 | 说明 |
| --- | --- |
| `profiler_path` | 必选。性能数据目录，目录下保存 `{worker_name}_{时间戳}_ascend_pt`。 |
| `max_process_number` | 可选。离线解析最大进程数，取值范围为 `1 ~ CPU 核数`。 |
| `export_type` | 可选。导出格式，支持 `text` 和 `db`。未配置时读取 `profiler_info.json` 中的配置。 |

## 使用建议

- 离线解析支持多个性能数据目录并行解析，但数据量较大时可能占用较多内存，可通过 `max_process_number` 控制资源占用。
- 解析过程日志保存在结果目录下的 `logs` 子目录。
- 解析完成后，可继续使用 MindStudio Insight 或 `msprof-analyze` 进行分析。
