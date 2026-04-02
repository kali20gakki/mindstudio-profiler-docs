> 说明：
> 本文档为历史整合版本，内容已按专题拆分到 `torch_npu_profiler` 文档集。
> 建议优先阅读 `index.md` 以及 `source/` 目录下的入门、使用指南、高级特性和参考说明章节。

# Ascend Pytorch调优工具性能数据采集和自动解析

Ascend PyTorch调优工具（Ascend PyTorch Profiler）是针对PyTorch框架开发的性能分析工具，通过在PyTorch训练/在线推理脚本中添加Ascend PyTorch Profiler，执行训练/在线推理的同时采集性能数据，完成训练/在线推理后直接输出可视化的性能数据文件，提升了性能分析效率。Ascend PyTorch Profiler可全面采集PyTorch训练/在线推理场景下的性能数据，主要包括PyTorch层算子信息、CANN层算子信息、底层NPU算子信息、以及算子内存占用信息等，可以全方位分析PyTorch训练/在线推理时的性能状态。

Ascend PyTorch Profiler当前支持如下性能数据采集方式：

- torch_npu.profiler.profile接口采集

  提供完整的采集接口，通过在代码中添加接口，可以自由选择采集的内容。

- dynamic_profile动态采集

  支持在训练过程中随时启动采集，支持不修改用户代码直接启动采集，采集方式更灵活。

- torch_npu.profiler._KinetoProfile接口采集

  基础的性能数据采集方式。

其他相关功能：

- [（可选）采集并解析mstx数据](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5940122172516)
- [（可选）采集环境变量信息](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section18689133819128)
- [（可选）以自定义字符串键和字符串值的形式标记性能数据采集过程](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section18921849102116)
- [（可选）显存可视化](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section206001142152910)
- [（可选）创建Profiler子线程采集](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1837159184120)

参考信息：

- [Ascend PyTorch Profiler接口说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5699454151510)
- [profiler_config.json文件说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1237447165314)
- [experimental_config参数说明（dynamic_profile动态采集场景）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section2015623185118)
- [experimental_config参数说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1548285513313)
- [torch_npu.profiler.schedule类参数说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5522846122614)
- [dynamic_profile动态采集维测日志介绍](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section4954228201816)

#### 约束

- Ascend PyTorch Profiler接口支持多种采集方式，各采集方式不可同时开启。
- 须保证Ascend PyTorch Profiler接口的调用与要采集的业务流程在同一个进程内。
- Ascend PyTorch Profiler接口进行采集任务时，进程与Device之间的关系如下：
  - 多进程多Device场景：支持每个Device下分别设置一个采集进程。
  - 单进程多Device场景：支持。须配套PyTorch 2.1.0post14、2.5.1post2、2.6.0及之后的版本。
  - 多进程单Device场景：需要保证多进程之间的采集动作是串行的，即各个采集动作不在同一时间开始，且各个采集动作须包含完整的启动和停止。
- 性能数据会占据一定的磁盘空间，可能存在磁盘写满导致服务器不可用的风险。性能数据所需空间跟模型的参数、采集开关配置、采集的迭代数量有较大关系，须用户自行保证落盘目录下的可用磁盘空间。

#### 前提条件

