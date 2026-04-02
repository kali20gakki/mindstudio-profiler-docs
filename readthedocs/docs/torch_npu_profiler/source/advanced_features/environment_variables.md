# 环境变量信息采集

通过 `torch_npu.profiler` 采集性能数据时，默认会采集部分环境变量信息。

## 当前支持的环境变量

- `ASCEND_GLOBAL_LOG_LEVEL`
- `HCCL_RDMA_TC`
- `HCCL_RDMA_SL`
- `ACLNN_CACHE_LIMIT`

## 示例

```bash
export ASCEND_GLOBAL_LOG_LEVEL=1
export HCCL_RDMA_TC=0
export HCCL_RDMA_SL=0
export ACLNN_CACHE_LIMIT=4096
```

## 结果位置

- 当 `export_type=Text` 时，环境变量信息保存在 `profiler_metadata.json` 和 `ascend_pytorch_profiler_{Rank_ID}.db` 的 `META_DATA` 表中。
- 当 `export_type=Db` 时，环境变量信息写入 `ascend_pytorch_profiler_{Rank_ID}.db` 的 `META_DATA` 表中。
