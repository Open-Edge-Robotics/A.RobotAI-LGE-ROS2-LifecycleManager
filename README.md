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
This document introduces "Deterministic Lifecycle Manager", a C++‑based lifecycle orchestration service designed to run ROS 2 reliably on low‑end embedded robotic platforms.

The target systems are cost‑constrained commercial robotic platforms built on entry‑level SoCs such as Rockchip PX30, LG DQ1, or other Cortex‑A35‑class CPUs, typically equipped with less than 1 GB of RAM.

On these platforms, boot‑time behavior and peak resource usage critically impact system stability.

Motivation
During system integration and production‑equivalent platform evaluation, we repeatedly encountered critical issues when using the standard Python‑based ros2 launch workflow on resource‑constrained hardware.

The most common problems were:

high baseline RAM usage before application logic starts,
large CPU spikes during parallel node startup,
frequent OOM kills during early boot,
unstable and non‑deterministic startup sequences in production images.
These issues were consistently reproducible across multiple low‑end SoCs and robot products.

They could not be resolved in a reliable way through parameter tuning, launch configuration changes, or partial optimization.

Our conclusion was that the problems originated from the runtime overhead and process model of the Python‑based launch system itself, which does not scale well under tight CPU and memory constraints.

Design Approach
To address these limitations, we implemented Deterministic Lifecycle Manager as a minimal, native C++ service that launches and supervises ROS 2 nodes directly as OS‑level processes.

Key characteristics of Deterministic Lifecycle Manager include:

no Python dependency on the target system,
ROS 2 nodes built and executed as native C++ binaries,
direct process control using standard POSIX primitives such as fork(), execvp(), and SIGCHLD.
This design preserves existing ROS 2 components—including DDS communication and lifecycle semantics—while eliminating the runtime overhead introduced by Python‑based launch tooling.

Deterministic Lifecycle Manager is not a replacement for ROS 2.

It is a focused orchestration layer that provides deterministic and resource‑efficient boot and lifecycle management, specifically tailored for low‑end embedded systems used in cost‑constrained production robots.

License: Apache 2.0
