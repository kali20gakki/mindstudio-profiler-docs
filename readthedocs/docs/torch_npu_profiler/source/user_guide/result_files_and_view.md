# 结果文件与查看方式

完成采集和解析后，可以从以下几个方向查看结果：

## 结果文件

- 文本结果：包括 `.json`、`.csv` 等 timeline 和 summary 文件。
- 数据库结果：`ascend_pytorch_profiler_{Rank_ID}.db`、`analysis.db`。
- 元数据文件：如 `profiler_metadata.json`。
- 原始性能数据：位于 `PROF_*`、`ASCEND_PROFILER_OUTPUT`、`FRAMEWORK` 等目录下。

## 查看工具

- MindStudio Insight：适合可视化查看 trace、内核、通信、显存等数据。
- `msprof-analyze`：适合基于 profiling 数据做进一步分析、比对和诊断。

## 相关说明

- 结果目录和字段详情可参考 [MindSpore&PyTorch 框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。
- 若只需要数据库格式结果，可在 `experimental_config.export_type` 中配置 `Db`。
