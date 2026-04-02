---
hide:
  - navigation
  - toc
---


# 性能调优工具链全景

<style>
.ai-arch-container {
    --ai-accent-blue: #3b82f6;
    --ai-text-dark: #1e293b;
    --ai-bg-section: #f8fafc;
    --ai-transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px 20px 40px 20px;
    box-sizing: border-box;
}

.ai-arch-container * {
    box-sizing: border-box;
}

.ai-arch-container .arch-wrapper {
    position: relative;
    display: flex;
    flex-direction: row;
    gap: 24px;
    padding: 40px;
    background: #ffffff;
    border-radius: 24px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08), 0 1px 3px rgba(0, 0, 0, 0.02);
    border: 1px solid rgba(255, 255, 255, 0.8);
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
}

.ai-arch-container .main-layers {
    display: flex;
    flex-direction: column;
    gap: 20px;
    flex: 1;
}

.ai-arch-container .layer-section {
    display: flex;
    background: var(--ai-bg-section);
    border-radius: 14px;
    overflow: hidden;
    border: 1px solid rgba(226, 232, 240, 0.7);
    transition: var(--ai-transition);
}

.ai-arch-container .layer-title {
    width: 48px;
    writing-mode: vertical-lr;
    text-orientation: mixed;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #e2e8f0;
    color: #475569;
    font-size: 13px;
    font-weight: 600;
    letter-spacing: 2px;
    border-right: 1px solid rgba(203, 213, 225, 0.5);
}

.ai-arch-container .layer-content {
    flex: 1;
    padding: 20px;
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 16px;
}

.ai-arch-container .layer-content-3 {
    grid-template-columns: repeat(3, 1fr);
}

.ai-arch-container .module {
    position: relative;
    background: #ffffff;
    padding: 14px 20px;
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 14px;
    font-weight: 700;
    cursor: pointer;
    transition: var(--ai-transition);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.04);
    color: var(--ai-text-dark);
    border: 1px solid rgba(226, 232, 240, 0.8);
    text-align: center;
}

.ai-arch-container .module:hover {
    background: var(--ai-accent-blue);
    color: #ffffff !important;
    transform: translateY(-3px);
    box-shadow: 0 8px 20px rgba(59, 130, 246, 0.25);
    border-color: var(--ai-accent-blue);
}

.ai-arch-container .tool-card {
    width: 100px;
    background: var(--ai-bg-section);
    border: 1px solid rgba(226, 232, 240, 0.8);
    border-radius: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 20px 10px;
    text-align: center;
    font-size: 14px;
    font-weight: 700;
    transition: var(--ai-transition);
    cursor: pointer;
    color: var(--ai-text-dark);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.02);
}

.ai-arch-container .tool-card:hover {
    background: #1e293b;
    color: #ffffff;
    transform: scale(1.05);
}

.ai-arch-container #dynamic-preview {
    position: absolute;
    width: 260px;
    background: #ffffff;
    border-radius: 14px;
    padding: 18px;
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
    pointer-events: none;
    opacity: 0;
    transform: translateX(10px);
    transition: opacity 0.3s, transform 0.3s;
    z-index: 2000;
    border: 1px solid rgba(0, 0, 0, 0.06);
}

.ai-arch-container #dynamic-preview.active {
    opacity: 1;
    transform: translateX(0);
}

.ai-arch-container .preview-h {
    font-weight: 800;
    color: var(--ai-accent-blue);
    margin-bottom: 8px;
    font-size: 15px;
}

.ai-arch-container .preview-p {
    font-size: 13px;
    color: #64748b;
    line-height: 1.6;
    margin: 0;
}

.ai-arch-container .span-2 { grid-column: span 2; }
.ai-arch-container .flex-center { justify-content: center; }
.ai-arch-container .justify-content-around { justify-content: space-around; }

@media (max-width: 850px) {
    .ai-arch-container .arch-wrapper {
        flex-direction: column;
        align-items: stretch;
    }

    .ai-arch-container .tool-card {
        width: 100%;
        flex-direction: row;
        padding: 15px;
        gap: 15px;
    }

    .ai-arch-container #dynamic-preview {
        display: none !important;
    }
}

@media (max-width: 550px) {
    .ai-arch-container {
        padding: 20px 10px;
    }

    .ai-arch-container .layer-content,
    .ai-arch-container .layer-content-3 {
        grid-template-columns: 1fr;
        padding: 12px;
    }

    .ai-arch-container .span-2 {
        grid-column: span 1;
    }

    .ai-arch-container .layer-title {
        width: 36px;
        font-size: 11px;
    }

    .ai-arch-container .module {
        font-size: 13px;
        padding: 12px;
    }
}
</style>

