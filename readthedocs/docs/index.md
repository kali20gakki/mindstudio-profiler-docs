---
hide:
  - navigation
  - toc
---

# Welcome to

<figure markdown="span">
  ![](./assets/logos/mindstudio.jpg){ align="center" alt="MindStudio" class="home-logo" width="112" }
</figure>

<p class="home-wordmark">MindStudio Profiling Tools</p>

<p class="home-tagline">
<strong>面向 Ascend 场景的性能工具全集，涵盖性能数据采集、监控、分析等各类工具。</strong>
</p>

<p class="home-actions">
  <a class="md-button md-button--primary" href="./overview/toolchain/">查看工具全景</a>
  <a class="md-button" href="./msprof/">开始使用工具</a>
</p>

在这里聚合了 Ascend 训练与推理场景下常用的 Profiling 工具文档，覆盖离线采集、框架侧 Profiling、接口埋点、在线监控和结果分析几个环节。
## Tool Roles

MSTT 当前文档按工具职责组织为 5 类核心能力：

- **msprof**：面向 Profiling 数据采集、导出与基础解析，是离线性能数据获取入口。
- **ascend-pytorch-profiler**：面向 `torch_npu`/PyTorch 使用场景，负责框架侧 Profiling 配置、采集与结果查看。
- **mspti**：提供 Profiling 相关的 C API 与 Python API，适合做自定义埋点、接口集成和能力接入。
- **msmonitor**：提供在线监控、动态采集和 Monitor API，适合运行中观测与问题发现。
- **msprof-analyze**：围绕已有 Profiling 数据做专家分析、性能对比、集群分析和调优定位。

## Start Here

<style>
.grid.cards .md-typeset hr {
    width: 100%;
    margin: 0.5rem 0 1rem 0;
    border: none;
    height: 1px;
    background: linear-gradient(90deg,
        rgba(0,0,0,0.08) 0%,
        rgba(0,0,0,0.2) 40%,
        rgba(0,0,0,0.2) 60%,
        rgba(0,0,0,0.08) 100%);
}

.grid.cards {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
}

.grid.cards .card {
    height: 100%;
    display: flex;
    flex-direction: column;
}

.grid.cards .card hr {
    width: 100%;
}

[data-md-color-scheme="slate"] .grid.cards .md-typeset hr {
    background: linear-gradient(90deg,
        rgba(255,255,255,0.08) 0%,
        rgba(255,255,255,0.2) 40%,
        rgba(255,255,255,0.2) 60%,
        rgba(255,255,255,0.08) 100%);
}
</style>

<div class="grid cards" markdown>

-   **Overview**

    ---

    先了解工具链全景、性能数据类型和调优入口。
    [打开 Overview](./overview/toolchain.md)

-   **msprof**

    ---

    面向离线采集、数据导出和基础结果获取。
    [打开 msprof](./msprof/)

-   **ascend-pytorch-profiler**

    ---

    面向 PyTorch / `torch_npu` 场景的框架侧 Profiling。
    [打开 ascend-pytorch-profiler](./torch_npu_profiler/)

-   **mspti**

    ---

    面向 Profiling 能力接入、埋点和 API 集成。
    [打开 mspti](./mspti/)

-   **msmonitor**

    ---

    面向在线监控、动态采集和运行中观测。
    [打开 msmonitor](./msmonitor/)

-   **msprof-analyze**

    ---

    面向结果分析、性能对比和调优定位。
    [打开 msprof-analyze](./msprof-analyze/)

</div>
