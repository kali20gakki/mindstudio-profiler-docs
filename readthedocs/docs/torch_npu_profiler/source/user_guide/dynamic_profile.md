# dynamic_profile 动态采集

`dynamic_profile` 允许在训练或在线推理任务运行过程中随时启动采集，适合长时间运行任务和需要按需触发采集的场景。

以下三种方式只能选择一种使用，不能同时启用两种及以上方式。

## 动态采集方式对比

| 方式 | 适用场景 | 特点 |
| --- | --- | --- |
| 环境变量方式 | 仅训练场景 | 不需要修改用户代码，通过配置文件触发采集。 |
| `dp.init()` 方式 | 训练和在线推理 | 需要在脚本中接入 `dynamic_profile`，通过修改配置文件触发采集。 |
| `dp.start()` 方式 | 训练和在线推理 | 在代码中预置精确触发点，适合缩小采集范围。 |

## 环境变量方式

### 步骤

1. 配置环境变量：

```bash
export PROF_CONFIG_PATH="/path/to/profiler_config_path"
```

2. 启动训练任务。`dynamic_profile` 会在 `profiler_config_path` 下自动生成模板文件 `profiler_config.json`。
3. 重新开启一个终端，修改 `profiler_config.json` 使能 Profiling。
4. 采集结束后，选择自动解析或手动解析结果。

### 说明

- 仅支持训练场景。
- 不支持采集第一个迭代 `step0`。
- 依赖 PyTorch 原生 `Optimizer.step()` 划分 step，不支持自定义 Optimizer。
- `PROF_CONFIG_PATH` 路径要求有读写权限，且路径仅支持字母、数字、下划线和连字符，不支持软链接。

## `dp.init()` 方式

```python
from torch_npu.profiler import dynamic_profile as dp

dp.init("profiler_config_path")

for step in steps:
    train_one_step()
    dp.step()
```

### 说明

- `dp.init()` 执行时会在 `profiler_config_path` 下自动创建模板文件 `profiler_config.json`。
- 运行任务后，可在新的终端中修改 `profiler_config.json`，动态触发采集。
- `dynamic_profile` 会轮询配置文件变化；若采集执行期间再次修改配置，则会在当前采集结束后启动最后一次修改对应的采集任务。

## `dp.start()` 方式

```python
from torch_npu.profiler import dynamic_profile as dp

dp.init("profiler_config_path")

for step in steps:
    if step == 5:
        dp.start("/home/xx/start_config_path/profiler_config.json")
    train_one_step()
    dp.step()
```

### 说明

- `dp.start()` 适合把采集范围缩小到指定代码位置附近。
- `start_config_path` 需要用户手动创建完整的 `profiler_config.json`。
- `dp.start()` 不感知后续配置文件修改，在一次任务执行过程中只触发一次采集。
- 若 `dp.init()` 触发的动态采集已经在执行中，`dp.start()` 不会生效；若 `dp.init()` 对应采集已结束，则 `dp.start()` 可继续触发新的采集任务。

## 配置文件与日志

- 配置字段说明详见 [profiler_config.json 参考](../reference/profiler_config_reference.md)。
- 维测日志说明详见 [dynamic_profile 维测日志](../reference/profiler_config_reference.md#dynamic_profile-维测日志)。

## 使用建议

- 推荐使用共享存储保存 `profiler_config_path`。
- `start_step` 应大于当前已执行 step，且不超过最大 step。
- 结果查看方式与常规采集一致，详见 [结果文件与查看方式](./result_files_and_view.md)。