<div class="ai-arch-container">
    <div class="arch-wrapper" id="container">
        <div class="main-layers">
            <div class="layer-section">
                <div class="layer-title">分析工具</div>
                <div class="layer-content">
                    <div class="module" data-title="msprof-analyze" data-desc="面向 AI 训练与推理场景的性能分析工具，具备性能比对、集群分析、专家建议等功能。">
                        msprof-analyze
                    </div>
                    <div class="module" data-title="msAgent" data-desc="专注于性能问题定位的Agent。">
                        msAgent
                    </div>
                </div>
            </div>

            <div class="layer-section">
                <div class="layer-title">性能监控</div>
                <div class="layer-content">
                    <div class="module span-2 flex-center" data-title="msMonitor" data-desc="面向昇腾集群场景的在线性能监控与动态采集工具。">
                        MindStudio Monitor
                    </div>
                </div>
            </div>

            <div class="layer-section">
                <div class="layer-title">AI框架</div>
                <div class="layer-content justify-content-around">
                    <div class="module" data-title="PyTorch Profiler" data-desc="基于PyTorch框架开发的昇腾软硬件性能调优工具。">PyTorch Profiler</div>
                    <div class="module" data-title="MindSpore Profiler" data-desc="基于昇思MindSpore框架开发的昇腾软硬件性能调优工具。">MindSpore Profiler</div>
                </div>
            </div>

            <div class="layer-section">
                <div class="layer-title">基础能力</div>
                <div class="layer-content layer-content-3">
                    <div class="module" data-title="msProf" data-desc="昇腾Profiling命令行工具，支持采集与解析 CANN 平台及昇腾 AI 处理器的软硬件性能数据。">msProf</div>
                    <div class="module" data-title="MSTX" data-desc="MindStudio Tools Extension Library，支持用户自定义打点。">MSTX</div>
                    <div class="module" data-title="MSPTI" data-desc="MindStudio Profiler Tools Interface，面向昇腾设备的 Profiling API 集合。">MSPTI</div>
                </div>
            </div>
        </div>

        <div class="tool-card" data-title="MindStudio Insight" data-desc="全流程可视化性能分析套件，支持 Timeline、算子清单及智能瓶颈定位。">
            MindStudio Insight
        </div>

        <div id="dynamic-preview">
            <div class="preview-h" id="pre-title"></div>
            <p class="preview-p" id="pre-desc"></p>
        </div>
    </div>
</div>

<script>
const preview = document.getElementById('dynamic-preview');
const pTitle = document.getElementById('pre-title');
const pDesc = document.getElementById('pre-desc');
const container = document.getElementById('container');
const targets = document.querySelectorAll('.ai-arch-container .module, .ai-arch-container .tool-card');

targets.forEach((target) => {
    target.addEventListener('mouseenter', () => {
        if (window.innerWidth <= 850) return;

        pTitle.innerText = target.getAttribute('data-title');
        pDesc.innerText = target.getAttribute('data-desc');

        const rect = target.getBoundingClientRect();
        const containerRect = container.getBoundingClientRect();

        preview.style.top = (rect.top - containerRect.top) + 'px';
        preview.style.left = (rect.right - containerRect.left + 15) + 'px';
        preview.classList.add('active');
    });

    target.addEventListener('mouseleave', () => {
        preview.classList.remove('active');
    });
});
</script>

## 工具链选择建议

| 采集方式 | 特点 | 推荐场景 |
| --- | --- | --- |
| msprof命令行 | 功能全面，支持采集AI任务性能、系统数据，提供解析能力 | 各类训练/推理场景，特别是非PyTorch/MindSpore框架或需精细控制采集过程 |
| torch_npu.profiler | API接口，对标PyTorch GPU生态，无缝迁移 | PyTorch框架模型的常规性能分析 |
| mindspore.profiler | 与MindSpore深度集成，使用简单 | MindSpore框架模型的常规性能分析 |
| dynamic_profile动态采集 | 可随时开启/停止，动态修改配置，无需改动脚本 | 启停成本高的场景（超大规模集群、长时间运行任务） |


## 性能数据

需要使用 `PyTorch Profiler` 或 `MindSpore Profiler` 采集。  

| 数据类型            | 说明                                                                 |
|------------------|--------------------------------------------------------------------|
| Python调用栈        | 记录Python代码的执行过程，各个函数的调用关系与耗时情况。                                    |
| AI框架层Trace       | 记录AI框架（如PyTorch）算子下发耗时。 AI框架层-->CANN层为第一级下发流水。                     |
| CANN Trace       | 包含 AscendCL、GE、Runtime 组件以及 Node（算子）的耗时数据，CANN层-->Device为第二级下发流水。  |
| Ascend Hardware  | 底层 NPU 数据，即任务调度信息。记录 AI 任务运行时，各个 Task 在不同加速器下的执行耗时以及 AI Core 性能指标。 |
| Communication    | 记录各个通信域下的通信算子信息，包含通信带宽、数据传输量等信息。                                   |
| Overlap Analysis | 按照NPU是否在执行通信或计算任务进行划分，拆解成计算、通信、通信未与计算掩盖、空闲这四个维度，评估计算与通信的并行效率       |

## 相关入口

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

-   **[msProf](../msprof/)**

    ---

    采集与解析 CANN 平台及昇腾 AI 处理器的软硬件性能数据。

-   **[PyTorch Profiler](../torch_npu_profiler/)**

    ---

    基于PyTorch框架开发的昇腾软硬件性能调优工具。

-   **[MSPTI](../mspti/)**

    ---

    面向昇腾设备的 Profiling API 集合。

-   **[msMonitor](../msmonitor/)**

    ---

    面向昇腾集群场景的在线性能监控与动态采集工具。

-   **[msprof-analyze](../msprof-analyze/)**

    ---

    面向 AI 训练与推理场景的性能分析工具，具备性能比对、集群分析、专家建议等功能。

</div>