- 请确保完成[使用前准备](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0002.html#ZH-CN_TOPIC_0000002504358330)。
- 准备好基于PyTorch 2.1.0或更高版本开发的训练模型以及配套的数据集，并按照《[PyTorch 训练模型迁移调优指南](https://www.hiascend.com/document/detail/zh/Pytorch/730/ptmoddevg/trainingmigrguide/PT_LMTMOG_0002.html)》中的“[模型迁移](https://www.hiascend.com/document/detail/zh/Pytorch/730/ptmoddevg/trainingmigrguide/PT_LMTMOG_0013.html)”完成PyTorch原始模型向昇腾AI处理器的迁移。



#### 采集并解析性能数据（torch_npu.profiler.profile）

1. 在训练脚本（如train_*.py文件）/在线推理脚本内添加如下示例代码进行性能数据采集参数配置，之后启动训练/在线推理。

   说明

   - 以下示例代码中的torch_npu.profiler.profile接口详细介绍请参见[Ascend PyTorch Profiler接口说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5699454151510)。
   - 以下给出两个示例代码，使用不同方式调用**torch_npu.profiler.profile**接口，可任选其一使用。

   - 示例一：使用with语句调用torch_npu.profiler.profile接口，自动创建Profiler，采集with范围内代码段的性能数据。

     ```
     import torch
     import torch_npu
     
     
     
     # 添加Profiling采集扩展配置参数，详细参数介绍可参考下文的参数说明
     experimental_config = torch_npu.profiler._ExperimentalConfig(
         export_type=[
             torch_npu.profiler.ExportType.Text
             ],
         profiler_level=torch_npu.profiler.ProfilerLevel.Level0,
         mstx=False,    # 原参数名msprof_tx改为mstx，新版本依旧兼容原参数名msprof_tx
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
         sys_interconnection=False
     )
     
     # 添加Profiling采集基础配置参数，详细参数介绍可参考下文的参数说明
     with torch_npu.profiler.profile(
         activities=[
             torch_npu.profiler.ProfilerActivity.CPU,
             torch_npu.profiler.ProfilerActivity.NPU
             ],
         schedule=torch_npu.profiler.schedule(wait=0, warmup=0, active=1, repeat=1, skip_first=1),    # 与prof.step()配套使用
         on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
         record_shapes=False,
         profile_memory=False,
         with_stack=False,
         with_modules=False,
         with_flops=False,
         experimental_config=experimental_config) as prof:
     
         # 启动性能数据采集
         for step in range(steps):
             train_one_step(step, steps, train_loader, model, optimizer, criterion)
             prof.step()    # 与schedule配套使用
     ```

   - 示例二：创建torch_npu.profiler.profile对象

     ，

     通过start和stop接口控制采集性能数据，用户可自定义采集启动的位置。

     ```
     import torch
     import torch_npu
     
     
     # 添加Profiling采集扩展配置参数，详细参数介绍可参考下文的参数说明
     experimental_config = torch_npu.profiler._ExperimentalConfig(
         export_type=[
             torch_npu.profiler.ExportType.Text
             ],
         profiler_level=torch_npu.profiler.ProfilerLevel.Level0,
         mstx=False,    # 原参数名msprof_tx改为mstx，新版本依旧兼容原参数名msprof_tx
         aic_metrics=torch_npu.profiler.AiCMetrics.AiCoreNone,
         l2_cache=False,
         op_attr=False,
         data_simplification=False,
         record_op_args=False,
         gc_detect_threshold=None,
         host_sys=[],
         sys_io=False,
         sys_interconnection=False
     )
     
     # 添加Profiling采集基础配置参数，详细参数介绍可参考下文的参数说明
     prof = torch_npu.profiler.profile(
         activities=[
             torch_npu.profiler.ProfilerActivity.CPU,
             torch_npu.profiler.ProfilerActivity.NPU
             ],
         schedule=torch_npu.profiler.schedule(wait=0, warmup=0, active=1, repeat=1, skip_first=1),
         on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
         record_shapes=False,
         profile_memory=False,
         with_stack=False,
         with_modules=False,
         with_flops=False,
         experimental_config=experimental_config)
     
     prof.start()    # 启动性能数据采集
     for step in range(steps):
         train_one_step()
         prof.step()    # 与schedule配套使用
     prof.stop()    # 结束性能数据采集
     ```

   以上两个示例主要使用tensorboard_trace_handler导出性能数据，也可以使用以下prof.export_chrome_trace方式导出单个性能文件“chrome_trace_{pid}.json”。由于tensorboard_trace_handler导出的性能数据包含了prof.export_chrome_trace导出的性能数据，所以根据实际需求选择一种导出方式即可。

   ```
   import torch
   import torch_npu
   
   
   
   with torch_npu.profiler.profile() as prof:
   
       # 启动性能数据采集
       for step in range(steps):
           train_one_step(step, steps, train_loader, model, optimizer, criterion)
   prof.export_chrome_trace('./chrome_trace_14.json')
   ```

2. 性能数据解析。

   支持自动解析（参照以上示例代码中**tensorboard_trace_handler**和**prof.****export_chrome_trace**）和[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)。

3. 查看性能数据结果文件和性能数据分析。

   性能数据结果文件详细介绍请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。

   请参见《[MindStudio Insight工具用户指南](https://www.hiascend.com/document/detail/zh/mindstudio/830/GUI_baseddevelopmenttool/msascendinsightug/Insight_userguide_0002.html)》将解析后的性能数据文件进行可视化展示和分析。

   可以使用[性能分析工具（msprof-analyze）](https://gitcode.com/Ascend/mstt/tree/master/profiler/msprof_analyze)辅助分析性能数据。



#### 采集并解析性能数据（dynamic_profile）

dynamic_profile动态采集，主要功能是在执行模型训练/在线推理过程中可以随时开启采集进程。

以下方式只选择一种使用，不可同时使用两种及以上方式开启dynamic_profile。

展开

| 动态采集方式                                                 | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 环境变量方式                                                 | 仅支持训练场景，通过修改配置文件profiler_config.json控制Profiling配置，不需要修改用户代码。 |
| 修改用户训练/在线推理脚本，添加dynamic_profile接口方式       | 支持训练/在线推理场景，通过修改配置文件profiler_config.json控制Profiling的启动，需要预先在用户脚本中添加dynamic_profile接口。 |
| 修改用户训练/在线推理脚本，添加dynamic_profile的dp.start()函数方式 | 支持训练/在线推理场景，通过预先在用户脚本中添加dp.start()接口控制Profiling的启动，可自定义dp.start()接口添加的位置，适合在需要缩小采集范围时使用。 |

**环境变量方式**

1. 配置如下环境变量。

   ```
   export PROF_CONFIG_PATH="/path/to/profiler_config_path"
   ```

   配置该环境变量后启动训练，dynamic_profile会在profiler_config_path下自动创建模板文件profiler_config.json，用户可基于模板文件自定义修改配置项。

   说明

   - 该方式仅支持训练场景。
   - 该方式下dynamic_profile不支持采集第一个迭代（step0）的数据。
   - 该方式依赖torch原生Optimizer.step()划分训练过程中Profiling的step，不支持自定义Optimizer场景。
   - PROF_CONFIG_PATH指定的路径可自定义（要求有读写权限），路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接，例如"/home/xxx/profiler_config_path"。

2. 启动训练任务。

3. 重新开启一个命令行窗口，修改profiler_config.json配置文件用以使能Profiling任务。

   配置文件中包含Profiler的性能数据采集参数，用户可以参考[profiler_config.json文件说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1237447165314)修改配置文件参数来执行不同的Profiling任务。

   说明

   - dynamic_profile通过识别profiler_config.json文件的状态判断文件是否被修改：
     - dynamic_profile每2s轮询一次，若发现profiler_config.json文件改动，则启动采集流程，之后记录相邻step之间的运行间隔，将此时间作为新的轮询时间，最小值为1s。
     - 若在dynamic_profile采集执行期间，profiler_config.json文件被修改，则在采集进程结束之后，再次启动最后一次文件修改的dynamic_profile采集。
   - 建议用户使用共享存储设置dynamic_profile的profiler_config_path。
   - profiler_config_path目录下会自动记录dynamic_profile的维测日志，详细介绍请参见[dynamic_profile动态采集维测日志介绍](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section4954228201816)。
   - 修改start_step参数值应大于当前已执行到的训练/在线推理step，且不超过最大step。例如step总数为10，且已执行到step3，则start_step只能配置在3~10之间，且注意由于配置过程中训练/在线推理任务还在持续执行，故配置大于step3。

4. 性能数据解析。

   支持自动解析和手动解析，请参见[表6](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_table10278112415418)中的analyse参数。

5. 查看性能数据结果文件和性能数据分析。

   性能数据结果文件详细介绍请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。

   请参见《[MindStudio Insight工具用户指南](https://www.hiascend.com/document/detail/zh/mindstudio/830/GUI_baseddevelopmenttool/msascendinsightug/Insight_userguide_0002.html)》将解析后的性能数据文件进行可视化展示和分析。

   可以使用[性能分析工具（msprof-analyze）](https://gitcode.com/Ascend/mstt/tree/master/profiler/msprof_analyze)辅助分析性能数据。

**修改用户训练/在线推理脚本，添加dynamic_profile接口方式**

1. 在训练脚本（如train_*.py文件）/在线推理脚本中添加如下示例代码。

   ```
   # 加载dynamic_profile模块
   from torch_npu.profiler import dynamic_profile as dp
   # 设置Profiling配置文件的路径
   dp.init("profiler_config_path")
   
   for step in steps:
       train_one_step()
       # 划分step
       dp.step()
   ```

   **init**时，dynamic_profile会在profiler_config_path下自动创建模板文件profiler_config.json，用户可基于模板文件自定义修改配置项。

   profiler_config_path路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。

2. 启动训练/在线推理任务。

3. 重新开启一个命令行窗口，修改profiler_config.json配置文件用以使能Profiling任务。

   配置文件中包含Profiler的性能数据采集参数，用户可以参考[profiler_config.json文件说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1237447165314)修改配置文件参数来执行不同的Profiling任务。

   说明

   - dynamic_profile通过识别profiler_config.json文件的状态判断文件是否被修改：
     - dynamic_profile每2s轮询一次，若发现profiler_config.json文件改动，则启动采集流程，之后记录相邻step之间的运行间隔，将此时间作为新的轮询时间，最小值为1s。
     - 若在dynamic_profile采集执行期间，profiler_config.json文件被修改，则在采集进程结束之后，再次启动最后一次文件修改的dynamic_profile采集。
   - 建议用户使用共享存储设置dynamic_profile的profiler_config_path。
   - profiler_config_path目录下会自动记录dynamic_profile的维测日志，详细介绍请参见[dynamic_profile动态采集维测日志介绍](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section4954228201816)。
   - 修改start_step参数值应大于当前已执行到的训练/在线推理step，且不超过最大step。例如step总数为10，且已执行到step3，则start_step只能配置在3~10之间，且注意由于配置过程中训练/在线推理任务还在持续执行，故配置大于step3。

4. 支持自动解析和手动解析，请参见[表6](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_table10278112415418)中的analyse参数。

5. 查看性能数据结果文件和性能数据分析。

   性能数据结果文件详细介绍请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。

   请参见《[MindStudio Insight工具用户指南](https://www.hiascend.com/document/detail/zh/mindstudio/830/GUI_baseddevelopmenttool/msascendinsightug/Insight_userguide_0002.html)》将解析后的性能数据文件进行可视化展示和分析。

   可以使用[性能分析工具（msprof-analyze）](https://gitcode.com/Ascend/mstt/tree/master/profiler/msprof_analyze)辅助分析性能数据。

**修改用户训练/在线推理脚本，添加dynamic_profile的****dp.start()****函数方式**

1. 在训练脚本（如train_*.py文件）/在线推理脚本中添加如下示例代码。

   ```
   # 加载dynamic_profile模块
   from torch_npu.profiler import dynamic_profile as dp
   # 设置init接口Profiling配置文件路径
   dp.init("profiler_config_path")
   
   for step in steps:
       if step==5:
           # 设置start接口Profiling配置文件路径
           dp.start("start_config_path")
       train_one_step()
       # 划分step，需要进行profile的代码需在dp.start()接口和dp.step()接口之间
       dp.step()
   ```

   **start_config_path**同样指定为profiler_config.json，但需要用户根据[profiler_config.json文件说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1237447165314)手动创建配置文件并根据场景需要配置参数。此处须指定具体文件名，例如dp.start("/home/xx/start_config_path/profiler_config.json")。

   profiler_config_path和start_config_path路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。

   说明

   - 添加**dp.start()**后，当训练/在线推理任务进行到**dp.start()**时，会自动按照**start_config_path**指定的profiler_config.json文件进行采集。**dp.start()**函数不感知profiler_config.json文件的修改，只会在训练/在线推理过程中触发一次采集任务。

   - 添加

     dp.start()

     并启动训练/在线推理后：

     - 若**dp.start()**未指定profiler_config.json配置文件或配置文件因错误未生效，则执行到**dp.start()**后按照**profiler_config_path**目录下的profiler_config.json文件配置进行采集。
     - 若在**dp.init()**配置的dynamic_profile生效期间，脚本运行至**dp.start()**，则**dp.start()**不生效。
     - 若在**dp.init()**配置的dynamic_profile采集结束后，脚本运行至**dp.start()**，则继续执行**dp.start()**采集，并在prof_dir目录下生成新的性能数据文件目录**。**
     - 若在**dp.start()**配置的dynamic_profile生效期间，改动**profiler_config_path**目录下的profiler_config.json文件，**dp.init()**会等待**dp.start()**采集完成后启动，并在prof_dir目录下生成新的性能数据文件目录。

   - 建议用户使用共享存储设置dynamic_profile的profiler_config_path。

   - 修改start_step参数值应大于当前已执行到的训练/在线推理step，且不超过最大step。例如step总数为10，且已执行到step3，则start_step只能配置在3~10之间，且注意由于配置过程中训练/在线推理任务还在持续执行，故配置大于step3。

2. 启动训练/在线推理任务。

3. 性能数据解析。

   支持自动解析和手动解析，请参见[表6](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_table10278112415418)中的analyse参数。

4. 查看性能数据结果文件和性能数据分析。

   性能数据结果文件详细介绍请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。

   请参见《[MindStudio Insight工具用户指南](https://www.hiascend.com/document/detail/zh/mindstudio/830/GUI_baseddevelopmenttool/msascendinsightug/Insight_userguide_0002.html)》将解析后的性能数据文件进行可视化展示和分析。

   可以使用[性能分析工具（msprof-analyze）](https://gitcode.com/Ascend/mstt/tree/master/profiler/msprof_analyze)辅助分析性能数据。



#### 采集并解析性能数据（torch_npu.profiler._KinetoProfile）

1. 在训练脚本（如train_*.py文件）/在线推理脚本内添加如下示例代码进行性能数据采集参数配置，之后启动训练/在线推理。

   说明

   以下示例代码中的torch_npu.profiler._KinetoProfile接口详细介绍请参见[Ascend PyTorch Profiler接口说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5699454151510)。

   ```
   import torch
   import torch_npu
   
   
   
   prof = torch_npu.profiler._KinetoProfile(activities=None, record_shapes=False, profile_memory=False, with_stack=False, with_flops=False, with_modules=False, experimental_config=None)
   for epoch in range(epochs):
       train_model_step()
       if epoch == 0:
           prof.start()
       if epoch == 1:
           prof.stop()
   prof.export_chrome_trace("result_dir/trace.json")
   ```

   该方式不支持使用**schedule**和**tensorboard_trace_handler**导出性能数据。

2. 性能数据解析。

   支持自动解析（参照以上示例代码中**prof.****export_chrome_trace**）。

3. 查看性能数据结果文件和性能数据分析。

   性能数据结果文件详细介绍请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。

   请参见《[MindStudio Insight工具用户指南](https://www.hiascend.com/document/detail/zh/mindstudio/830/GUI_baseddevelopmenttool/msascendinsightug/Insight_userguide_0002.html)》将解析后的性能数据文件进行可视化展示和分析。

   可以使用[性能分析工具（msprof-analyze）](https://gitcode.com/Ascend/mstt/tree/master/profiler/msprof_analyze)辅助分析性能数据。



#### （可选）采集并解析mstx数据

针对大集群场景传统Profiling数据量大、分析流程复杂的现象，通过[experimental_config](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1548285513313)的**mstx**参数开启自定义打点功能，自定义采集时间段或者关键函数的开始和结束时间点，识别关键函数或迭代等信息，对性能问题快速定界。

使用方式及示例代码如下：

1. 使能torch_npu.profiler，打开mstx开关，搭配profiler_level开关设置为Level_none（可根据实际采集需要，配置对应的level）以及mstx_domain_include或mstx_domain_exclude的domain过滤属性，采集打点数据。
2. 在PyTorch脚本中对于想采集的事件调用torch_npu.npu.mstx、torch_npu.npu.mstx.mark、torch_npu.npu.mstx.range_start、torch_npu.npu.mstx.range_end、torch_npu.npu.mstx.mstx_range的打点mark接口实现打点，采集对应事件的耗时。接口详细介绍请参见《[Ascend Extension for PyTorch 自定义API参考](https://www.hiascend.com/document/detail/zh/Pytorch/730/apiref/torchnpuCustomsapi/docs/context/overview.md)》中的“Python接口 > torch_npu.npu > [profiler](https://www.hiascend.com/document/detail/zh/Pytorch/730/apiref/torchnpuCustomsapi/docs/context/torch_npu-npu-mstx.md)”。

只记录Host侧range耗时：

```
id = torch_npu.npu.mstx.range_start("dataloader", None)    # 第二个入参设置None或者不设置，只记录Host侧range耗时
dataloader()
torch_npu.npu.mstx.range_end(id)
```

在计算流上打点，记录Host侧range耗时和Device侧对应的range耗时：

```
stream = torch_npu.npu.current_stream()
id = torch_npu.npu.mstx.range_start("matmul", stream)    # 第二个入参设置有效的stream，记录Host侧range耗时和Device侧对应的range耗时
torch.matmul()    # 计算流操作示意
torch_npu.npu.mstx.range_end(id)
```

在集合通信流上打点：

```
from torch.distributed.distributed_c10d import _world

if (torch.__version__ != '1.11.0') :
    stream_id = _world.default_pg._get_backend(torch.device('npu'))._get_stream_id(False)
    collective_stream = torch.npu.Stream(stream_id=collective_stream_id, device_type=20, device_index=device_id)    # device_index设置实际业务的device_id值
else:
    stream_id = _world.default_pg._get_stream_id(False)
    current_stream = torch.npu.current_stream()
    cdata = current_stream._cdata & 0xffff000000000000
    collective_stream = torch.npu.Stream(_cdata=( stream_id + cdata), device_index=device_id)    # device_index设置实际业务的device_id值
id = torch_npu.npu.mstx.range_start("allreduce", collective_stream)    # 第二个入参设置有效的stream，记录Host侧range耗时和Device侧对应的range耗时
torch.allreduce()    # 集合通信流操作示意
torch_npu.npu.mstx.range_end(id)
```

在P2P通信流上打点：

```
from torch.distributed.distributed_c10d import _world
 
if (torch.__version__ != '1.11.0') :
    stream_id = _world.default_pg._get_backend(torch.device('npu'))._get_stream_id(True)
    p2p_stream = torch.npu.Stream(stream_id=collective_stream_id, device_type=20, device_index=device_id)    # device_index设置实际业务的device_id值
else:
    stream_id = _world.default_pg._get_stream_id(True)
    current_stream = torch.npu.current_stream()
    cdata = current_stream._cdata & 0xffff000000000000
    p2p_stream = torch.npu.Stream(_cdata=( stream_id + cdata), device_index=device_id)    # device_index设置实际业务的device_id值
id = torch_npu.npu.mstx.range_start("send", p2p_stream)    # 第二个入参设置有效的stream，记录Host侧range耗时和Device侧对应的range耗时
torch.send()
torch_npu.npu.mstx.range_end(id)
```

想要采集如上场景数据，需要配置torch_npu.profiler.profile接口，使能mstx开关，参考样例如下：

```
import torch
import torch_npu

experimental_config = torch_npu.profiler._ExperimentalConfig(
    profiler_level=torch_npu.profiler.ProfilerLevel.Level_none,
    mstx=True,    # 原参数名msprof_tx改为mstx，新版本依旧兼容原参数名msprof_tx
    export_type=[
        torch_npu.profiler.ExportType.Db
        ])
with torch_npu.profiler.profile(
    schedule=torch_npu.profiler.schedule(wait=1, warmup=1, active=2, repeat=1, skip_first=1),
    on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
    experimental_config=experimental_config) as prof:
       
    for step in range(steps):
        train_one_step()    # 用户代码，包含调用mstx接口
        prof.step()
```

根据domain域进行采集：

```
import torch
import torch_npu
import time
experimental_config = torch_npu.profiler._ExperimentalConfig(
    data_simplification=False,
    # 开启mstx，并设置mstx_domain_include或mstx_domain_exclude的domain过滤属性
    mstx=True,
    mstx_domain_include=['default','domain1']    # 配置采集'default'和'domain1'打点数据
    # mstx_domain_exclude=['domain2']    # 配置不采集'domain2'的打点数据，与mstx_domain_include不同时配置
)
with torch_npu.profiler.profile(
    activities=[torch_npu.profiler.ProfilerActivity.CPU, torch_npu.profiler.ProfilerActivity.NPU],
    schedule=torch_npu.profiler.schedule(wait=1, warmup=0, active=1, repeat=1, skip_first=1),
    on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
    experimental_config=experimental_config) as prof:
    for i in range(5):
        # 标记默认domain的mstx打点
        torch_npu.npu.mstx.mark("mark_with_default_domain")
        range_id = torch_npu.npu.mstx.range_start("range_with_default_domain")
        time.sleep(1)    # 模拟用户代码
        torch_npu.npu.mstx.range_end(range_id)
        ...    # 用户代码
        # 标记用户自定义domain1的mstx打点
        torch_npu.npu.mstx.mark("mark_with_domain1", domain = "domain1")
        range_id1 = torch_npu.npu.mstx.range_start("range_with_domain1", domain="domain1")
        time.sleep(1)    # 模拟用户代码
        torch_npu.npu.mstx.range_end(range_id1, domain="domain1")
        ...    # 用户代码
        # 标记用户自定义domain2的mstx打点
        torch_npu.npu.mstx.mark("mark_with_domain2", domain = "domain2")
        range_id2 = torch_npu.npu.mstx.range_start("range_with_domain2", domain="domain2")
        time.sleep(1)    # 模拟用户代码
        torch_npu.npu.mstx.range_end(range_id2, domain="domain2")
        prof.step()
```

打点数据使用MindStudio Insight工具打开，可视化效果如下：

**图1** 打点结果示例
![img](https://www.hiascend.com/doc_center/source/zh/mindstudio/830/T&ITools/Profiling/figure/zh-cn_image_0000002534478699.png)

mstx功能默认采集通信算子、dataloader耗时、保存检查点接口耗时的性能数据，数据内容格式分别为：

- 格式：{"streamId": "{pg streamId}","count": "{count}","dataType": "{dataType}",["srcRank": "{srcRank}"],["destRank": "{destRank}"],"groupName": "{groupName}","opName": "{opName}"}

  示例：{"streamId": "32","count": "25701386","dataType": "fp16","groupName": "group_name_43","opName": "HcclAllreduce"}

  - streamId：用于执行打点任务的Stream ID。
  - count：输入数据个数。
  - dataType：输入数据的数据类型。
  - srcRank：通信域内数据发送端的Rank ID，hcclRecv算子才有srcRank。
  - destRank：通信域内数据接收端的Rank ID，hcclSend算子才有destRank。
  - groupName：通信域名称。
  - opName：算子名称。

- dataloader

- save_checkpoint

此外，mstx功能还可以通过**mstx_torch_plugin**获取PyTorch模型中的dataloader、forward、step、save_checkpoint这四个关键阶段的性能数据，详细介绍请参见《[mstx_torch_plugin](https://gitcode.com/Ascend/mstt/tree/br_release_MindStudio_8.3.0_20261231/profiler/example/mstx_torch_plugin)》。

可以通过该功能查看用户自定义打点从框架侧到CANN层再到NPU侧的执行调度情况，进而帮助识别用户想观察的关键函数或者事件，定界性能问题。

mstx采集结果数据详细介绍请参见[msproftx数据说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0145.html#ZH-CN_TOPIC_0000002504198582)。



#### （可选）采集环境变量信息

通过Ascend PyTorch Profiler接口采集性能数据时，默认采集环境变量信息，当前支持采集的环境变量如下：

- "ASCEND_GLOBAL_LOG_LEVEL"
- "HCCL_RDMA_TC"
- "HCCL_RDMA_SL"
- "ACLNN_CACHE_LIMIT"

操作步骤：

1. 在环境下配置环境变量，示例如下：

   ```
   export ASCEND_GLOBAL_LOG_LEVEL=1
   export HCCL_RDMA_TC=0
   export HCCL_RDMA_SL=0
   export ACLNN_CACHE_LIMIT=4096
   ```

   环境变量根据用户实际需要配置。

2. 执行Ascend PyTorch Profiler接口采集。

3. 查看结果数据。

   - 当experimental_config参数的export_type配置为torch_npu.profiler.ExportType.Text时，以上步骤配置的环境变量信息将保存在{worker_name}_{时间戳}_ascend_pt目录下的profiler_metadata.json文件中以及ascend_pytorch_profiler_{Rank_ID}.db文件下的META_DATA表中。
   - 当experimental_config参数的export_type配置为torch_npu.profiler.ExportType.Db时，在ascend_pytorch_profiler_{Rank_ID}.db文件下的META_DATA表写入环境变量信息。



#### （可选）以自定义字符串键和字符串值的形式标记性能数据采集过程

- 示例一

  ```
  with torch_npu.profiler.profile(...)  as prof:
      prof.add_metadata(key, value)
  ```

- 示例二

  ```
  with torch_npu.profiler._KinetoProfile(...)  as prof:
      prof.add_metadata_json(key, value)
  ```

add_metadata和add_metadata_json可以配置在torch_npu.profiler.profile和torch_npu.profiler._KinetoProfile下，须添加在profiler初始化后，finalize之前，即性能数据采集过程的代码中。

展开

| 类、函数名        | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| add_metadata      | 添加字符串标记，取值：*key*：字符串键。*value*：字符串值。示例：`prof.add_metadata("test_key1", "test_value1") ` |
| add_metadata_json | 添加json格式字符串标记，取值：*key*：字符串键。*value*：字符串值，json格式。示例：`prof.add_metadata_json("test_key2", json.dumps({"key1": test_value1, "key2": test_value2})) ` |

调用此接口传入的metadata数据写入到Ascend PyTorch Profiler接口的采集结果根目录下的profiler_metadata.json文件中。



#### （可选）显存可视化

本功能实现在模型训练过程中训练进程占用存储空间时，对所占用的数据进行分类并可视化展示。主要通过[export_memory_timeline](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_p16433135874116)导出可视化文件memory_timeline.html。输出html文件需要先在Python环境中安装matplotlib，且将对应的torch_npu.profiler.profile参数设置为True，另外使用该功能会在当前目录下生成后缀为ascend_pt数据文件。操作示例如下：

```
import torch
import torch_npu


def trace_handler(prof: torch_npu.profiler.profile):
    prof.export_memory_timeline(output_path="./memory_timeline.html", device="npu:0")

with torch_npu.profiler.profile(
    activities=[
        torch_npu.profiler.ProfilerActivity.CPU,
        torch_npu.profiler.ProfilerActivity.NPU
    ],
    schedule=torch_npu.profiler.schedule(wait=0, warmup=0, active=4, repeat=1, skip_first=0),
    on_trace_ready=trace_handler,
    record_shapes=True,           # 设为True
    profile_memory=True,          # 设为True
    with_stack=True,              # with_stack或者with_modules其中一个设为True
    with_modules=True
) as prof:
    for _ in range(steps):
        ...
        prof.step()
```

执行采集并导出memory_timeline.html后，可视化效果如下：

**图2** memory_timeline
![img](https://www.hiascend.com/doc_center/source/zh/mindstudio/830/T&ITools/Profiling/figure/zh-cn_image_0000002502558824.png)

- Time(ms)：为横坐标，表示tensor类型对内存的占用时间，单位ms。
- Memory(GB)：为纵坐标，表示tensor类型占用的内存大小，单位GB。
- Max memory allocated：最大内存分配总额，单位GB。
- Max memory reserved：最大内存预留总额，单位GB。
- PARAMETER：模型参数、模型权重。
- OPTIMIZER_STATE：优化器状态，例如Adam优化器会记录模型训练过程中的一些状态。
- INPUT：输入数据。
- TEMPORARY：临时占用，这里被定义为单个算子下申请后又被释放，通常是一些保存中间值的tensor。
- ACTIVATION：前向计算中得到的激活值。
- GRADIENT：梯度值。
- AUTOGRAD_DETAIL：反向计算过程中产生的内存占用。
- UNKNOWN：未知类型。



#### （可选）创建Profiler子线程采集

在推理场景下，单进程多线程调用torch算子的用法较为常见。在这种情况下，由于Profiler无法感知用户自行创建的子线程，因此Profiler也无法采集这些子线程下发的torch算子等框架侧数据。那么这里在用户创建的子线程中调用torch_npu.profiler.profile.enable_profiler_in_child_thread和torch_npu.profiler.profile.disable_profiler_in_child_thread接口来注册Profiler采集回调函数并对子线程下发的torch算子等框架侧数据进行采集。

示例如下：

```
import threading
import torch
import torch_npu

# 推理模型定义


def infer(device, child_thread):
    torch.npu.set_device(device)

    if child_thread:
        # 开始采集子线程的torch算子等框架侧数据
        torch_npu.profiler.profile.enable_profiler_in_child_thread(with_modules=True)

    for _ in range(5):
        outputs = model(input_data)

    if child_thread:
        # 停止采集子线程的torch算子等框架侧数据
        torch_npu.profiler.profile.disable_profiler_in_child_thread()


if __name__ == "__main__":
    experimental_config = torch_npu.profiler._ExperimentalConfig(
        aic_metrics=torch_npu.profiler.AiCMetrics.PipeUtilization,
        profiler_level=torch_npu.profiler.ProfilerLevel.Level1
    )

    prof = torch_npu.profiler.profile(
        activities=[torch_npu.profiler.ProfilerActivity.CPU, torch_npu.profiler.ProfilerActivity.NPU],
        on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result"),
        record_shapes=True,
        profile_memory=True,
        with_stack=False,
        with_flops=False,
        with_modules=True,
        experimental_config=experimental_config)

    prof.start()

    threads = []
    for i in range(1, 3):
        # 创建2个子线程，分别在device1与device2上进行推理任务
        t = threading.Thread(target=infer, args=(i, True))
        t.start()
        threads.append(t)

    # 主线程在device0上运行推理任务，由Profiler正常采集，非enable_profiler_in_child_thread接口采集
    infer(0, False)

    for t in threads:
        t.join()

    prof.stop()
```

完成子线程采集后，生成的子线程性能数据如下：

**图3** 子线程性能数据
![img](https://www.hiascend.com/doc_center/source/zh/mindstudio/830/T&ITools/Profiling/figure/zh-cn_image_0000002534398661.png)

上图中Thread 455385为主线程，Profiler可正常采集，本功能场景无需关注。另外两个线程中aten前缀的timeline即为本功能采集的torch算子数据。



#### Ascend PyTorch Profiler接口说明

展开

| 参数名称            | 说明                                                         | 是否必选 |
| ------------------- | ------------------------------------------------------------ | -------- |
| activities          | CPU、NPU事件采集列表，Enum类型。取值为：torch_npu.profiler.ProfilerActivity.CPU：框架侧数据采集的开关。torch_npu.profiler.ProfilerActivity.NPU：CANN软件栈及NPU数据采集的开关。默认情况下两个开关同时开启。 | 否       |
| schedule            | 设置不同step的行为，Callable类型，由[schedule类](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_table278012196491)控制。默认不执行任何操作。torch_npu.profiler._KinetoProfile不支持该参数。 | 否       |
| on_trace_ready      | 采集结束时自动执行操作，Callable类型。当前支持执行[tensorboard_trace_handler函数](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_p155624313516)操作。当采集的数据量过大时，在当前环境下不适合直接解析性能数据，或者采集过程中中断了训练/在线推理进程，只采集了部分性能数据，可以采用[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)。默认不执行任何操作。torch_npu.profiler._KinetoProfile不支持该参数。说明对于使用共享存储的多卡大集群场景，直接使用on_trace_ready执行tensorboard_trace_handler函数的方式进行性能数据落盘，可能因多卡数据直接落盘到共享存储导致性能膨胀的问题。解决方式请参见[PyTorch多卡大集群场景如何避免性能数据直接落盘到共享存储时导致的性能膨胀问题](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0315.html#ZH-CN_TOPIC_0000002536038457)。 | 否       |
| record_shapes       | 算子的InputShapes和InputTypes，Bool类型。取值为：True：开启。False：关闭。默认值。开启torch_npu.profiler.ProfilerActivity.CPU时生效。 | 否       |
| profile_memory      | 算子的显存占用情况，Bool类型。取值为：True：开启。False：关闭。默认值。开启torch_npu.profiler.ProfilerActivity.CPU时，采集框架显存占用情况；torch_npu.profiler.ProfilerActivity.NPU时，采集CANN的显存占用。说明已知在安装有glibc<2.34的环境上采集memory数据，可能触发glibc的一个已知[Bug 19329](https://sourceware.org/bugzilla/show_bug.cgi?id=19329)，通过升级环境的glibc版本可解决此问题。 | 否       |
| with_stack          | 算子调用栈，Bool类型。包括框架层及CPU算子层的调用信息。取值为：True：开启。False：关闭。默认值。开启torch_npu.profiler.ProfilerActivity.CPU时生效。说明开启该配置后会引入额外的性能膨胀。 | 否       |
| with_modules        | modules层级的Python调用栈，即框架层的调用信息，Bool类型。取值为：True：开启。False：关闭。默认值。开启torch_npu.profiler.ProfilerActivity.CPU时生效。说明开启该配置后会引入额外的性能膨胀。 | 否       |
| with_flops          | 算子浮点操作，Bool类型（该参数暂不支持解析性能数据）。取值为：True：开启。False：关闭。默认值。开启torch_npu.profiler.ProfilerActivity.CPU时生效。 | 否       |
| experimental_config | 性能数据采集扩展。支持采集项和详细介绍请参见[experimental_config参数说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1548285513313)。 | 否       |
| use_cuda            | 昇腾环境不支持。开启采集cuda性能数据开关，Bool类型。取值为：True：开启。False：关闭。默认值。torch_npu.profiler._KinetoProfile不支持该参数。 | 否       |

展开

| 方法名                           | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| step                             | 划分不同迭代。torch_npu.profiler._KinetoProfile不支持该方法。 |
| export_chrome_trace              | 导出trace。在指定的.json文件里写入trace数据。Trace为Ascend PyTorch Profiler接口整合框架侧CANN软件栈及NPU数据后展示的各算子和接口的运行时间及关联关系。包含参数：*path*：trace文件（.json）路径。指定文件所在的路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。必选。多卡场景下需要将不同卡设置不同的文件名，示例代码：`pid = os.getpid() prof.export_chrome_trace(f'./chrome_trace_{pid}.json') ` |
| export_stacks                    | 导出堆栈信息到文件。包含参数：*path*：堆栈文件保存路径，需要配置文件名为“*.log”，可以指定路径，例如：/home/*.log，直接配置文件名时，文件保存在当前目录。路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。必选。metric：保存的芯片类型可选择CPU或NPU，配置为“self_cpu_time_total”或“self_npu_time_total”。必选。与export_chrome_trace方法在训练/在线推理脚本中的位置相同，示例如下：`export_stacks('result_dir/stack.log', metric='self_npu_time_total') `导出的结果文件可使用FlameGraph工具进行查看，操作方法如下：`git clone https://github.com/brendangregg/FlameGraph cd FlameGraph ./flamegraph.pl –title "NPU time" –countname "us." profiler.stacks > perf_viz.svg` |
| export_memory_timeline           | 从采集的数据中导出给定设备的内存事件信息，并导出时间线图。使用export_memory_timeline可以导出3个文件，每个文件都由output_path的后缀控制：对于与HTML兼容的绘图，使用后缀.html，内存时间线图将作为PNG文件嵌入到HTML文件中。对于由[timestamp, [sizes by category]]组成的绘图点，其中timestamp是时间戳，sizes是每个类别的内存使用量。内存时间线图将保存为.json文件或压缩的.json.gz，具体取决于后缀。对于原始内存信息，使用后缀raw.json.gz。每个原始内存事件将由(timestamp, action, numbytes, category)组成，其中action是[PREEXISTING, CREATE, INCREMENT_VERSION, DESTROY]其中之一，category是[PARAMETER，OPTIMIZER_STATE，INPUT，TEMPORARY，ACTIVATION，GRADIENT，AUTOGRAD_DETAIL，UNKNOWN]其中之一。参数：output_path：配置导出的结果文件，string类型，配置格式为：path = "$PATH/*.html"，$PATH为结果文件路径，*为结果文件名，路径或文件不存在时会自动创建。必选。device：指定需要导出的Device ID，string类型，配置格式为：device = "npu:*"，*为Device ID或Rank ID，须配置为采集数据中已有的Device ID或Rank ID，当前仅支持指定一个值。必选。配置示例：`export_memory_timeline(output_path="./memory_timeline.html", device="npu:0") `详细操作指导请参见[（可选）显存可视化](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section206001142152910)。 |
| start                            | 设置采集开始的位置。可参考如下样例，在需要采集性能数据的训练/在线推理代码前后添加start和stop：`prof = torch_npu.profiler.profile( on_trace_ready=torch_npu.profiler.tensorboard_trace_handler("./result")) for step in range(steps):    if step == 5:        prof.start()    train_one_step()    if step == 5:        prof.stop() ` |
| stop                             | 设置采集结束的位置，需要先执行start。                        |
| enable_profiler_in_child_thread  | 注册Profiler采集回调函数，采集用户子线程下发的torch算子等框架侧数据。该参数中可另外配置torch_npu.profiler.profile的其他参数（包括record_shapes、profile_memory、with_stack、with_flops、with_modules），作为Profiler子线程的采集配置。与torch_npu.profiler.profile.enable_profiler_in_child_thread配对使用。详细使用请参见[（可选）创建Profiler子线程采集](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1837159184120)。torch_npu.profiler._KinetoProfile不支持该方法。 |
| disable_profiler_in_child_thread | 注销Profiler采集回调函数。与torch_npu.profiler.profile.enable_profiler_in_child_thread配对使用。torch_npu.profiler._KinetoProfile不支持该方法。 |



展开

| 类、函数名                                   | 说明                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| torch_npu.profiler.schedule                  | 设置不同step的行为，默认不执行该操作。为了获取更稳定的性能数据，建议配置该类的具体参数，参数取值及详细用法请参见[torch_npu.profiler.schedule类参数说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5522846122614)。 |
| torch_npu.profiler.tensorboard_trace_handler | 导出性能数据。取值为：dir_name：采集到的性能数据的存放路径，string类型。路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。若配置tensorboard_trace_handler函数后未指定具体路径，性能数据默认落盘在当前目录；若代码中未使用on_trace_ready=torch_npu.profiler.tensorboard_trace_handler，那么落盘的性能数据为原始数据，需要使用[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)。可选。该函数优先级高于ASCEND_WORK_PATH，具体请参考《[环境变量参考](https://www.hiascend.com/document/detail/zh/canncommercial/850/maintenref/envvar/envref_07_0001.html)》。worker_name：用于区分唯一的工作线程，string类型，默认为{hostname}_{pid}。路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。可选。analyse_flag：性能数据自动解析开关，bool类型。取值True（开启自动解析，默认值）、False（关闭自动解析，采集完后的性能数据可以使用[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)）。可选。async_mode：控制是否开启异步解析（表示解析进程不会阻塞AI任务主流程），bool类型。取值True（开启异步解析）、False（关闭异步解析，即同步解析，默认值）。torch_npu.profiler._KinetoProfile不支持该函数。解析过程日志存放在{worker_name}_{时间戳}_ascend_pt/logs目录下。 |
| torch_npu.profiler.ProfilerAction            | Profiler状态，Enum类型。取值为：NONE：无任何行为。WARMUP：性能数据采集预热。RECORD：性能数据采集。RECORD_AND_SAVE：性能数据采集并保存。 |
| torch_npu.profiler._ExperimentalConfig       | 性能数据采集扩展，Enum类型。通过torch_npu.profiler.profile的experimental_config调用，详细介绍请参见[experimental_config参数说明](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section1548285513313)。 |
| torch_npu.profiler.supported_activities      | 查询当前支持采集的activities参数的CPU、NPU事件。             |
| torch_npu.profiler.supported_profiler_level  | 查询当前支持的experimental_config参数的profiler_level级别。  |
| torch_npu.profiler.supported_ai_core_metrics | 查询当前支持的experimental_config参数的AI Core性能指标采集项。 |
| torch_npu.profiler.supported_export_type     | 查询当前支持的torch_npu.profiler.ExportType的性能数据结果文件类型。 |



#### profiler_config.json文件说明

profiler_config.json文件内容如下，以默认配置为例：

```
{
    "activities": ["CPU", "NPU"],
    "prof_dir": "./",
    "analyse": false,
    "record_shapes": false,
    "profile_memory": false,
    "with_stack": false,
    "with_flops": false,
    "with_modules": false,
    "active": 1,
    "warmup": 0,
    "start_step": 0,
    "is_rank": false,
    "rank_list": [],
    "experimental_config": {
        "profiler_level": "Level0",
        "aic_metrics": "AiCoreNone",
        "l2_cache": false,
        "op_attr": false,
        "gc_detect_threshold": null,
        "data_simplification": true,
        "record_op_args": false,
        "export_type": ["text"],
        "mstx": false,
        "mstx_domain_include": [],
        "mstx_domain_exclude": [],
        "host_sys": [],
        "sys_io": false,
        "sys_interconnection": false
    }
}
```



展开

| 参数                | 说明                                                         | 是否必选 |
| ------------------- | ------------------------------------------------------------ | -------- |
| start_step          | 设置开始采集的step，默认值为0（即不采集），设置为-1时表示在保存配置之后的下个step启动采集，配置为正整数时表示执行到该step时启动采集。启动采集进程首先需要配置该参数为有效值。 | 是       |
| activities          | CPU、NPU事件采集列表。取值为：CPU：框架侧数据采集的开关。NPU：CANN软件栈及NPU数据采集的开关。默认情况下两个开关同时开启。 | 否       |
| prof_dir            | 采集到的性能数据的存放路径。默认路径为：./。路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。 | 否       |
| analyse             | 性能数据自动解析开关，取值：true：开启自动解析。false：关闭自动解析，即手动解析，采集完后的性能数据可以使用[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)，默认值。 | 否       |
| record_shapes       | 算子的InputShapes和InputTypes。取值为：true：开启。false：关闭，默认值。activities配置为CPU时生效。 | 否       |
| profile_memory      | 算子的显存占用情况。取值为：true：开启。false：关闭，默认值。activities开启CPU时，采集框架显存占用情况；activities开启NPU时，采集CANN的显存占用。说明已知在安装有glibc<2.34的环境上采集memory数据，可能触发glibc的一个已知[Bug 19329](https://sourceware.org/bugzilla/show_bug.cgi?id=19329)，通过升级环境的glibc版本可解决此问题。 | 否       |
| with_stack          | 算子调用栈。包括框架层及CPU算子层的调用信息。取值为：true：开启。false：关闭，默认值。activities配置为CPU时生效。 | 否       |
| with_flops          | 算子浮点操作，Bool类型（该参数暂不支持解析性能数据）。取值为：true：开启。false：关闭，默认值。activities配置为CPU时生效。 | 否       |
| with_modules        | modules层级的Python调用栈，即框架层的调用信息。取值为：true：开启。false：关闭，默认值。activities配置为CPU时生效。 | 否       |
| active              | 配置采集的迭代数，取值为正整数，默认值为1。                  | 否       |
| warmup              | 预热的step轮数，默认值为0，建议设置1轮预热。                 | 否       |
| is_rank             | 开启指定Rank采集功能。取值为：true：开启。false：关闭，默认值。开启后，dynamic_profile会识别rank_list参数中配置的Rank ID，根据配置的Rank ID识别环境中存在的对应Rank执行采集操作；若开启后rank_list配置为空则不采集性能数据。开启后，analyse自动解析不生效，需要使用[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)。 | 否       |
| rank_list           | 配置采集的Rank ID，取值为整数，默认值为空，表示不采集任何性能数据。须配置为环境中有效的Rank ID。可同时指定一个或多个Rank，配置示例："rank_list": [1,2,3]。 | 否       |
| async_mode          | 控制是否开启异步解析（表示解析进程不会阻塞AI任务主流程），bool类型。取值true（开启异步解析）、false（关闭异步解析，即同步解析，默认值）。 | 否       |
| experimental_config | 扩展参数，通过扩展配置性能分析工具常用的采集项。详见[experimental_config参数说明（dynamic_profile动态采集场景）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section2015623185118)。对于动态采集场景，该配置文件中配置的experimental_config的子参数选项取实际参数值即可，例如"aic_metrics": "PipeUtilization"。 | 否       |
| metadata            | 采集模型超参数（key）和配置信息（value）。保存数据到ascend_pytorch_profiler_{Rank_ID}.db中的META_DATA表以及{worker_name}_{时间戳}_ascend_pt目录下的profiler_metadata.json文件中。配置示例：`    "metadata": {        "distributed_args":{            "tp":2,            "pp":4,            "dp":8        }    } ` | 否       |



#### experimental_config参数说明（dynamic_profile动态采集场景）

experimental_config参数均为可选参数，支持扩展的采集项如下：

展开

| 参数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| profiler_level      | 采集的Level等级。取值如下：Level_none：不采集所有Level层级控制的数据，即关闭profiler_level。Level0：采集上层应用数据、底层NPU数据以及NPU上执行的算子信息。默认值。配置该参数时，仅采集部分数据，其中部分算子信息不采集，详细情况请参见[op_summary（算子详细信息）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0151.html#ZH-CN_TOPIC_0000002536038367)。Level1：在Level0的基础上多采集CANN层AscendCL数据和NPU上执行的AI Core性能指标信息、开启aic_metrics=torch_npu.profiler.AiCMetrics.PipeUtilization、生成通信算子的communication.json和communication_matrix.json以及api_statistic.csv文件。Level2：在Level1的基础上多采集CANN层Runtime数据以及AI CPU（data_preprocess.csv文件）数据。 |
| aic_metrics         | AI Core的性能指标采集项。取值如下：以下采集项的结果数据将在Kernel View呈现。以下采集项的结果数据含义可参见[op_summary（算子详细信息）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0151.html#ZH-CN_TOPIC_0000002536038367)，但具体采集结果请以实际情况为准。AiCoreNone：关闭AI Core的性能指标采集。PipeUtilization：计算单元和搬运单元耗时占比。ArithmeticUtilization：各种计算类指标占比统计。Memory：外部内存读写类指令占比。MemoryL0：内部L0内存读写类指令占比。ResourceConflictRatio：流水线队列类指令占比。MemoryUB：内部UB内存读写类指令占比。L2Cache：读写cache命中次数和缺失后重新分配次数。MemoryAccess：算子在核上访存的带宽数据量。当profiler_level设置为Level_none或Level0，默认值为AiCoreNone；当profiler_level设置为Level1或Level2，默认值为PipeUtilization。 |
| l2_cache            | 控制L2 Cache数据采集开关。取值true（开启）或false（关闭），默认关闭。该采集项在ASCEND_PROFILER_OUTPUT生成l2_cache.csv文件，结果字段介绍请参见[l2_cache（L2 Cache命中率）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0157.html#ZH-CN_TOPIC_0000002504198588)。 |
| op_attr             | 控制采集算子的属性信息开关，当前仅支持采集aclnn算子。取值true（开启）或false（关闭），默认关闭。Level_none时，该参数不生效。 |
| gc_detect_threshold | GC检测阈值。取值范围为大于等于0的数值，单位ms。当用户设置的阈值为数字时，表示开启GC检测，只采集超过阈值的GC事件。配置为0时表示采集所有的GC事件（可能造成采集数据量过大，请谨慎配置），推荐设置为1ms。默认为null，表示不开启GC检测功能。**GC**是Python进程对已经销毁的对象进行内存回收。该参数解析结果为在trace_view.json中生成[GC](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0205.html#ZH-CN_TOPIC_0000002504198614__zh-cn_topic_0000002534478419_fig136071315133614)层或在ascend_pytorch_profiler_{Rank_ID}.db中生成GC_RECORD表。 |
| data_simplification | 数据精简模式，开启后将在导出性能数据后删除多余数据，仅保留profiler_*.json文件、ASCEND_PROFILER_OUTPUT目录、PROF_XXX目录下的原始性能数据、FRAMEWORK目录和logs目录，以节省存储空间。取值true（开启）或false（关闭），默认开启。 |
| record_op_args      | 控制算子信息统计功能开关。取值true（开启）或false（关闭），默认关闭。开启后会在{worker_name}_{时间戳}_ascend_pt_op_args目录输出采集到算子信息文件。说明该参数在AOE工具执行PyTorch训练场景下调优时使用，且不建议与其他性能数据采集接口同时开启。详见《[AOE调优工具用户指南](https://www.hiascend.com/document/detail/zh/canncommercial/850/devaids/aoe/auxiliarydevtool_aoe_0001.html)》。 |
| export_type         | 设置导出的性能数据结果文件格式，List类型。取值：text：表示解析为.json和.csv格式的timeline和summary文件以及汇总所有性能数据的.db格式文件（ascend_pytorch_profiler_{Rank_ID}.db、analysis.db）。db：表示仅解析为汇总所有性能数据的.db格式文件（ascend_pytorch_profiler_{Rank_ID}.db、analysis.db），使用MindStudio Insight工具展示。仅支持on_trace_ready接口导出和[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)导出，需配套安装支持导出db格式的CANN Toolkit开发套件包和ops算子包。设置无效值或未配置均取默认值text。解析结果数据请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。 |
| mstx或msprof_tx     | 打点控制开关，通过开关开启自定义打点功能。取值true（开启）或false（关闭），默认关闭。该参数使用请参见[（可选）采集并解析mstx数据](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5940122172516)。原参数名msprof_tx改为mstx，新版本依旧兼容原参数名msprof_tx。 |
| mstx_domain_include | 输出需要的domain数据。调用torch_npu.npu.mstx系列打点接口，使用默认domain或指定domain进行打点时，可选择只输出本参数配置的domain数据。domain名称为用户调用torch_npu.npu.mstx系列接口传入的domain或默认domain（'default'），domain名称使用List类型输入。与mstx_domain_exclude参数互斥，若同时配置，则只有mstx_domain_include生效。须配置mstx=True。 |
| mstx_domain_exclude | 过滤不需要的domain数据。调用torch_npu.npu.mstx系列打点接口，使用默认domain或指定domain进行打点时，可选择不输出本参数配置的domain数据。domain名称为用户调用torch_npu.npu.mstx系列接口传入的domain或默认domain（'default'），domain名称使用List类型输入。与mstx_domain_include参数互斥，若同时配置，则只有mstx_domain_include生效。须配置mstx=True。 |
| host_sys            | Host侧系统数据采集开关，List类型。默认未配置，表示未开启Host侧系统数据采集。取值：cpu：进程级别的CPU利用率。mem：进程级别的内存利用率。disk：进程级别的磁盘I/O利用率。network：系统级别的网络I/O利用率。osrt：进程级别的syscall和pthreadcall。配置示例：host_sys : ["cpu", "disk"]。说明采集Host侧disk性能数据需要安装第三方开源工具iotop，采集osrt性能数据需要安装第三方开源工具perf和ltrace，其安装方法参见[安装perf、iotop、ltrace工具](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0210.html#ZH-CN_TOPIC_0000002504358452)。完成安装后须参见[配置用户权限](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0211.html#ZH-CN_TOPIC_0000002536038405)完成用户权限配置，且每次重新安装CANN软件包需要重新配置。使用开源工具ltrace采集osrt性能数据会导致CPU占用率过高，其与应用工程的pthread加解锁相关，会影响进程运行速度。x86_64架构的KylinV10SP1操作系统支持osrt参数， aarch64架构的KylinV10SP1操作系统下不支持osrt参数。虚拟化环境Euler2.9系统下不支持network参数。 |
| sys_io              | NIC、ROCE、MAC采集开关。取值true（开启）或false（关闭），默认关闭。 |
| sys_interconnection | 集合通信带宽数据（HCCS）、PCIe数据采集开关、片间传输带宽信息采集开关。取值true（开启）或false（关闭），默认关闭。 |



#### experimental_config参数说明

experimental_config参数均为可选参数，支持扩展的采集项如下：

展开

| 参数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| export_type         | 设置导出的性能数据结果文件格式，List类型。取值：torch_npu.profiler.ExportType.Text：表示解析为.json和.csv格式的timeline和summary文件以及汇总所有性能数据的.db格式文件（ascend_pytorch_profiler_{Rank_ID}.db、analysis.db）。torch_npu.profiler.ExportType.Db：表示仅解析为汇总所有性能数据的.db格式文件（ascend_pytorch_profiler_{Rank_ID}.db、analysis.db），使用MindStudio Insight工具展示。仅支持on_trace_ready接口导出和[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)导出，需配套安装支持导出db格式的CANN-Toolkit开发套件包和ops算子包。设置无效值或未配置均取默认值torch_npu.profiler.ExportType.Text。解析结果数据请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。 |
| profiler_level      | 采集的Level等级，Enum类型。取值如下：torch_npu.profiler.ProfilerLevel.Level_none：不采集所有Level层级控制的数据，即关闭profiler_level。torch_npu.profiler.ProfilerLevel.Level0：采集上层应用数据、底层NPU数据以及NPU上执行的算子信息。默认值。配置该参数时，仅采集部分数据，其中部分算子信息不采集，详细情况请参见[op_summary（算子详细信息）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0151.html#ZH-CN_TOPIC_0000002536038367)中有关task_time为l0时的说明。torch_npu.profiler.ProfilerLevel.Level1：在Level0的基础上多采集CANN层AscendCL数据和NPU上执行的AI Core性能指标信息、开启aic_metrics=torch_npu.profiler.AiCMetrics.PipeUtilization、生成通信算子的communication.json和communication_matrix.json以及api_statistic.csv文件。torch_npu.profiler.ProfilerLevel.Level2：在Level1的基础上多采集CANN层Runtime数据以及AI CPU（data_preprocess.csv文件）数据。 |
| mstx或msprof_tx     | 打点控制开关，通过开关开启自定义打点功能，bool类型。取值True（开启）或False（关闭），默认关闭。该参数使用请参见[（可选）采集并解析mstx数据](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0121.html?framework=pytorch#ZH-CN_TOPIC_0000002504198570__zh-cn_topic_0000002534478481_section5940122172516)。原参数名msprof_tx改为mstx，新版本依旧兼容原参数名msprof_tx。 |
| mstx_domain_include | 输出需要的domain数据。调用[torch_npu.npu.mstx](https://www.hiascend.com/document/detail/zh/Pytorch/720/apiref/torchnpuCustomsapi/context/torch_npu-npu-mstx.md)系列打点接口，使用默认domain或指定domain进行打点时，可选择只输出本参数配置的domain数据。domain名称为用户调用torch_npu.npu.mstx系列接口传入的domain或默认domain（'default'），domain名称使用List类型输入。与mstx_domain_exclude参数互斥，若同时配置，则只有mstx_domain_include生效。须配置mstx=True。 |
| mstx_domain_exclude | 过滤不需要的domain数据。调用[torch_npu.npu.mstx](https://www.hiascend.com/document/detail/zh/Pytorch/720/apiref/torchnpuCustomsapi/context/torch_npu-npu-mstx.md)系列打点接口，使用默认domain或指定domain进行打点时，可选择不输出本参数配置的domain数据。domain名称为用户调用torch_npu.npu.mstx系列接口传入的domain或默认domain（'default'），domain名称使用List类型输入。与mstx_domain_include参数互斥，若同时配置，则只有mstx_domain_include生效。须配置mstx=True。 |
| aic_metrics         | AI Core的性能指标采集项。取值如下：以下采集项的结果数据将在Kernel View呈现。以下采集项的结果数据含义可参见[op_summary（算子详细信息）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0151.html#ZH-CN_TOPIC_0000002536038367)，但具体采集结果请以实际情况为准。AiCoreNone：关闭AI Core的性能指标采集。PipeUtilization：计算单元和搬运单元耗时占比。ArithmeticUtilization：各种计算类指标占比统计。Memory：外部内存读写类指令占比。MemoryL0：内部L0内存读写类指令占比。ResourceConflictRatio：流水线队列类指令占比。MemoryUB：内部UB内存读写类指令占比。L2Cache：读写cache命中次数和缺失后重新分配次数。MemoryAccess：算子在核上访存的带宽数据量。当profiler_level设置为torch_npu.profiler.ProfilerLevel.Level_none或torch_npu.profiler.ProfilerLevel.Level0时，默认值为AiCoreNone；当profiler_level设置为torch_npu.profiler.ProfilerLevel.Level1或torch_npu.profiler.ProfilerLevel.Level2时，默认值为PipeUtilization。 |
| l2_cache            | 控制L2 Cache数据采集开关，bool类型。取值True（开启）或False（关闭），默认关闭。该采集项在ASCEND_PROFILER_OUTPUT生成l2_cache.csv文件，结果字段介绍请参见[l2_cache（L2 Cache命中率）](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0157.html#ZH-CN_TOPIC_0000002504198588)。 |
| op_attr             | 控制采集算子的属性信息开关，当前仅支持采集aclnn算子，bool类型。取值True（开启）或False（关闭），默认关闭。该参数采集的性能数据仅db格式文件生效；torch_npu.profiler.ProfilerLevel.None时，该参数不生效。 |
| data_simplification | 数据精简模式，开启后将在导出性能数据后删除多余数据，仅保留profiler_*.json文件、ASCEND_PROFILER_OUTPUT目录、PROF_XXX目录下的原始性能数据、FRAMEWORK目录和logs目录，以节省存储空间。取值true（开启）或false（关闭），默认开启。 |
| record_op_args      | 控制算子信息统计功能开关，bool类型。取值True（开启）或False（关闭），默认关闭。开启后会在{worker_name}_{时间戳}_ascend_pt_op_args目录输出采集到算子信息文件。说明该参数在AOE工具执行PyTorch训练场景下调优时使用，且不建议与其他性能数据采集接口同时开启。详见《[AOE调优工具用户指南](https://www.hiascend.com/document/detail/zh/canncommercial/850/devaids/aoe/auxiliarydevtool_aoe_0001.html)》。 |
| gc_detect_threshold | GC检测阈值，float类型。取值范围为大于等于0的数值，单位ms。当用户设置的阈值为数字时，表示开启GC检测，只采集超过阈值的GC事件。配置为0时表示采集所有的GC事件（可能造成采集数据量过大，请谨慎配置），推荐设置为1ms。默认为None，表示不开启GC检测功能。**GC**是Python进程对已经销毁的对象进行内存回收。该参数解析结果为在trace_view.json中生成[GC](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0205.html#ZH-CN_TOPIC_0000002504198614__zh-cn_topic_0000002534478419_fig136071315133614)层或在ascend_pytorch_profiler_{Rank_ID}.db中生成GC_RECORD表。 |
| host_sys            | Host侧系统数据采集开关，List类型。默认未配置，表示未开启Host侧系统数据采集。取值：torch_npu.profiler.HostSystem.CPU：进程级别的CPU利用率。torch_npu.profiler.HostSystem.MEM：进程级别的内存利用率。torch_npu.profiler.HostSystem.DISK：进程级别的磁盘I/O利用率。torch_npu.profiler.HostSystem.NETWORK：系统级别的网络I/O利用率。torch_npu.profiler.HostSystem.OSRT：进程级别的syscall和pthreadcall。说明采集Host侧disk性能数据需要安装第三方开源工具iotop，采集osrt性能数据需要安装第三方开源工具perf和ltrace，其安装方法参见[安装perf、iotop、ltrace工具](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0210.html#ZH-CN_TOPIC_0000002504358452)。完成安装后须参见[配置用户权限](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0211.html#ZH-CN_TOPIC_0000002536038405)完成用户权限配置，且每次重新安装CANN软件包需要重新配置。使用开源工具ltrace采集osrt性能数据会导致CPU占用率过高，其与应用工程的pthread加解锁相关，会影响进程运行速度。x86_64架构的KylinV10SP1操作系统支持torch_npu.profiler.HostSystem.OSRT参数， aarch64架构的KylinV10SP1操作系统下不支持torch_npu.profiler.HostSystem.OSRT参数。虚拟化环境Euler2.9系统下不支持torch_npu.profiler.HostSystem.NETWORK参数。 |
| sys_io              | NIC、ROCE、MAC采集开关，bool类型。取值True（开启）或False（关闭），默认关闭。 |
| sys_interconnection | 集合通信带宽数据（HCCS）、PCIe数据采集开关、片间传输带宽信息采集开关，bool类型。取值True（开启）或False（关闭），默认关闭。 |



#### torch_npu.profiler.schedule类参数说明

torch_npu.profiler.schedule类用于在采集进程中设置在不同step时的采集行为。接口原型为：

```
torch_npu.profiler.schedule(wait, active, warmup = 0, repeat = 0, skip_first = 0)
```

展开

| 参数                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| wait                                                         | 每次重复执行采集跳过的step轮数，int类型。必选。              |
| active                                                       | 采集的step轮数，int类型。必选。                              |
| warmup                                                       | 预热的step轮数，int类型。默认值为0。建议设置1轮预热。可选。  |
| repeat                                                       | 重复执行wait + warmup + active的次数，int类型。取值范围为大于等于0的整数，默认值为0。可选。说明当使用集群分析工具或MindStudio Insight查看时，建议配置repeat = 1（表示执行1次，仅生成一份性能数据），因为：repeat > 1会在同一目录下生成多份性能数据，则需要手动将采集的性能数据文件夹分为repeat等份，放到不同文件夹下重新解析，分类方式按照文件夹名称中的时间戳先后。repeat = 0时，重复执行的具体次数就由总训练步数决定，例如总训练步数为100，wait + active + warmup = 10，skip_first = 10，则repeat = ( 100 - 10 ) / 10 = 9，表示重复执行9次，生成9份性能数据。 |
| skip_first                                                   | 采集前先跳过的step轮数，int类型。默认值为0。动态Shape场景建议跳过前10轮保证性能数据稳定；对于其他场景，可以根据实际情况自行配置。可选。 |
| 注：建议根据此公式配置schedule：step总数 > = skip_first + ( wait + warmup + active ) * repeat |                                                              |

torch_npu.profiler.schedule类、step和on_trace_ready函数使用关系示意图如下：

**图4** torch_npu.profiler.schedule类、step和on_trace_ready函数使用关系示意图
![img](https://www.hiascend.com/doc_center/source/zh/mindstudio/830/T&ITools/Profiling/figure/zh-cn_image_0000002502558842.png)

设置示例代码如下：

```
with torch_npu.profiler.profile(
    activities=[
        torch_npu.profiler.ProfilerActivity.CPU,
        torch_npu.profiler.ProfilerActivity.NPU,
    ],
    schedule=torch_npu.profiler.schedule(
        wait=1,                        # 等待阶段，跳过1个step
        warmup=1,                      # 预热阶段，跳过1个step
        active=2,                      # 记录2个step的活动数据，并在之后调用on_trace_ready
        repeat=2,                      # 循环wait+warmup+active过程2遍
        skip_first=1                   # 跳过1个step
    ),
    on_trace_ready=torch_npu.profiler.tensorboard_trace_handler('./result')
    ) as prof:
        for _ in range(9):
            train_one_step()
            prof.step()                # 通知profiler完成一个step
```



#### dynamic_profile动态采集维测日志介绍

dynamic_profile动态采集在profiler_config_path目录下自动记录dynamic_profile的维测日志，生成日志目录结构示例如下：

```
profiler_config_path/
├── log
│    ├── dp_ubuntu_xxxxxx_rank_*.log
│    ├── dp_ubuntu_xxxxxx_rank_*.log.1
│    ├── monitor_dp_ubuntu_xxxxxx_rank_*.log
│    ├── monitor_dp_ubuntu_xxxxxx_rank_*.log.1
├── profiler_config.json
└── shm
```

- dp_ubuntu_xxxxxx.log：dynamic_profile动态采集的执行日志，记录动态采集执行过程中的所有动作（INFO）、警告（WARNING）和错误（ERROR）。文件命名格式：dp_{操作系统}_{AI任务进程ID}_{Rank_ID}.log。

  AI任务启动时每个Rank会开启一个AI任务进程，dynamic_profile根据每个AI任务进程ID生成各个AI任务进程下的日志文件。

- dp_ubuntu_xxxxxx.log.1：日志老化备份文件，dp_ubuntu_xxxxxx.log文件的存储上限为200K，达到上限后将时间最早的日志记录转移到dp_ubuntu_xxxxxx.log.1中，dp_ubuntu_xxxxxx.log.1文件存储上限同样为200K，达到上限后则将最早的日志记录老化删除。

- monitor_dp_ubuntu_xxxxxx.log：profiler_config.json文件修改日志，开启dynamic_profile动态采集后，实时记录profiler_config.json文件的每次修改时间、修改是否生效以及dynamic_profile进程的结束，示例如下：

  ```
  2024-08-21 15:51:46,392 [INFO] [2127856] _dynamic_profiler_monitor.py: Dynamic profiler process load json success
  2024-08-21 15:51:58,406 [INFO] [2127856] _dynamic_profiler_monitor.py: Dynamic profiler process load json success
  2024-08-21 15:58:16,795 [INFO] [2127856] _dynamic_profiler_monitor.py: Dynamic profiler process done
  ```

  文件命名格式：monitor_dp_{操作系统}_{monitor进程ID}_{Rank_ID}.log。

- monitor_dp_ubuntu_xxxxxx.log.1：日志老化备份文件，monitor_dp_ubuntu_xxxxxx.log文件的存储上限为200K，达到上限后将时间最早的日志记录转移到monitor_dp_ubuntu_xxxxxx.log.1中，monitor_dp_ubuntu_xxxxxx.log.1文件存储上限同样为200K，达到上限后则将最早的日志记录老化删除。

- shm目录：为了适配Python3.7，dynamic_profile会在py37环境下生成shm目录，目录下生成一个二进制文件（DynamicProfileNpuShm+时间）映射共享内存，程序正常结束后会自动清理，当使用pkill终止程序时，由于是异常终止，程序无法释放资源，需要用户手动清理此文件，否则短时间内（<1h）下次使用同一配置路径启动dynamic_profile，则会导致dynamic_profile异常。对于Python3.8及以上版本，二进制文件（DynamicProfileNpuShm+时间）存放在/dev/shm目录下，当使用pkill终止程序时，同样需要手动清理此文件。

# 离线解析

当使用Ascend PyTorch Profiler接口采集的性能数据较大时，若在当前环境直接使用on_trace_ready接口进行自动解析，则可能导致资源占用过大出现卡顿，那么可以取消on_trace_ready接口，并通过环境变量ASCEND_WORK_PATH设置落盘目录（例如：export ASCEND_WORK_PATH=xx/xx），在采集完成性能数据后，使用如下方式进行离线解析：

1. 创建*{file_name}*.py文件，*{file_name}*自定义，并编辑如下代码。

   ```
   from torch_npu.profiler.profiler import analyse
   
   if __name__ == "__main__":
       analyse(profiler_path="./result_data", max_process_number=max_process_number, export_type=export_type)
   ```

   展开

   | 参数               | 描述                                                         | **可选/必选** |
   | ------------------ | ------------------------------------------------------------ | ------------- |
   | profiler_path      | PyTorch性能数据路径。路径格式仅支持由字母、数字、下划线和连字符组成的字符串，不支持软链接。指定的目录下保存PyTorch性能数据目录{worker_name}_{时间戳}_ascend_pt。 | 必选          |
   | max_process_number | 离线解析最大进程数。取值范围为1~CPU核数，默认为CPU核数的一半。若设置超过该环境的CPU核数，则自动取CPU核数；若设置为非法值，则取默认值CPU核数的一半。 | 可选          |
   | export_type        | 设置导出的性能数据结果文件格式，List类型。取值：text：表示解析为.json和.csv格式的timeline和summary文件以及汇总所有性能数据的.db格式文件（ascend_pytorch_profiler_{Rank_ID}.db、analysis.db）。db：表示仅解析为汇总所有性能数据的.db格式文件（ascend_pytorch_profiler_{Rank_ID}.db、analysis.db），使用MindStudio Insight工具展示。仅支持on_trace_ready接口导出和[离线解析](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0122.html#ZH-CN_TOPIC_0000002504358406)导出，需配套安装支持导出db格式的CANN Toolkit开发套件包和ops算子包。设置无效值或未配置时，则读取profiler_info.json中的export_type字段，确定导出格式。解析结果数据请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。 | 可选          |

   说明

   - 离线解析接口支持多性能数据目录并行解析，当性能数据量较大且数据目录较多的情况下，可能因环境内存不足导致解析失败，此时可以通过自定义最大进程数（max_process_number）来控制资源的占用。
   - 解析过程日志存放在{worker_name}_{时间戳}_ascend_pt/logs目录下。

2. 保存文件后执行如下命令解析性能数据：

   ```
   python3 {file_name}.py
   ```

3. 查看性能数据结果文件和性能数据分析。

   性能数据结果文件详细介绍请参见[MindSpore&PyTorch框架性能数据文件参考](https://www.hiascend.com/document/detail/zh/mindstudio/830/T&ITools/Profiling/atlasprofiling_16_0203.html#ZH-CN_TOPIC_0000002536038401)。

   请参见《[MindStudio Insight工具用户指南](https://www.hiascend.com/document/detail/zh/mindstudio/830/GUI_baseddevelopmenttool/msascendinsightug/Insight_userguide_0002.html)》将解析后的性能数据文件进行可视化展示和分析。

   可以使用[性能分析工具（msprof-analyze）](https://gitcode.com/Ascend/mstt/tree/master/profiler/msprof_analyze)辅助分析性能数据。
