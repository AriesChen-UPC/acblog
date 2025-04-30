---
title: "EDGS: 消除致密化以实现高效三维高斯溅射收敛及其相关研究进展分析"
description: ""
tags: [
    "3DGS",
    "EDGS",
    "Projects",
]
date: "2025-04-30"
categories: [
    "docs",
    "Development",
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 3


---

## **1\. 引言：迈向高效的三维高斯溅射**

三维高斯溅射（3D Gaussian Splatting, 3DGS）作为一种新兴的三维场景表示与新视角合成技术，近年来在计算机视觉和图形学领域引起了广泛关注 1。该技术通过使用数百万个可学习的三维高斯基元来显式表达场景 2，并结合高效的可微分渲染管线 4，实现了前所未有的实时渲染速度和照片级的视觉质量 4。相较于先前主流的基于神经网络辐射场（NeRF）的隐式方法 5，3DGS 在渲染效率上取得了革命性突破，避免了 NeRF 中耗时的体积渲染采样过程 5。凭借这些优势，3DGS 被迅速应用于各种下游任务，包括场景重建、虚拟现实、机器人学、自动驾驶、数字人以及三维内容生成等 11。

然而，标准的 3DGS 方法在取得卓越成就的同时，也面临着严峻的效率挑战，尤其是在模型优化（训练）阶段 5。为了达到高保真度的重建效果，标准 3DGS 通常需要优化数百万个高斯基元 2，并且其优化过程包含一个复杂且耗时的“自适应密度控制”（Adaptive Density Control, ADC）环节，特别是其中的“致密化”（Densification）步骤 5。这一过程不仅显著增加了训练时间（通常需要数十分钟甚至数小时 16），还导致了巨大的内存和存储开销（模型文件可达数百兆字节 23），限制了其在资源受限设备（如移动设备、边缘计算平台、AR/VR 头显）上的部署和应用 19。渲染速度与训练/存储成本之间的巨大差距，成为了推动 3DGS 效率优化研究的核心驱动力。

正是在这样的背景下，研究论文 “EDGS: Eliminating Densification for Efficient Convergence of 3DGS” 6 应运而生。EDGS 提出了一种旨在解决 3DGS 效率瓶颈的创新方法，其核心思想是彻底消除标准 3DGS 中缓慢的致密化步骤。本报告旨在深入剖析 EDGS 的核心内容，包括其试图解决的问题、提出的主要方法、关键创新点及实验效果。同时，报告将详细阐述标准 3DGS 的工作流程及其致密化步骤的作用与弊端，并在此基础上，广泛调研和梳理近期旨在提升 3DGS 效率（训练速度、收敛性、内存占用）的相关研究进展。最后，报告将对当前高效 3DGS 研究领域的现状进行概述，明确 EDGS 方法的贡献与地位，并探讨该领域的未来发展方向。

## **2\. EDGS 方法详解：消除致密化以加速收敛**

### **2.1 EDGS 旨在解决的问题**

EDGS 方法的核心目标是解决标准 3DGS 中**固有缓慢且迭代的致密化过程**所带来的效率问题 21。标准 3DGS 的流程通常始于由运动恢复结构（Structure-from-Motion, SfM）算法产生的稀疏点云 5，然后通过迭代优化和致密化（即在高斯基元梯度较大的区域进行分裂或克隆操作）来逐步精化场景表示，填补欠重建区域 5。

EDGS 的研究者指出，这种标准的增量式致密化方法存在以下关键弊端：

1. **过程缓慢（Slowness）**: 致密化过程需要大量的优化迭代。高斯基元必须反复调整其参数（位置、形状、颜色等），模型才能判断哪些区域需要增加新的基元 21。这导致了一个漫长的优化路径 21。致密化操作本身虽然计算效率尚可，但整个迭代发现和填充细节的过程拖慢了整体收敛速度，因为模型需要等待足够多的迭代才能识别出需要更高重建保真度的区域 21。  
2. **收敛延迟（Delayed Convergence）**: 由于模型需要“等待”并“发现”何处需要添加细节，这本质上是低效的 21。研究者通过一个生动的类比指出，这种迭代精化的方式类似于艺术家创作（先勾勒轮廓再添加细节），但这与相机一次性捕捉所有光线信息的工作方式不同 21。等待模型自行发现细节的过程延迟了收敛。  
3. **次优质量（Suboptimal Quality）**: 增量式的方法可能导致最终渲染质量并非最优，尤其是在包含丰富细节和复杂纹理的高频区域，标准 3DGS 及其它方法常常难以处理 21。

EDGS 正是为了克服上述标准 3DGS 致密化过程带来的速度慢、收敛延迟和潜在的质量瓶颈而提出的。

### **2.2 EDGS 核心方法：基于三角化稠密对应的密集初始化**

面对标准 3DGS 的挑战，EDGS 提出了一种“根本上不同的方法”：**完全消除致密化过程** 21。其核心机制是通过**利用跨多个输入视图的稠密图像对应关系进行像素三角化，实现场景几何的一步近似** 21。

具体流程如下：

1. **稠密对应计算**: EDGS 首先在输入的训练图像对之间计算稠密的二维（2D）像素对应关系 21。这与传统方法依赖 SfM 产生的稀疏关键点形成对比 21。  
2. **三角化恢复三维位置**: 已知每个对应像素的观察射线方向和相机的精确位姿（但不知道沿射线的深度），EDGS 通过**三角化匹配的像素对**来恢复这些像素对应的三维（3D）空间位置 21。这是一个经典的计算机视觉几何问题，利用多视图几何约束来估计点的 3D 坐标。  
3. **密集高斯基元预计算**: 通过对所有找到的稠密对应进行三角化，EDGS **预先计算出一个密集的 3D 高斯基元集合** 21。这个集合旨在直接近似场景的几何形状，并保留输入 RGB 图像中的丰富细节 21。  
4. **高斯参数初始化**: 对于每个通过三角化生成的 3D 点，EDGS 会为其分配一个高斯基元，并从一开始就赋予其**充分信息的初始属性**，包括：  
   * **位置 (Position)**: 直接使用三角化得到的 3D 坐标 21。  
   * **颜色 (Color)**: 利用对应 2D 像素的 RGB 颜色信息进行初始化，例如取平均值或根据某种策略选择 21。  
   * **尺度/协方差 (Scale/Covariance)**: 根据图像信息进行初始化 21。虽然具体机制未详述，但可能与三角化的重投影误差、对应点距离或局部图像梯度有关，目的是提供一个初始的大小和方向估计。

通过这种方式，EDGS 用一个密集的、预先计算好的高斯集合替换了标准 3DGS 中缓慢、迭代的致密化过程 21。每个高斯基元从优化开始就受到丰富的、来自对应像素的光度信号的直接监督，使得整个集合能够被高效地优化，从而显著加速收敛 21。

### **2.3 EDGS 的关键创新与贡献**

EDGS 的核心创新和主要贡献可以总结为以下几点：

1. **彻底消除致密化**: 最根本的创新在于用直接的密集初始化完全取代了标准 3DGS 中迭代式的致密化步骤（分裂与克隆） 21。  
2. **基于稠密对应的初始化策略**: 采用稠密对应而非稀疏关键点进行初始化，确保了即使在标准 3DGS 难以处理的高频区域也能获得均匀的细节覆盖 21。  
3. **显著缩短优化路径**: 通过提供一个更接近最终状态的密集且信息丰富的初始化，EDGS 极大地减少了每个高斯基元在参数空间中需要移动的距离，从而显著缩短了优化路径 21。实验表明，最终坐标位移减少了 50 倍，坐标总路径长度缩短了 30 倍 29。  
4. **并行化初始化**: 所有的高斯基元在优化开始时并行初始化，消除了标准致密化中需要等待模型调整现有基元才能添加新基元的串行依赖和等待时间 21。  
5. **高质量与高效率**: EDGS 不仅在训练效率上超越了速度优化的模型，而且在渲染质量上优于现有先进方法，同时使用的基元数量仅为标准 3DGS 的一半左右 21。  
6. **兼容性**: EDGS 方法与其他 3DGS 加速技术完全兼容 21。由于其主要改进在于初始化阶段，并未根本改变高斯表示或渲染管线，因此可以灵活地集成到现有的加速框架中，作为一种通用的效率提升方案。

### **2.4 EDGS 实验结果概览**

根据论文及其项目页面的信息，EDGS 在实验中展现出显著的优势：

* **效率**: 收敛速度比标准 3DGS 快约 10 倍 29。对于前向视角视频等场景，包含稠密对应计算的一次性成本在内，可在 10-20 秒内完成训练 21。训练效率优于其他以速度优化为目标的模型 21。  
* **质量**: 最终渲染质量（如 PSNR, SSIM, LPIPS 等指标衡量）高于标准 3DGS 及其他先进方法 21。在包含复杂纹理和几何结构的区域，保真度更高 21。  
* **紧凑性**: 达到同等或更高质量时，使用的 3D 高斯基元数量仅为标准 3DGS 的一半左右 21。

EDGS 的方法论体现了一种范式转变：从标准 3DGS 的“被动式精化”（等待误差或梯度信号触发致密化）转向了“主动式近似”。它假设现代的稠密对应算法已经足够强大，能够提供一个足够精确的初始几何支架，从而绕过模型迭代“发现”场景结构的需要。这种策略如果成功，就能通过更好的初始化直接达到或接近最终目标，避免了低效的探索和调整过程，自然地带来更快的收敛速度。同时，由于初始化更接近真实表面，所需的基元数量也可能更少，并且最终的优化结果可能落入一个更好的局部最优，从而获得更高的渲染质量。这种对初始化阶段的侧重，也解释了其与其他主要作用于优化或渲染阶段的加速技术的兼容性 21。

## **3\. 标准三维高斯溅射（3DGS）工作流程解析**

为了更好地理解 EDGS 的创新之处，有必要先深入了解标准 3DGS 的工作流程，特别是其核心的自适应密度控制（ADC）机制。

### **3.1 标准 3DGS 工作流程概述**

标准的 3DGS 方法旨在从多视图图像中高效地学习场景的三维表示并实现高质量的新视角合成。其主要流程包括以下步骤：

1. **初始化 (Initialization)**:  
   * 通常从稀疏的三维点云开始，这些点云一般是通过对输入图像运行运动恢复结构 (SfM) 算法免费获得的 5。  
   * 每个 SfM 点被初始化为一个 3D 高斯基元，并赋予初始的属性值（如位置、接近零的协方差、平均颜色等）。在某些数据集（如 NeRF-Synthetic）上，随机初始化也能取得较好效果 5。  
2. **场景表示 (Representation)**:  
   * 整个三维场景被显式地表示为大量（通常是数百万个）3D 高斯基元的集合 1。  
   * 每个高斯基元 Gk​ 由一组可学习的参数定义，通常包括 1：  
     * 中心位置 (Mean): μk​∈R3  
     * 协方差矩阵 (Covariance): Σk​∈R3×3。为了保证正半定性并便于优化，通常将其分解为一个缩放因子向量 sk​∈R3 和一个表示旋转的四元数 qk​ 9。协方差矩阵决定了高斯基元的形状和大小（各向异性）。  
     * 不透明度 (Opacity): αk​∈R，控制基元的透明度。  
     * 视角相关颜色 (View-dependent Color): 通常使用球谐函数 (Spherical Harmonics, SH) 系数 fk​ 来表示，使得基元从不同方向观察时呈现不同的颜色 9。  
3. **优化 (Optimization)**:  
   * 使用随机梯度下降 (SGD) 或其变种 (如 Adam) 来优化所有高斯基元的参数 5。  
   * 优化的目标是最小化渲染图像与对应的真实训练图像之间的差异。损失函数通常结合了 L1 损失和结构相似性指数 (D-SSIM) 损失 9。  
   * 优化过程与自适应密度控制 (ADC) 交错进行 4。  
4. **渲染 (Rendering)**:  
   * 采用一种高效的可微分光栅化器 (Tile-based Rasterizer) 4。  
   * 对于给定的新视角，首先将所有 3D 高斯基元投影到该视角的 2D 图像平面上，形成 2D 高斯 9。投影过程考虑了透视变换和仿射近似 9。  
   * 然后，对于图像中的每个像素，收集所有覆盖该像素的 2D 高斯基元。  
   * 将这些覆盖像素的基元按照深度进行排序。  
   * 最后，使用 alpha 合成 (Alpha Blending) 的方式，从前到后依次混合这些基元的颜色和不透明度，得到该像素的最终颜色 9。  
   * 这个基于光栅化的渲染过程避免了 NeRF 中逐点采样和体积渲染的巨大计算开销，是 3DGS 实现实时渲染的关键 5。

### **3.2 自适应密度控制 (ADC) 的作用与必要性**

自适应密度控制 (ADC) 是标准 3DGS 框架中不可或缺的一部分 1。它的主要目的是在优化过程中动态地调整场景中高斯基元的数量和分布，以适应不同区域的几何复杂度和细节需求 4。ADC 主要包含两个相互关联的操作：致密化 (Densification) 和剪枝 (Pruning)。

**ADC 的必要性体现在以下几个方面：**

1. **从稀疏到稠密**: 由于初始化通常基于稀疏的 SfM 点云 5，这些点不足以精细地表达整个场景的几何和外观。ADC 中的致密化步骤能够根据需要自动增加高斯基元的数量，特别是在几何结构复杂或纹理细节丰富的区域，从而弥补初始化的不足，提升重建质量 4。  
2. **资源有效分配**: 场景的不同区域具有不同的复杂度。例如，平坦光滑的区域可能只需要少量较大的高斯基元来表示，而细节丰富的区域则需要大量较小的高斯基元 42。ADC 允许模型将计算资源（即高斯基元）集中分配到最需要的区域，避免在简单区域浪费资源。  
3. **控制模型复杂度**: 如果只增不减，高斯基元的数量可能会无限制地增长，导致过拟合、计算开销增大和存储需求膨胀 18。ADC 中的剪枝步骤负责移除冗余或无效的基元，有助于维持一个相对紧凑且高效的场景表示 4。  
4. **提高鲁棒性**: ADC 提供了一种自动管理基元生命周期的机制，使得 3DGS 对初始化的依赖性相对降低（相比于需要稠密 MVS 点云的早期点基方法 5），并能在优化过程中纠正一些初始表示的不足。

### **3.3 致密化 (Densification) 机制详解**

致密化是 ADC 中负责增加高斯基元数量的过程，旨在提高场景在欠重建区域的表达能力 4。标准 3DGS 中的致密化主要通过两种方式进行：克隆 (Cloning) 和分裂 (Splitting)。

* **触发条件**: 致密化的主要触发条件是高斯基元在视图空间中的**位置梯度**的大小 1。具体来说，计算每个高斯基元在所有可见视图中的平均 2D 位置梯度幅度 τk​。如果 τk​ 超过一个预设的阈值 Tgrad​ （例如，原始论文中设为 0.0002 1），则该高斯基元被选为致密化候选者。其背后的直觉是，如果一个高斯基元在优化过程中频繁移动（即具有较大的位置梯度），说明它可能试图覆盖一个较大的区域或者一个模型难以精确拟合的复杂区域，因此需要增加该区域的基元密度 1。后续研究也探索了基于像素渲染误差的触发条件 36。  
* **克隆 (Cloning)**: 如果一个候选致密化的高斯基元相对“小”（其最大尺度 max(sk​) 小于场景范围 escene​ 的某个百分比 Pdense​，例如 max(sk​)\<Pdense​⋅escene​ 42），则执行克隆操作 1。克隆会创建一个与父基元参数完全相同的副本，并将其添加到场景中 42。父基元和克隆出的子基元随后会独立进行优化 4。克隆操作旨在增加小尺度细节区域的基元数量。  
* **分裂 (Splitting)**: 如果一个候选致密化的高斯基元相对“大”（即 max(sk​)≥Pdense​⋅escene​ 42），则执行分裂操作 1。分裂会将原始的父基元移除，并替换为多个（通常是 Nchildren​=2 个 42）新的子基元。这些子基元的位置通常根据父基元的概率分布进行采样，位于父基元的空间范围内 42。子基元的尺度通常会按比例缩小（例如，缩小为父基元尺度的 1/Nchildren​​ 或乘以一个固定的缩放因子，如 1.6 或 5 的倒数 42），而不透明度和颜色等属性则继承自父基元。分裂操作旨在将一个覆盖范围过大的基元细化为多个更小的基元，以更好地捕捉大尺度结构内部的细节。  
* **执行频率**: 致密化操作通常在优化的早期阶段比较频繁地执行（例如，每 100 次迭代执行一次 4），并且可能在达到一定的迭代次数（例如 15k 次迭代 42）后停止或降低频率，以避免后期过拟合。一些研究提出了动态调整梯度阈值 Tgrad​ 的策略，例如早期使用较低阈值鼓励快速增加密度，后期使用较高阈值限制增长 1。

### **3.4 剪枝 (Pruning) 机制详解**

剪枝是 ADC 中负责移除冗余或无效高斯基元的过程，目的是控制模型大小，提高效率，并消除潜在的视觉伪影 4。标准 3DGS 主要采用以下几种剪枝策略：

* **基于不透明度的剪枝 (Opacity Pruning)**: 这是最主要的剪枝方式。如果一个高斯基元的不透明度 αk​ (或 ok​) 降低到一个非常小的值（低于预设的最小不透明度阈值 omin​，例如接近于 0 42），则认为该基元对最终渲染结果的贡献微乎其微，可以安全地将其从场景中移除 1。  
* **基于尺寸的剪枝 (Size Pruning)**: 如果一个高斯基元变得异常大（例如，其最大尺度 max(sk​) 超过场景范围 escene​ 的某个较大比例，如 10% 42），也可能被视为无效或伪影而被移除 42。一些方法也在优化过程中对过大的尺度进行约束或重置 27。  
* **不透明度重置 (Opacity Reset)**: 为了辅助基于不透明度的剪枝，标准 3DGS 会周期性地（例如，每 3000 次迭代 42）将所有高斯基元的不透明度强制重置为一个非常小的值（例如 0） 31。这迫使优化过程重新“证明”每个基元的必要性——只有那些对降低渲染损失确实有贡献的基元才能在后续优化中重新获得较高的不透明度，而不再需要的基元则会保持低不透明度，最终在下一次剪枝步骤中被移除 42。  
* **执行频率**: 剪枝操作通常在每次致密化操作之后立即执行 42，但不透明度重置的频率较低 42。

通过致密化和剪枝这两种互补的机制，ADC 使得 3DGS 能够从稀疏初始化开始，自适应地调整基元密度以匹配场景复杂度，最终在保持相对高效表示的同时实现高质量的重建 31。然而，正如 EDGS 所指出的，这个过程本身也带来了显著的挑战。

## **4\. 标准 3DGS 致密化过程的挑战与弊端**

尽管自适应密度控制（ADC）及其致密化步骤是标准 3DGS 成功的关键因素之一，但这个过程本身也引入了一系列挑战和固有的弊端，这些正是 EDGS 及其他效率优化方法试图解决的核心问题。

### **4.1 计算开销与训练时间**

* **迭代性质导致耗时**: 致密化过程的迭代性质是其最显著的缺点之一。模型需要经历大量的优化步骤来调整现有高斯基元的参数，然后根据梯度或误差信号决定在何处以及如何添加新的基元 21。这个“优化-评估-致密化”的循环需要重复多次，极大地延长了总训练时间 21。虽然单次致密化操作（分裂或克隆）本身的计算量可能不大，但累积起来的开销以及在致密化之间进行的必要优化步骤构成了主要的计算负担 21。  
* **冗长的优化路径**: 由于需要等待模型“发现”需要细化的区域，高斯基元在达到其最终稳定状态之前，可能会经历多次移动、调整、甚至被克隆或分裂 21。这意味着每个基元在参数空间中走过了漫长而曲折的路径，这不仅耗时，也可能不是最高效的收敛方式 29。EDGS 的实验通过量化坐标移动距离的显著减少，直观地证明了标准方法优化路径的冗长性 29。  
* **实际训练时间**: 对于复杂场景，标准 3DGS 的训练时间通常以分钟甚至小时为单位 16，这与其实时渲染能力形成了鲜明对比，限制了其在需要快速建模或迭代的应用场景中的实用性。

### **4.2 内存占用与存储开销**

* **模型规模不可控**: 标准 3DGS 的 ADC 机制通常缺乏对最终高斯基元数量的严格上限控制 17。致密化过程可能会产生远超预期的基元数量，尤其是在复杂场景或优化参数设置不当的情况下。这导致了所谓的“无界模型大小”问题 17。  
* **内存溢出风险**: 不受控制的基元数量增长会急剧增加训练过程中 GPU 内存的需求 24。对于大规模场景或在内存有限的硬件上进行训练时，很容易发生内存溢出 (Out-of-Memory, OOM) 错误 27。  
* **巨大的存储需求**: 最终模型需要存储数百万个高斯基元的全部参数（位置、尺度、旋转、不透明度、球谐系数等） 2。这导致最终的模型文件体积庞大，通常达到数百兆字节 23，远超许多基于 NeRF 的紧凑表示 23。巨大的存储体积不利于模型的分发、加载和在存储资源有限的设备（如移动端、嵌入式系统）上的应用 19。

### **4.3 对收敛速度与优化稳定性的影响**

* **延迟收敛**: 致密化过程本身就延迟了模型的收敛 21。模型需要花费大量迭代时间来探测和识别哪些区域需要更高的重建保真度，然后才能通过添加新基元来提升这些区域的质量 21。相比之下，如果能从一开始就提供一个更完善的初始化，收敛速度有望大幅提升。  
* **启发式规则的脆弱性**: ADC 依赖于一系列启发式规则和超参数（如梯度阈值、尺寸阈值、不透明度阈值、执行频率等） 18。这些参数通常需要针对特定数据集或场景进行仔细调整，缺乏普适性 18。不合适的参数设置可能导致致密化不足（质量差）或过度致密化（效率低、伪影多）。这种对启发式规则的依赖使得优化过程显得有些“脆弱” 18。  
* **优化轨迹的扰动**: 周期性的操作，如不透明度重置，虽然有助于剪枝，但也可能对优化轨迹造成“冲击”，引入不稳定性，不利于平稳和可预测的训练动态 36。

### **4.4 对渲染质量与几何精度的潜在负面影响**

* **高频区域的次优渲染**: 增量式的致密化方法可能难以完美捕捉场景中的所有高频细节，尤其是在纹理复杂或几何精细的区域，可能导致渲染结果次优或模糊 21。  
* **初始化依赖与伪影**: 如果 SfM 初始化质量较差（例如，在纹理重复或弱纹理区域点过少），标准的基于梯度的致密化可能无法有效增加这些区域的基元密度 32。这会导致渲染模糊或出现“针状”伪影 (Needle-like Artifacts)，即高斯基元变得在一个维度上极度拉长 9。仅仅降低致密化阈值并不能解决问题，反而可能在已经足够稠密的区域产生不必要的基元 32。  
* **漂浮物 (Floaters)**: 优化过程中可能会在空白区域产生一些 spurious 的高斯基元（漂浮物）。如果剪枝机制不够有效，这些漂浮物可能会残留下来，影响渲染质量 8。  
* **克隆操作的偏见**: 有研究指出，标准 ADC 在克隆操作中保留原始不透明度的做法会引入偏见，导致克隆区域整体不透明度偏高，影响 alpha 合成和后续的致密化过程 36。

综上所述，标准 3DGS 中的致密化步骤虽然是实现高质量重建的关键环节，但其迭代、启发式的本质带来了显著的计算、内存、收敛性和质量方面的挑战。这些挑战不仅限制了 3DGS 的应用范围和效率，也成为了驱动研究者们探索如 EDGS 这样消除或改进致密化方案的强大动力。理解这些弊端有助于认识到 EDGS 等方法试图通过更优的初始化或替代策略来从根本上提高 3DGS 效率和鲁棒性的重要意义。同时，这些挑战也解释了为何除了改进初始化之外，还有大量研究工作致力于优化训练策略、压缩模型表示以及改进核心的高斯基元本身。

## **5\. EDGS 技术实现深入探究**

EDGS 的核心在于其创新的初始化策略，该策略完全取代了标准 3DGS 中的自适应密度控制和迭代致密化。本节将深入探讨实现 EDGS 所涉及的关键技术环节。

### **5.1 稠密对应查找策略**

EDGS 的起点是获取输入图像之间的稠密 2D 对应关系 21。论文强调了使用“稠密”对应，以区别于传统 SfM 流程中使用的“稀疏”关键点匹配 21。稠密对应的目标是为图像中的尽可能多的像素（理想情况下是每一个像素）找到其在其他视图中的对应像素。

* **重要性**: 稠密对应是 EDGS 方法的基石。对应点的数量和精度直接决定了后续三角化生成的 3D 点云的密度和准确性，进而影响初始高斯基元集合的质量。只有足够稠密且准确的对应关系，才能确保初始化能够覆盖场景的各个部分，尤其是在高频区域提供足够的细节 21。  
* **潜在技术**: 尽管 EDGS 的公开资料（如提供的摘要和项目页面）没有明确指定所使用的具体稠密对应算法，但该领域存在多种成熟的技术可以实现这一目标。例如：  
  * **光流法 (Optical Flow)**: 像 RAFT (Recurrent All-Pairs Field Transforms) 及其变种是当前先进的光流估计算法，能够计算图像间密集的像素位移场。  
  * **基于特征匹配的方法**: 如 LoFTR (Local Feature TRansformer) 或 SuperGlue 等利用深度学习进行特征提取和匹配的方法，也能生成相对稠密的对应关系，尤其在具有挑战性的场景（如弱纹理、大视角变化）下表现鲁棒。  
  * **传统立体匹配/多视图立体 (MVS) 算法**: 经典的立体匹配算法也可以输出稠密的视差图或深度图，从中可以推导出像素对应关系。  
* **选择考量**: EDGS 的实现需要选择一种能够平衡精度、密度和计算效率的稠密对应算法。算法的选择将直接影响 EDGS 的预处理时间（计算对应关系是一次性成本 21）和最终的重建质量。

### **5.2 三角化恢复三维位置**

在获得稠密 2D 对应和已知相机位姿（通常由 SfM 或其他相机标定方法提供）后，EDGS 通过三角化来恢复这些对应像素点的三维空间位置 21。

* **基本原理**: 三角化是利用来自至少两个不同视角的观察射线来确定空间中某一点位置的过程。对于一对匹配的像素 (pi​,pj​)，它们分别位于图像 Ii​ 和 Ij​ 上。已知相机 i 和 j 的内参和外参，可以确定从相机中心出发、穿过像素 pi​ 和 pj​ 的两条空间射线。理想情况下，这两条射线应该交于空间中的同一点，即对应场景点的三维位置 X。由于噪声和测量误差，射线通常不会精确相交，因此三角化算法的目标是找到一个 3D 点 X，它能最好地解释这两条（或多条）观察射线（例如，使该点到所有射线的距离之和最小，或者使该点在所有图像上的重投影误差最小）。  
* **关键信息**: EDGS 明确指出，这个过程利用了已知的相机位姿和观察射线，但**不需要预先计算的深度图** 21。这是三角化的标准设定。  
* **潜在算法**: 存在多种三角化算法，从简单的线性方法（如直接线性变换 DLT 的变种）到更复杂的非线性优化方法（如基于重投影误差最小化的方法）。考虑到需要处理大量的稠密对应点，EDGS 可能采用了一种计算效率较高且相对鲁棒的三角化技术。虽然具体算法未指明，但可以参考更广泛的计算机视觉文献，例如 47 讨论了多种三角剖分技术（如 Delaunay 三角剖分），48 讨论了基于图像一致性的三角剖分优化，这些概念可能与 EDGS 的实现相关或提供了背景知识。

### **5.3 高斯基元参数的初始化**

通过三角化得到密集的 3D 点云后，EDGS 为每个点初始化一个高斯基元，并赋予其初始参数 21。

* **位置 (**μ**)**: 直接使用三角化计算得到的 3D 坐标作为高斯基元的初始中心位置 21。  
* **颜色 (**f**)**: 利用原始输入 RGB 图像中对应 2D 像素的颜色信息来初始化 21。一种简单的方法是取所有参与三角化的对应像素颜色的平均值。对于视角相关颜色（如 SH 系数），可能将平均颜色作为零阶 SH 系数，并将高阶系数初始化为零。  
* **尺度/协方差 (**s,q **或** Σ**)**: 这是初始化中相对不明确的一环。论文提到会根据图像信息初始化尺度 21，但未提供细节。合理的初始化对于后续优化至关重要。可能的策略包括：  
  * 基于三角化的不确定性或重投影误差来估计初始尺寸。  
  * 基于邻近 3D 点的距离来设定初始尺度。  
  * 考虑对应像素周围的图像梯度或纹理信息。  
  * 将初始协方差设置为各向同性的小球体，让优化过程自行学习其形状和大小。  
* **不透明度 (**α**)**: EDGS 的资料中未明确提及不透明度的初始化。通常，在 3DGS 中，不透明度会被初始化为一个非零的小值（例如，通过 sigmoid 函数作用于一个可学习的参数），以确保基元在优化初期是可见的，并能接收到梯度信号。

这种“信息丰富”的初始化是 EDGS 的核心优势。与标准 3DGS 从非常稀疏且属性简单的 SfM 点出发相比，EDGS 的初始高斯基元集合不仅在数量上密集得多，而且在位置、颜色和可能的尺度上都更接近场景的真实状态。

### **5.4 替代致密化的机制：直接优化固定集合**

EDGS 的关键在于，通过上述的密集初始化步骤，它**完全避免了在优化过程中动态增加高斯基元的需要** 21。

* **固定基元集合**: 一旦通过稠密对应和三角化生成了初始的密集高斯基元集合，EDGS 的优化过程就只针对这个**固定的集合**进行 21。优化的目标是调整这些初始基元的参数（位置、颜色、尺度、旋转、不透明度），使其更好地拟合训练视图。  
* **无分裂与克隆**: 不再有基于梯度或误差的分裂和克隆操作。场景的几何复杂性完全由初始化的密度和后续参数的优化来表达。  
* **剪枝的可能性**: 虽然 EDGS 消除了致密化，但它是否保留了剪枝机制（例如，移除优化后不透明度过低的基元）在提供的资料中并未明确说明。考虑到剪枝对于移除无效基元和控制最终模型大小的重要性，保留某种形式的剪枝（如基于低不透明度的剪枝）是可能的，并且这通常不与初始化策略冲突。

EDGS 的基本假设是，如果稠密对应和三角化足够好，产生的初始点云已经足够密集且准确地覆盖了场景表面，那么就不再需要一个额外的机制来“发现”和“填充”缺失的几何信息。优化过程只需要对这个初始集合进行微调，就能达到高质量的重建。这种策略的成功高度依赖于上游的稠密对应和三角化步骤的质量。如果这些步骤产生大量错误或噪声，由于缺乏动态添加基元的机制来纠正大的初始错误，EDGS 的性能可能会受到影响。然而，其报告的优异结果表明，在常用基准数据集上，现代稠密对应技术似乎能够提供足够高质量的输入，使得这种“消除致密化”的策略是可行的，并且在效率和质量上都带来了显著的提升。

## **6\. 性能分析与比较**

EDGS 的提出旨在显著提升 3DGS 的效率和质量。本节将基于现有信息，对其性能进行分析，并与标准 3DGS 及其他相关方法进行比较。

### **6.1 定量比较：EDGS vs. 标准 3DGS**

EDGS 论文及其项目页面提供了令人信服的定量数据，证明了其相对于标准 3DGS 的优势：

* **训练时间与收敛速度**:  
  * **显著加速**: EDGS 的收敛速度据称比标准 3DGS 快约 10 倍 29。这是一个巨大的提升，意味着训练时间可以从几十分钟缩短到几十秒或几分钟。  
  * **绝对时间**: 对于某些场景（如前向视角视频），EDGS 可以在 10-20 秒内完成训练 29，这个速度已经接近实时交互的需求。需要注意的是，报告的时间通常包含了计算稠密对应关系的一次性预处理成本 21。  
  * **优化路径**: 通过大幅缩短优化路径（坐标总路径长度减少 30 倍，最终位移减少 50 倍 29），EDGS 证明了其初始化策略使得优化过程更加直接高效 21。  
  * **超越速度优化模型**: EDGS 声称其训练效率甚至优于那些专门为速度优化的 3DGS 变体 21。  
* **内存占用与模型紧凑性**:  
  * **更少的基元**: EDGS 在达到同等或更高质量的情况下，使用的 3D 高斯基元数量大约只有标准 3DGS 的一半 21。  
  * **内存与存储**: 更少的基元数量直接意味着更低的训练时 GPU 内存占用和更小的最终模型存储体积。这对于在资源受限环境下部署 EDGS 模型至关重要。  
* **渲染质量**:  
  * **更高保真度**: EDGS 报告了比标准 3DGS 和其他当前最优 (State-of-the-Art, SOTA) 方法更高的渲染质量 21。这通常通过标准的图像质量评估指标来衡量，如峰值信噪比 (PSNR)、结构相似性指数 (SSIM) 和学习感知图像块相似度 (LPIPS) 49。  
  * **复杂区域优势**: EDGS 在处理包含复杂纹理和精细几何结构的区域时表现尤为出色，能够重建出比标准方法更丰富的细节 21。

**表 1: EDGS 与标准 3DGS 性能对比概要**

| 性能指标 | 标准 3DGS (基线) | EDGS | 优势来源 (EDGS) | 相关文献 |
| :---- | :---- | :---- | :---- | :---- |
| **训练收敛速度** | 较慢 (分钟级) | 显著加快 (约 10x)，可达秒级 29 | 消除迭代致密化，缩短优化路径 21 | 21 |
| **高斯基元数量** | 较多 (N) | 显著减少 (约 N/2) 21 | 密集初始化更有效，无需过度增殖 | 21 |
| **内存/存储占用** | 较高 | 较低 (因基元数量减少) | 更紧凑的表示 | (推断自基元数量减少) |
| **渲染质量 (PSNR/SSIM/LPIPS)** | 高，但在高频区域可能次优 21 | 更高，尤其在复杂区域保真度更好 21 | 更好的初始化引导优化至更优解 21 | 21 |
| **初始化方法** | 稀疏 SfM 点 \+ 迭代致密化 5 | 稠密对应三角化，一步密集初始化 21 | 直接近似几何，避免迭代发现 | 21 |

*注意：具体的 PSNR/SSIM/LPIPS 数值依赖于数据集和评估设置，此处主要描述相对性能。*

这种同时在速度、紧凑性和质量三个方面都取得显著提升的情况在优化研究中并不常见。通常，加速或压缩往往会牺牲一定的质量 19。EDGS 能够打破这种常规权衡，暗示其核心理念——通过高质量的密集初始化来避免低效的迭代式几何发现和精化——可能是一种在根本上更有效的方法。

### **6.2 与其他效率优化方法的比较**

EDGS 的独特之处在于它从**初始化**入手，彻底**消除**了致密化。这与其他主流的效率优化思路形成了对比：

* **改进致密化**: 像 Pixel-GS 32, 基于误差的致密化 36, AtomGS 37 等方法，它们承认 ADC 的重要性，但试图改进其触发条件或执行方式，使其更智能、更鲁棒，以解决标准 ADC 的特定问题（如弱纹理区域的欠致密化 32 或高频区域的失效 36）。这些方法保留了迭代增加基元的框架，因此在收敛速度上可能仍不及 EDGS，但它们可能对初始对应关系的噪声更具鲁棒性，因为它们仍然有动态调整密度的能力。  
* **优化策略与调度**: 如 DashGaussian 16 通过动态调整训练分辨率和基元数量上限来加速优化。RadSplat 17 利用 NeRF 作为先验来指导优化和剪枝。这些方法主要作用于**优化过程**本身。由于 EDGS 主要改变初始化阶段 21，理论上可以与这些优化策略结合使用，可能带来叠加的效率提升。  
* **压缩与量化**: CompGS 23, Self-Organizing Gaussians 25, Compact-3DGS 57 等方法专注于**减小最终模型的大小**，通过剪枝、向量量化、参数排序和编码等技术实现。这些通常在优化之后或优化过程中进行，旨在降低存储和传输成本。EDGS 本身通过减少基元数量实现了初步的紧凑性 21，并且其声称的兼容性 21 意味着可以进一步应用这些压缩技术来获得更小的模型。例如，EDGS 产生的较少基元可能使得后续的量化或编码更加高效。  
* **表示修改**: 一些方法通过修改高斯基元本身（如使用各向同性高斯 58）或结合其他表示（如 GSDF 40 中的 SDF，4DGF 59 中的神经场）来提升效率或几何精度。EDGS 并未改变核心的 3D 高斯表示，这再次支持了其与其他方法结合的可能性。

目前，公开资料主要集中在 EDGS 与标准 3DGS 的对比上。要全面评估 EDGS 在整个高效 3DGS 生态中的地位，还需要更多关于它与其他 SOTA 效率优化方法（尤其是在结合使用时）的直接比较数据。例如，EDGS \+ CompGS 与标准 3DGS \+ CompGS 在总时间、最终大小和质量上的对比将会非常有价值。

### **6.3 重建与渲染的定性差异**

除了定量指标，定性比较也揭示了 EDGS 的优势：

* **细节保真度**: EDGS 能够更好地重建复杂纹理和精细几何结构，渲染结果在这些区域比标准 3DGS 更清晰、更逼真 29。  
* **伪影减少**: 标准 3DGS 中因致密化不足或不当可能产生的模糊 36 或针状伪影 32 在 EDGS 中有望得到缓解，因为它依赖于更全面的初始几何信息。  
* **均匀性**: 基于稠密对应的初始化有助于在整个场景中实现更均匀的细节覆盖，而不是像标准方法那样可能只在梯度高的区域（如边缘）过度集中基元 21。

项目页面 29 提供的对比可视化（例如左右对比图）是理解这些定性差异的重要参考。

总的来说，性能分析表明 EDGS 是一种非常有前景的 3DGS 效率提升方法，它通过一种新颖的初始化策略，在加速收敛、减少模型复杂度和提升渲染质量方面都取得了显著的成果。其与其他加速技术的潜在兼容性进一步增强了它的吸引力。

## **7\. 高效 3DGS 相关研究进展概览**

EDGS 并非孤立的工作，它是近年来涌现的众多旨在提升 3DGS 效率的研究浪潮中的一员。这些研究从不同角度切入，试图解决标准 3DGS 在训练速度、内存占用、收敛性、模型大小等方面的局限性。根据其核心技术侧重点，可以将这些相关研究大致归纳为以下几个类别（部分工作可能跨越多个类别）：

### **7.1 替代性初始化与致密化策略**

这类方法的核心目标是改进或完全取代标准 3DGS 中基于稀疏 SfM 点和启发式梯度驱动的 ADC 初始化与增长机制。

* **EDGS**: 如前所述，通过稠密对应三角化进行一次性密集初始化，完全消除迭代致密化 21。  
* **基于像素误差的致密化 (Pixel-Error Driven Densification)**: 不再依赖位置梯度，而是使用渲染图像的像素级误差（例如基于 SSIM 计算的误差）来反向传播，识别需要增加基元的区域 36。这种方法试图建立更符合视觉感知的致密化标准，并改进对高频纹理区域的处理。它还修正了标准克隆操作中的不透明度偏见，并提供了更好的基元数量控制机制 36。  
* **Pixel-GS**: 针对标准梯度平均策略在稀疏初始化区域对大尺度高斯基元致密化不足的问题，提出根据高斯在每个视图中覆盖的像素面积来加权平均梯度 32。目的是在需要的地方（大而欠致密化的基元）有效促进分裂/克隆，同时避免在已足够稠密的区域过度增长 32。还提出了梯度缩放策略来抑制近处漂浮物 32。  
* **AtomGS**: 提出一种两阶段方法，首先使用“原子高斯 (Atom Gaussians)”在精细细节区域进行快速增殖以对齐几何，然后通过剪枝和融合策略来合并覆盖平滑表面的基元，旨在提升几何精度同时控制基元数量 37。  
* **基于 MCMC 的致密化 (MCMC-based Densification)**: 将高斯基元集合视为从描述场景物理表示的潜在概率分布中抽取的马尔可夫链蒙特卡洛 (MCMC) 样本 44。将致密化和剪枝重新解释为 MCMC 样本的确定性状态转移，试图移除启发式规则，并引入正则项鼓励移除未使用的基元，以提高对初始化的鲁棒性 44。  
* **渲染引导的致密化 (Rendering-Guided Densification)**: 特别是在 SLAM 应用中，利用渲染质量或几何一致性误差来指导致密化过程，以适应实时建图的需求 14 (如 HF-SLAM)。  
* **针对特定数据的致密化**: 如 ODGS 10 为全向（360°）图像设计了动态致密化规则，考虑了等距柱状投影造成的拉伸变形。LE3D 22 提出了锥形散布初始化 (Cone Scatter Initialization) 来增强夜间低信噪比 RAW 图像的 SfM 点云，以改善初始化质量。  
* **渐进式传播 (Progressive Propagation)**: 如 GaussianPro 36 提出一种基于估计平面、使用块匹配和几何一致性进行引导的渐进式基元传播方法，以填补 SfM 初始化的空白。

### **7.2 替代性优化与收敛策略**

这类方法着重于改进 3DGS 的训练过程本身，以加速收敛或提高优化鲁棒性。

* **训练调度 (Scheduling)**:  
  * **DashGaussian**: 提出一种优化复杂度调度方案，根据场景频率拟合的进度动态调整渲染分辨率和基元数量上限，旨在去除冗余计算，大幅加速多种 3DGS 骨干网络的训练 16。  
  * **粗到细训练 (Coarse-to-Fine Training)**: 从较低分辨率开始训练，然后逐渐增加分辨率。这被认为有助于移除漂浮物并提高学习效率 8。Opti3DGS 24 在粗到细框架中结合了频率调制。  
* **频率正则化 (Frequency Regularization)**: 如 FreGS 15 采用渐进式频率正则化来指导优化。  
* **利用先验信息或指导信号**:  
  * **NeRF 先验 (RadSplat)**: 使用预训练的 NeRF 模型作为 3DGS 优化的先验和监督信号，以提高在复杂真实场景中的优化稳定性和最终质量，并结合新的剪枝策略实现更快的渲染 17。  
  * **SDF 指导 (GSDF, NeuSG, 3DGSR)**: 将 3DGS 与神经符号距离场 (SDF) 相结合，通过两个分支的联合优化和相互指导来提升几何重建的准确性和渲染质量 40。SDF 可以指导高斯基元更贴近表面分布，减少漂浮物；同时，高斯基元也能加速 SDF 的收敛 40。

### **7.3 剪枝、稀疏化与量化技术**

这类方法的目标是在优化过程中或优化之后，通过移除冗余基元、降低基元参数精度或利用参数间的相关性来减小模型规模（内存占用和存储大小），并可能附带加速渲染。

* **剪枝与稀疏化**:  
  * **可学习掩码 (Learnable Masking)**: 如 Compact-3DGS 57 在训练中学习一个掩码来识别并移除不重要的高斯基元。  
  * **不透明度正则化 (Opacity Regularization)**: 在损失函数中加入不透明度的 L1 正则项，鼓励模型将不重要基元的不透明度驱动至零，从而使其在后续基于阈值的剪枝中被移除 23。  
  * **贡献/可见性剪枝 (Contribution/Visibility Pruning)**: 如 RadSplat 17 提出基于射线贡献或视点可见性的剪枝策略。  
  * **片段剪枝 (Fragment Pruning)**: 不同于移除整个高斯基元，该方法在渲染时选择性地剪除每个高斯基元投影到 2D 图像上的部分片段（像素），以减少光栅化阶段的计算量，特别适用于加速边缘设备上的渲染 20。  
* **量化与编码**:  
  * **向量量化 (Vector Quantization, VQ)**: 使用码本 (Codebook) 来压缩高斯基元的属性（如几何参数或颜色特征）。每个基元不再存储完整的参数值，而是存储指向码本中最近向量的索引 19。例如，CompGS 23 和 Compact-3DGS 57 都采用了 VQ。LightGaussian 19 也是一种相关技术。  
  * **参数排序与 2D 网格化 (Parameter Sorting & Gridding)**: 如 Self-Organizing Gaussians 25 提出一种方法，在训练过程中周期性地将高维高斯参数排序并映射到一个 2D 网格中，使得相邻网格单元的参数具有局部相似性。这种结构化的 2D 数据可以利用现成的图像或视频压缩算法（如 JPEG XL, AVIF, VVC）进行高效压缩，实现极高的压缩率。  
  * **残差补偿压缩 (Residual Compensation Compression)**: 如 HiFi4G 63 中用于动态人体性能捕捉的方法，采用了标准的残差补偿、量化和熵编码流程来压缩随时间变化的 4D 高斯参数。  
  * **其他编码**: Huffman 编码等无损熵编码方法常被用作量化后的最后一步，以进一步压缩索引或量化值 57。

### **7.4 高斯表示的修改**

这类方法通过改变 3D 高斯基元本身的定义或与其他表示方法结合，来寻求效率、质量或功能的提升。

* **简化高斯**:  
  * **各向同性高斯 (Isotropic Gaussians)**: 提出使用计算上更简单的各向同性高斯（球形）代替各向异性高斯（椭球形），以加速分裂、合并等操作，尽管可能牺牲部分表达能力 58。  
* **改进属性表示**:  
  * **基于网格的神经场颜色 (Grid-Based Neural Fields for Color)**: 如 Compact-3DGS 57 使用紧凑的基于哈希网格的神经场来代替球谐函数 (SH) 表示视角相关颜色，以减少存储开销。  
  * **替代外观模型**: 如 GaussianShader 8 引入了基于物理的着色函数来更好地模拟反射表面。LE3D 22 使用 Color MLP 来表示 RAW 图像的线性颜色空间，以克服 SH 对 HDR 信息的有限表达能力。NexusSplats 64 采用中心化的外观学习策略，通过 Nexus 核来协调一组高斯基元的颜色映射，以高效解耦光照变化并保证局部颜色一致性。  
* **混合表示 (Hybrid Representations)**:  
  * **高斯 \+ 神经场 (Gaussians \+ Neural Fields)**: 如 4DGF 59 使用 3D 高斯作为高效的几何支架，但将外观信息存储在更紧凑灵活的神经场中，以减少内存占用并更好地处理动态和异构数据。  
  * **高斯 \+ SDF (Gaussians \+ SDF)**: 如 GSDF 40 结合 3DGS 和 SDF，利用各自优势提升渲染和几何重建。  
  * **Triplane \+ 高斯 (Triplane \+ Gaussians)**: 如 TGS 66 结合了 Triplane 表示和高斯溅射，用于快速的单视图三维重建。  
* **几何正则化**:  
  * **有效秩正则化 (Effective Rank Regularization)**: 提出使用协方差矩阵的有效秩作为正则项，约束高斯基元的形状，避免其退化成“针状”，从而改善法线和几何重建质量 9。

**表 2: 高效 3DGS 相关研究技术分类**

| 类别 | 代表性方法/技术 | 核心思想 | 主要优势 | 相关文献 |
| :---- | :---- | :---- | :---- | :---- |
| **1\. 初始化与致密化策略** | EDGS | 稠密对应三角化，消除致密化 | 极快收敛，基元少，质量高 | 21 |
|  | Pixel-Error Driven Densification | 基于像素误差指导致密化 | 更符合感知，处理纹理，控制更好 | 36 |
|  | Pixel-GS | 像素面积加权梯度 | 改善稀疏区域致密化，减少模糊 | 32 |
|  | AtomGS | 原子高斯优先增殖+融合 | 提升几何精度 | 37 |
|  | MCMC-based Densification | MCMC 状态转移取代启发式规则 | 对初始化更鲁棒 | 44 |
| **2\. 优化与收敛策略** | DashGaussian | 动态调度渲染分辨率/基元数 | 加速多种 3DGS 骨干网络收敛 | 16 |
|  | RadSplat (NeRF Prior) | NeRF 指导优化与剪枝 | 提高复杂场景鲁棒性与质量 | 17 |
|  | GSDF (SDF Guidance) | 3DGS 与 SDF 联合优化，相互指导 | 提升几何精度与渲染质量 | 40 |
|  | Coarse-to-Fine / Frequency Regularization | 渐进式优化（分辨率/频率） | 加速收敛，移除漂浮物 | 8 |
| **3\. 剪枝、稀疏化与量化** | Fragment Pruning | 剪除高斯内的冗余像素片段 | 加速光栅化，提升边缘设备 FPS | 20 |
|  | Vector Quantization (VQ) | 使用码本压缩高斯属性 | 大幅减小模型存储体积 | 23 |
|  | Self-Organizing Gaussians | 参数排序+2D网格化+图像压缩 | 极高压缩率，保持渲染质量 | 25 |
|  | Learnable Masking / Opacity Regularization | 学习移除冗余基元/鼓励稀疏 | 减少基元数量 | 23 |
| **4\. 高斯表示的修改** | Isotropic Gaussians | 使用计算更简单的球形高斯 | 加速几何操作 | 58 |
|  | Hybrid Representations (GS+Neural Field/SDF/Triplane) | 结合隐式表示的优势 | 提升紧凑性、几何精度或特定任务性能 | 40 |
|  | Alternative Appearance Models | 替换 SH (MLP/PBR Shading/Nexus Kernels) | 更好模拟 HDR/反射/光照变化，更紧凑 | 8 |
|  | Effective Rank Regularization | 约束协方差形状 | 减少针状伪影，改善几何 | 9 |

这张表格清晰地展示了当前高效 3DGS 研究的多样性。研究者们正从初始化、优化过程、模型表示、压缩存储等几乎所有环节入手，寻求突破。一个重要的观察是，许多技术是正交的或互补的。例如，像 EDGS 这样的初始化改进理论上可以与优化调度、片段剪枝以及最终的量化压缩相结合。另一个趋势是越来越多地将纯粹显式的高斯表示与隐式的神经表示（如神经场或 SDF）相结合，试图取长补短，在保持 3DGS 渲染效率的同时，利用隐式表示在连续性、紧凑性或几何约束方面的优势。

## **8\. 结论：高效 3DGS 研究现状与未来展望**

### **8.1 当前研究格局总结**

三维高斯溅射 (3DGS) 自提出以来，凭借其出色的实时渲染能力和高质量的视觉效果，已迅速成为三维场景表示和新视角合成领域的主流技术之一。然而，其在训练效率、内存占用和模型存储方面的挑战也随之凸显，催生了大量以提升效率为目标的研究工作。

当前，高效 3DGS 的研究呈现出多元化的格局：

* **改进或替代初始化与致密化**: 认识到标准 ADC 的启发式本质和效率瓶颈，研究者们提出了多种方案。以 EDGS 为代表的方法试图通过高质量的密集初始化彻底消除迭代致密化，从根本上改变优化范式。其他方法则致力于改进 ADC 的触发条件（如基于像素误差或覆盖面积）或执行方式（如 MCMC、原子高斯），使其更加智能和鲁棒。  
* **优化策略与训练加速**: 通过改进优化算法本身（如引入先验指导）、设计更高效的训练调度（如动态调整分辨率和基元数量）来缩短训练时间。  
* **模型压缩与紧凑表示**: 针对 3DGS 模型体积庞大的问题，研究者们开发了各种剪枝、稀疏化、量化和编码技术，旨在显著降低存储需求，同时尽量保持渲染质量和速度。将高斯参数组织成结构化数据（如 2D 网格）以便利用现有压缩工具是一个新兴方向。  
* **高斯表示的演进**: 不满足于标准的高斯基元定义，研究工作开始探索修改其属性表示（如用神经场替代 SH 系数）、简化其几何（如各向同性高斯）、或将其与其它三维表示（如 SDF、Triplane）相结合，以寻求在效率、几何精度或特定功能上的突破。  
* **渲染效率提升**: 除了减少基元数量或压缩参数外，还有直接针对渲染管线优化的工作，如片段剪枝，旨在降低光栅化阶段的计算负载。

### **8.2 EDGS 的贡献与地位**

在众多效率优化方法中，EDGS 占据了一个独特且重要的位置。

* **核心贡献**: EDGS 的核心贡献在于提出了一种全新的思路——**通过基于稠密对应的三角化进行一次性密集初始化，彻底消除了标准 3DGS 中缓慢且迭代的致密化过程** 21。  
* **显著影响**: 实验结果表明，EDGS 能够实现**数量级的收敛速度提升**（约 10 倍），同时获得**更高的渲染质量**，并且使用的**高斯基元数量更少**（约一半） 21。这种在速度、质量和紧凑性三方面同时取得显著改进的成果，使其成为一项极具吸引力的技术。  
* **范式转变**: EDGS 代表了从“被动式迭代精化”到“主动式全局近似”的范式转变。它证明了利用现代稠密匹配技术的强大能力，可以在优化开始前就构建出足够好的场景几何先验，从而绕过效率低下的动态几何发现过程。  
* **基础性与兼容性**: 由于 EDGS 主要作用于初始化阶段，它与其他专注于优化、渲染或压缩阶段的技术具有良好的**兼容性** 21。这使得 EDGS 有潜力成为未来高效 3DGS 系统的一个基础模块，与其他技术叠加使用以实现更极致的性能。  
* **潜在局限**: EDGS 的性能高度依赖于上游稠密对应和三角化步骤的质量。在对应困难的区域（如无纹理、遮挡严重），其效果可能会受到影响。

总而言之，EDGS 以其简洁而有效的方法，为解决 3DGS 的核心效率瓶颈提供了一个强有力的解决方案，并在效率和质量上树立了新的标杆。

### **8.3 未来的挑战与研究方向**

尽管高效 3DGS 研究已取得长足进步，但仍面临诸多挑战，同时也孕育着丰富的未来研究机遇：

1. **极致效率的追求**: 能否将训练时间进一步压缩到秒级甚至亚秒级？能否实现数十倍乃至百倍的模型压缩率而不损失质量？探索不同效率优化技术的协同效应（如 EDGS \+ DashGaussian \+ VQ \+ Fragment Pruning）将是重要方向。  
2. **鲁棒性与泛化性**: 如何让高效 3DGS 方法在更具挑战性的数据上（如**极度稀疏的输入视图** 12、**低质量或噪声图像**、**剧烈变化的光照** 64）保持高性能？如何处理 SfM 失败或不准确的情况 22？提高模型对不同场景类型和数据质量的泛化能力至关重要。  
3. **动态场景 (4DGS) 的效率**: 将 EDGS 等高效静态场景方法扩展到处理**动态场景**是一个活跃的研究领域 12。如何高效地初始化和优化随时间变化的几何与外观？如何处理快速运动区域的初始化难题 35？如何实现高效的 4D 数据流式传输与渲染 67？  
4. **大规模场景处理**: 如何将高效 3DGS 技术**扩展到城市级别甚至更大范围的场景** 14？这需要解决 GPU 内存限制 27、分布式训练、场景分解与合并等问题。  
5. **几何精度与质量**: 在追求渲染效率的同时，如何进一步**提升重建的几何准确性** 9，减少漂浮物、针状体等渲染伪影 8？结合隐式几何表示（如 SDF）是一个有前景的方向 40。  
6. **理论基础深化**: 目前许多高效 3DGS 技术仍带有一定的启发性。需要更深入地理解各种优化策略的**理论基础、收敛保证以及不同方法间的根本联系与权衡**。  
7. **下游应用集成**: 如何将高效 3DGS 模型更无缝地**集成到实际应用**中，如实时 SLAM 14、场景编辑与交互 13、物理仿真 13、机器人导航与操作 14 以及 AR/VR 体验 23？这需要考虑实时性、内存占用、易用性等多方面因素。  
8. **多模态融合**: 探索融合除 RGB 图像外的其他模态信息（如 LiDAR、事件相机 70、语义标签 14）来辅助高效和鲁棒的 3DGS 重建。

未来的研究很可能会朝着结合多种优化策略的方向发展。例如，利用 EDGS 或类似方法进行高质量的快速初始化，结合智能的优化调度和正则化技术（如 SDF 指导或频率控制）进行训练，最后应用先进的量化和剪枝技术（如片段剪枝和 VQ）来生成极其紧凑且渲染高效的模型。同时，需要认识到“效率”本身是多维度的——训练速度、渲染速度、内存占用、存储大小等，不同的应用场景对效率的侧重点不同，因此需要根据具体需求选择或组合最合适的技术。EDGS 作为初始化阶段的重大突破，无疑将在未来高效 3DGS 的发展中扮演关键角色。

#### **Works cited**

1. Improving Adaptive Density Control for 3D Gaussian Splatting \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2503.14274v1](https://arxiv.org/html/2503.14274v1)  
2. \[2401.03890\] A Survey on 3D Gaussian Splatting \- arXiv, accessed April 23, 2025, [https://arxiv.org/abs/2401.03890](https://arxiv.org/abs/2401.03890)  
3. A Survey on 3D Gaussian Splatting \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2401.03890v2](https://arxiv.org/html/2401.03890v2)  
4. 3D Gaussian Splatting for Real-Time Radiance Field Rendering \- Inria, accessed April 23, 2025, [https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/)  
5. (PDF) 3D Gaussian Splatting for Real-Time Radiance Field Rendering \- ResearchGate, accessed April 23, 2025, [https://www.researchgate.net/publication/372989904\_3D\_Gaussian\_Splatting\_for\_Real-Time\_Radiance\_Field\_Rendering](https://www.researchgate.net/publication/372989904_3D_Gaussian_Splatting_for_Real-Time_Radiance_Field_Rendering)  
6. 3D Gaussian Splatting for Real-Time Radiance Field Rendering \- Hugging Face, accessed April 23, 2025, [https://huggingface.co/papers/2308.04079](https://huggingface.co/papers/2308.04079)  
7. \[2308.04079\] 3D Gaussian Splatting for Real-Time Radiance Field Rendering \- arXiv, accessed April 23, 2025, [https://arxiv.org/abs/2308.04079](https://arxiv.org/abs/2308.04079)  
8. NeurIPS Poster Spec-Gaussian: Anisotropic View-Dependent Appearance for 3D Gaussian Splatting, accessed April 23, 2025, [https://nips.cc/virtual/2024/poster/93509](https://nips.cc/virtual/2024/poster/93509)  
9. NeurIPS Poster Effective Rank Analysis and Regularization for Enhanced 3D Gaussian Splatting, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/96009](https://neurips.cc/virtual/2024/poster/96009)  
10. ODGS: 3D Scene Reconstruction from Omnidirectional Images with 3D Gaussian Splatting \- NIPS papers, accessed April 23, 2025, [https://proceedings.neurips.cc/paper\_files/paper/2024/file/6882dbdc34bcd094e6f858c06ce30edb-Paper-Conference.pdf](https://proceedings.neurips.cc/paper_files/paper/2024/file/6882dbdc34bcd094e6f858c06ce30edb-Paper-Conference.pdf)  
11. yangjiheng/3DGS\_and\_Beyond\_Docs: This is a collective repository for all 3DGS related progresses in research and industry world \- GitHub, accessed April 23, 2025, [https://github.com/yangjiheng/3DGS\_and\_Beyond\_Docs](https://github.com/yangjiheng/3DGS_and_Beyond_Docs)  
12. SplatFields: Neural Gaussian Splats for Sparse 3D and 4D Reconstruction \- European Computer Vision Association, accessed April 23, 2025, [https://www.ecva.net/papers/eccv\_2024/papers\_ECCV/papers/00254.pdf](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/00254.pdf)  
13. 3D Gaussian Splatting: Survey, Technologies, Challenges, and Opportunities \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2407.17418v2](https://arxiv.org/html/2407.17418v2)  
14. zstsandy/Awesome-3D-Gaussian-Splatting-in-Robotics \- GitHub, accessed April 23, 2025, [https://github.com/zstsandy/Awesome-3D-Gaussian-Splatting-in-Robotics](https://github.com/zstsandy/Awesome-3D-Gaussian-Splatting-in-Robotics)  
15. dtc111111/awesome-3dgs-for-robotics \- GitHub, accessed April 23, 2025, [https://github.com/dtc111111/awesome-3dgs-for-robotics](https://github.com/dtc111111/awesome-3dgs-for-robotics)  
16. DashGaussian: Optimizing 3D Gaussian Splatting in 200 Seconds \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2503.18402v2](https://arxiv.org/html/2503.18402v2)  
17. RadSplat: Radiance Field-Informed Gaussian Splatting for Robust Real-Time Rendering with 900+ FPS \- Michael Niemeyer, accessed April 23, 2025, [https://m-niemeyer.github.io/radsplat/static/pdf/niemeyer2024radsplat.pdf](https://m-niemeyer.github.io/radsplat/static/pdf/niemeyer2024radsplat.pdf)  
18. RadSplat: Radiance Field-Informed Gaussian Splatting for Robust Real-Time Rendering with 900+ FPS \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2403.13806v1](https://arxiv.org/html/2403.13806v1)  
19. 3D Gaussian Rendering Can Be Sparser: Efficient Rendering via Learned Fragment Pruning | OpenReview, accessed April 23, 2025, [https://openreview.net/forum?id=IVqzbuLfoL\&referrer=%5Bthe%20profile%20of%20Zhifan%20Ye%5D(%2Fprofile%3Fid%3D\~Zhifan\_Ye1)](https://openreview.net/forum?id=IVqzbuLfoL&referrer=%5Bthe+profile+of+Zhifan+Ye%5D\(/profile?id%3D~Zhifan_Ye1\))  
20. NeurIPS Poster 3D Gaussian Rendering Can Be Sparser: Efficient ..., accessed April 23, 2025, [https://nips.cc/virtual/2024/poster/95764](https://nips.cc/virtual/2024/poster/95764)  
21. EDGS: Eliminating Densification for Efficient Convergence of 3DGS \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2504.13204v1](https://arxiv.org/html/2504.13204v1)  
22. Lighting Every Darkness with 3DGS: Fast Training and Real-Time Rendering for HDR View Synthesis \- NIPS papers, accessed April 23, 2025, [https://proceedings.neurips.cc/paper\_files/paper/2024/file/92dd1adab39f362046f99dfe3c39d90f-Paper-Conference.pdf](https://proceedings.neurips.cc/paper_files/paper/2024/file/92dd1adab39f362046f99dfe3c39d90f-Paper-Conference.pdf)  
23. CompGS: Smaller and Faster Gaussian Splatting with Vector Quantization \- Computer Science | UC Davis Engineering, accessed April 23, 2025, [https://web.cs.ucdavis.edu/\~hpirsiav/papers/CompGS\_eccv24.pdf](https://web.cs.ucdavis.edu/~hpirsiav/papers/CompGS_eccv24.pdf)  
24. (PDF) Optimized 3D Gaussian Splatting using Coarse-to-Fine Image Frequency Modulation, accessed April 23, 2025, [https://www.researchgate.net/publication/389947368\_Optimized\_3D\_Gaussian\_Splatting\_using\_Coarse-to-Fine\_Image\_Frequency\_Modulation](https://www.researchgate.net/publication/389947368_Optimized_3D_Gaussian_Splatting_using_Coarse-to-Fine_Image_Frequency_Modulation)  
25. Compact 3D Scene Representation via Self-Organizing Gaussian Grids \- European Computer Vision Association, accessed April 23, 2025, [https://www.ecva.net/papers/eccv\_2024/papers\_ECCV/papers/11348.pdf](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/11348.pdf)  
26. Trimming the Fat: Efficient Compression of 3D Gaussian Splats through Pruning \- BMVA Archive, accessed April 23, 2025, [https://bmva-archive.org.uk/bmvc/2024/papers/Paper\_358/paper.pdf](https://bmva-archive.org.uk/bmvc/2024/papers/Paper_358/paper.pdf)  
27. GaussianFocus: Constrained Attention Focus for 3D Gaussian Splatting | OpenReview, accessed April 23, 2025, [https://openreview.net/forum?id=LieTse3fQB](https://openreview.net/forum?id=LieTse3fQB)  
28. \[2504.13204\] EDGS: Eliminating Densification for Efficient Convergence of 3DGS \- arXiv, accessed April 23, 2025, [https://arxiv.org/abs/2504.13204](https://arxiv.org/abs/2504.13204)  
29. EDGS: Eliminating Densification for Efficient Convergence of 3DGS \- GitHub Pages, accessed April 23, 2025, [https://compvis.github.io/EDGS/](https://compvis.github.io/EDGS/)  
30. Graphics \- arXiv, accessed April 23, 2025, [https://arxiv.org/list/cs.GR/recent](https://arxiv.org/list/cs.GR/recent)  
31. 3D Gaussian Splatting: Survey, Technologies, Challenges, and Opportunities \- arXiv, accessed April 23, 2025, [https://www.arxiv.org/pdf/2407.17418](https://www.arxiv.org/pdf/2407.17418)  
32. www.ecva.net, accessed April 23, 2025, [https://www.ecva.net/papers/eccv\_2024/papers\_ECCV/papers/02926.pdf](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/02926.pdf)  
33. NeurIPS Poster Lighting Every Darkness with 3DGS: Fast Training and Real-Time Rendering for HDR View Synthesis, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/96520](https://neurips.cc/virtual/2024/poster/96520)  
34. NeurIPS Poster Structure Consistent Gaussian Splatting with Matching Prior for Few-shot Novel View Synthesis, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/94282](https://neurips.cc/virtual/2024/poster/94282)  
35. NeurIPS Poster 4D Gaussian Splatting in the Wild with Uncertainty-Aware Regularization, accessed April 23, 2025, [https://nips.cc/virtual/2024/poster/96899](https://nips.cc/virtual/2024/poster/96899)  
36. www.ecva.net, accessed April 23, 2025, [https://www.ecva.net/papers/eccv\_2024/papers\_ECCV/papers/08041.pdf](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/08041.pdf)  
37. AtomGS: Atomizing Gaussian Splatting for High-Fidelity Radiance Field \- BMVA Archive, accessed April 23, 2025, [https://bmva-archive.org.uk/bmvc/2024/papers/Paper\_577/paper.pdf](https://bmva-archive.org.uk/bmvc/2024/papers/Paper_577/paper.pdf)  
38. NeurIPS Poster 3DGS-Enhancer: Enhancing Unbounded 3D Gaussian Splatting with View-consistent 2D Diffusion Priors, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/95333](https://neurips.cc/virtual/2024/poster/95333)  
39. CoherentGS: Sparse Novel View Synthesis with Coherent 3D Gaussians \- European Computer Vision Association, accessed April 23, 2025, [https://www.ecva.net/papers/eccv\_2024/papers\_ECCV/papers/04306.pdf](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/04306.pdf)  
40. NeurIPS Poster GSDF: 3DGS Meets SDF for Improved Neural ..., accessed April 23, 2025, [https://nips.cc/virtual/2024/poster/93457](https://nips.cc/virtual/2024/poster/93457)  
41. NeurIPS Poster ReGS: Reference-based Controllable Scene Stylization with Gaussian Splatting, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/92993](https://neurips.cc/virtual/2024/poster/92993)  
42. www.scitepress.org, accessed April 23, 2025, [https://www.scitepress.org/Papers/2025/133085/133085.pdf](https://www.scitepress.org/Papers/2025/133085/133085.pdf)  
43. Revising Densification in Gaussian Splatting \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2404.06109v1](https://arxiv.org/html/2404.06109v1)  
44. 3D Gaussian Splatting as Markov Chain Monte Carlo \- GitHub Pages, accessed April 23, 2025, [https://ubc-vision.github.io/3dgs-mcmc/paper.pdf](https://ubc-vision.github.io/3dgs-mcmc/paper.pdf)  
45. NeurIPS Poster Gaussian Graph Network: Learning Efficient and Generalizable Gaussian Representations from Multi-view Images, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/96803](https://neurips.cc/virtual/2024/poster/96803)  
46. RadSplat: Radiance Field-Informed Gaussian Splatting for Robust Real-Time Rendering with 900+ FPS \- Hugging Face, accessed April 23, 2025, [https://huggingface.co/papers/2403.13806](https://huggingface.co/papers/2403.13806)  
47. TRIANGULATION FOR SURFACE MODELLING \- CiteSeerX, accessed April 23, 2025, [https://citeseerx.ist.psu.edu/document?repid=rep1\&type=pdf\&doi=f8f6af71287d891d882e03550aaf6282d903557b](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=f8f6af71287d891d882e03550aaf6282d903557b)  
48. Image-Consistent Surface Triangulation \- Carnegie Mellon University Robotics Institute, accessed April 23, 2025, [https://www.ri.cmu.edu/pub\_files/pub2/morris\_daniel\_2000\_1/morris\_daniel\_2000\_1.pdf](https://www.ri.cmu.edu/pub_files/pub2/morris_daniel_2000_1/morris_daniel_2000_1.pdf)  
49. Quantitative Metrics (PSNR, SSIM, LPIPS) of 3DGS Method \- ResearchGate, accessed April 23, 2025, [https://www.researchgate.net/figure/Quantitative-Metrics-PSNR-SSIM-LPIPS-of-3DGS-Method\_tbl2\_390439943](https://www.researchgate.net/figure/Quantitative-Metrics-PSNR-SSIM-LPIPS-of-3DGS-Method_tbl2_390439943)  
50. Unleashing the Power of One-Step Diffusion based Image Super-Resolution via a Large-Scale Diffusion Discriminator \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2410.04224v3](https://arxiv.org/html/2410.04224v3)  
51. S3IM: Stochastic Structural SIMilarity and Its Unreasonable Effectiveness for Neural Fields \- CVF Open Access, accessed April 23, 2025, [https://openaccess.thecvf.com/content/ICCV2023/papers/Xie\_S3IM\_Stochastic\_Structural\_SIMilarity\_and\_Its\_Unreasonable\_Effectiveness\_for\_Neural\_ICCV\_2023\_paper.pdf](https://openaccess.thecvf.com/content/ICCV2023/papers/Xie_S3IM_Stochastic_Structural_SIMilarity_and_Its_Unreasonable_Effectiveness_for_Neural_ICCV_2023_paper.pdf)  
52. A comprehensive review of image super-resolution metrics: classical and AI-based approaches \- Acta IMEKO, accessed April 23, 2025, [https://acta.imeko.org/index.php/acta-imeko/article/view/1679/2939](https://acta.imeko.org/index.php/acta-imeko/article/view/1679/2939)  
53. Quantitative comparison using the LOL dataset in terms of PSNR, SSIM \- ResearchGate, accessed April 23, 2025, [https://www.researchgate.net/figure/Quantitative-comparison-using-the-LOL-dataset-in-terms-of-PSNR-SSIM-and-LPIPS-the-best\_tbl1\_372554686](https://www.researchgate.net/figure/Quantitative-comparison-using-the-LOL-dataset-in-terms-of-PSNR-SSIM-and-LPIPS-the-best_tbl1_372554686)  
54. A Real-World Benchmark for Sentinel-2 Multi-Image Super-Resolution \- PMC, accessed April 23, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10514337/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10514337/)  
55. Image Quality Assessment through FSIM, SSIM, MSE and PSNR—A Comparative Study, accessed April 23, 2025, [https://www.scirp.org/journal/paperinformation?paperid=90911](https://www.scirp.org/journal/paperinformation?paperid=90911)  
56. Boundary-aware Decoupled Flow Networks for Realistic Extreme Rescaling \- IJCAI, accessed April 23, 2025, [https://www.ijcai.org/proceedings/2024/0110.pdf](https://www.ijcai.org/proceedings/2024/0110.pdf)  
57. maincold2/Compact-3DGS: The official repository of ... \- GitHub, accessed April 23, 2025, [https://github.com/maincold2/Compact-3DGS](https://github.com/maincold2/Compact-3DGS)  
58. \[2403.14244\] Isotropic Gaussian Splatting for Real-Time Radiance Field Rendering \- arXiv, accessed April 23, 2025, [https://arxiv.org/abs/2403.14244](https://arxiv.org/abs/2403.14244)  
59. NeurIPS Poster Dynamic 3D Gaussian Fields for Urban Areas, accessed April 23, 2025, [https://neurips.cc/virtual/2024/poster/93077](https://neurips.cc/virtual/2024/poster/93077)  
60. RadSplat: Radiance Field-Informed Gaussian Splatting for Robust ..., accessed April 23, 2025, [https://m-niemeyer.github.io/radsplat/](https://m-niemeyer.github.io/radsplat/)  
61. Compact 3D Scene Representation via Self-Organizing Gaussian Grids \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2312.13299](https://arxiv.org/html/2312.13299)  
62. Compact 3D Scene Representation via Self-Organizing Gaussian Grids \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2312.13299v1](https://arxiv.org/html/2312.13299v1)  
63. CVPR Poster HiFi4G: High-Fidelity Human Performance Rendering via Compact Gaussian Splatting, accessed April 23, 2025, [https://cvpr.thecvf.com/virtual/2024/poster/30570](https://cvpr.thecvf.com/virtual/2024/poster/30570)  
64. NexusSplats: Efficient 3D Gaussian Splatting in the Wild \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2411.14514v5](https://arxiv.org/html/2411.14514v5)  
65. NexusSplats: Efficient 3D Gaussian Splatting in the Wild \- arXiv, accessed April 23, 2025, [https://arxiv.org/html/2411.14514v4](https://arxiv.org/html/2411.14514v4)  
66. Triplane Meets Gaussian Splatting: Fast and Generalizable Single-View 3D Reconstruction with Transformers \- CVF Open Access, accessed April 23, 2025, [https://openaccess.thecvf.com/content/CVPR2024/papers/Zou\_Triplane\_Meets\_Gaussian\_Splatting\_Fast\_and\_Generalizable\_Single-View\_3D\_Reconstruction\_CVPR\_2024\_paper.pdf](https://openaccess.thecvf.com/content/CVPR2024/papers/Zou_Triplane_Meets_Gaussian_Splatting_Fast_and_Generalizable_Single-View_3D_Reconstruction_CVPR_2024_paper.pdf)  
67. QUEEN: QUantized Efficient ENcoding of Dynamic Gaussians for Streaming Free-viewpoint Videos \- NIPS papers, accessed April 23, 2025, [https://proceedings.neurips.cc/paper\_files/paper/2024/file/4c9477b9e2c7ec0ad3f4f15077aaf85a-Paper-Conference.pdf](https://proceedings.neurips.cc/paper_files/paper/2024/file/4c9477b9e2c7ec0ad3f4f15077aaf85a-Paper-Conference.pdf)  
68. 3DGStream: On-the-Fly Training of 3D Gaussians for Efficient Streaming of Photo-Realistic Free-Viewpoint Videos, accessed April 23, 2025, [https://openaccess.thecvf.com/content/CVPR2024/papers/Sun\_3DGStream\_On-the-Fly\_Training\_of\_3D\_Gaussians\_for\_Efficient\_Streaming\_of\_CVPR\_2024\_paper.pdf](https://openaccess.thecvf.com/content/CVPR2024/papers/Sun_3DGStream_On-the-Fly_Training_of_3D_Gaussians_for_Efficient_Streaming_of_CVPR_2024_paper.pdf)  
69. Enhanced 3-D Urban Scene Reconstruction and Point Cloud Densification Using Gaussian Splatting and Google Earth Imagery \- University of Waterloo, accessed April 23, 2025, [https://uwaterloo.ca/geospatial-intelligence/sites/default/files/uploads/documents/enhanced\_3-d\_urban\_scene\_reconstruction\_and\_point\_cloud\_densification\_using\_gaussian\_splatting\_and\_google\_earth\_imagery.pdf](https://uwaterloo.ca/geospatial-intelligence/sites/default/files/uploads/documents/enhanced_3-d_urban_scene_reconstruction_and_point_cloud_densification_using_gaussian_splatting_and_google_earth_imagery.pdf)  
70. Lee-JaeWon/2024-Arxiv-Paper-List-Gaussian-Splatting \- GitHub, accessed April 23, 2025, [https://github.com/Lee-JaeWon/2024-Arxiv-Paper-List-Gaussian-Splatting](https://github.com/Lee-JaeWon/2024-Arxiv-Paper-List-Gaussian-Splatting)