# A.RobotAI-LGE-ROS2-LifecycleManager
A C++ deterministic lifecycle orchestrator optimized for ROS 2 embedded systems.
>
> 🚧 **[NOTICE] Project Status**
>
> This repository provides architectural documentation, YAML-based configuration
> examples, and empirical validation results for the Deterministic Lifecycle Manager.
>
> The execution model, lifecycle semantics, dependency resolution strategy, and
> performance characteristics are fully documented here to allow technical review
> and reproduction of system behavior.
>
> The complete C++ source code is planned to be released under the Apache License 2.0
> following internal corporate compliance procedures.

A production-grade, C++ deterministic lifecycle orchestrator optimized for resource-constrained ROS 2 embedded systems.
## 📑 Table of Contents
1. [Overview - "What & Why?"](#1-overview---what--why)
2. [Structural Pain Points in Production Systems](#2-structural-pain-points-in-production-systems)
3. [Architecture & LifecycleManager](#3-architecture--lifecyclemanager)
4. [Deterministic Boot Flow (Deep-dive)](#4-deterministic-boot-flow-deep-dive)
5. [Metrics & Validation](#5-metrics--validation)
6. [Source, Build & License](#6-source-build--license)
---
## 1. Overview - "What & Why?"
This document introduces the **"Deterministic Lifecycle Manager"**, a C++‑based lifecycle orchestration service designed to run ROS 2 reliably on low‑end embedded robotic platforms.
The target systems are cost-constrained commercial robotic platforms built on entry-level SoCs such as **Rockchip PX30, LG DQ1**, or other Cortex-A35-class CPUs, typically equipped with **less than 1GB of RAM**. On these platforms, boot-time behavior and peak resource usage critically impact system stability.
### 🔴 Motivation
During system integration and production‑equivalent platform evaluation, we repeatedly encountered critical issues when using the standard Python‑based `ros2 launch` workflow on resource‑constrained hardware.
The most common problems were:
* High baseline RAM usage before application logic starts
* Large CPU spikes during parallel node startup
* Frequent OOM (Out Of Memory) kills during early boot
* Unstable and non-deterministic startup sequences in production images
These issues were consistently reproducible across multiple low-end SoCs and robot products. They could not be resolved in a reliable way through parameter tuning, launch configuration changes, or partial optimization. 
**Our conclusion** was that the problems originated from the runtime overhead and process model of the Python-based launch system itself, which does not scale well under tight CPU and memory constraints.
### 🟢 Design Approach
To address these limitations, we implemented the **Deterministic Lifecycle Manager** as a minimal, native C++ service that launches and supervises ROS 2 nodes directly as OS-level processes.
Key characteristics include:
* **Zero Python dependency** on the target system
* ROS 2 nodes built and executed as native C++ binaries
* **Fast and Safe Concurrent Spawning:** Utilizes C++ threads alongside standard POSIX primitives (`fork()`, `execvp()`, `SIGCHLD`) to achieve true parallel execution without the CPU bottlenecks typical of Python.
This design preserves existing ROS 2 components—including DDS communication and lifecycle semantics—while eliminating the runtime overhead introduced by Python-based launch tooling.
> **💡 Note:** Deterministic Lifecycle Manager is *not* a replacement for ROS 2. It is a focused orchestration layer that provides deterministic and resource-efficient boot and lifecycle management, specifically tailored for low-end embedded systems used in cost-constrained production robots.
## 2. Why Python launch breaks on low-end SoCs - "Pain Point"
### Structural Pain Points in Production Systems
This section explains why the standard Python‑based `ros2 launch` workflow becomes a reliability bottleneck on low‑end SoCs, based on issues repeatedly observed during production‑equivalent system integration.
#### 2.1 Excessive Runtime Overhead During Boot
`ros2 launch` relies on Python processes that are loaded and initialized during system boot.
* On low‑end platforms with limited CPU performance and less than 1 GB of RAM, this introduces **substantial overhead before any application logic starts**.
* As the number of nodes increases, Python interpreter initialization and runtime management consume a significant portion of system resources, frequently leading to **memory pressure and OOM (Out Of Memory) events during early boot**.
#### 2.2 Non‑Deterministic Startup Under CPU Contention
* On entry‑level CPUs (such as Cortex‑A35‑class cores), parallel startup of multiple Python processes causes **severe CPU contention** during boot.
* Although `ros2 launch` supports parallel spawning, it does not enforce strict OS‑level startup ordering or readiness guarantees.
* As a result, dependent nodes may start before prerequisite nodes are fully initialized, leading to **race conditions and unstable boot behavior** in production environments.
#### 2.3 Fragmented Lifecycle Control After Launch
The launch system focuses on process creation and parameter loading.
* After startup, lifecycle state transitions are not centrally coordinated.
* In systems managing many nodes, this results in fragmented lifecycle handling, **uncoordinated state transitions**, and the absence of a single authoritative component responsible for global system state—an important weakness on resource‑constrained platforms.
#### 2.4 Mismatch Between Robot Missions and Lifecycle Semantics
ROS 2 lifecycle states are intentionally minimal and low‑level, while production robots operate in mission‑level modes such as standby or navigation.
* Without centralized orchestration, mapping mission‑level behavior to coordinated lifecycle transitions across multiple nodes becomes **error‑prone and difficult to validate**, especially on low‑end SoCs where deterministic behavior is critical.
> **💡 Summary**  
> On low‑end embedded platforms, the Python‑based launch system introduces overhead and non‑determinism during boot and state transitions. **These limitations are structural and cannot be reliably mitigated through launch configuration alone**, motivating a native and deterministic lifecycle orchestration approach.
