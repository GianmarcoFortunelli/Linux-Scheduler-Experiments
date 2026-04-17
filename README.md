# Linux Scheduler Experiments

A portfolio repository for a set of **Linux CFS scheduler experiments** carried out on **Linux 5.4.281**, with a focus on **wake-up latency**, **scheduler tuning**, and **adaptive per-core behavior**.

The project combines **kernel-level modifications**, **controlled benchmarking**, **Ftrace-based observability**, and a reproducible experimental pipeline built around **QEMU** and a minimal **BusyBox** userspace.

## 🚀 Overview

This work explores how targeted modifications to the **Completely Fair Scheduler (CFS)** affect responsiveness and scheduling behavior under controlled workloads.

Rather than evaluating scheduler changes informally, the project builds a reproducible infrastructure for:

- compiling and running both **baseline** and **modified** kernels
- generating a minimal execution environment
- tracing scheduler events with **Ftrace**
- extracting latency statistics automatically
- comparing different scheduling strategies under identical conditions

The experiments were conducted on **Linux 5.4.281** for **x86_64**, using a tightly controlled virtualized setup to reduce non-deterministic noise.

## ⚙️ Experimental Infrastructure

The evaluation pipeline is based on:

- **Linux kernel 5.4.281 (LTS)**
- **BusyBox 1.36.1**
- **QEMU system-x86**
- **Ftrace** for scheduler event tracing
- **static benchmark binaries**
- automated parsing and statistical aggregation

To improve reproducibility, the setup includes:

- **single-core VM execution** for baseline/custom comparisons
- CPU pinning of the QEMU process
- disabled turbo and frequency scaling
- automated build, boot, trace dump, and shutdown
- repeated runs with **median-of-medians** aggregation over **20 executions**

## 🧠 Project Structure

The project is organized around three scheduler interventions:

### 1. Static vruntime boosting
A direct modification of CFS placement logic to favor tasks exhibiting I/O-like behavior through a manual **vruntime subtraction**.

This approach was based on simple task-behavior filters derived from context-switch statistics and was used to explore whether direct boosting in the wake-up path could reduce observed latency.

### 2. Short block preempt
A wakeup-preemption refinement inside **check_preempt_wakeup()**, designed to give recent sleepers a second chance to preempt the running task when fairness conditions still hold under a reduced granularity.

This experiment focused in particular on the behavior of the **tail of the latency distribution** under mixed workloads.

### 3. Adaptive per-core scheduling policy
A policy that dynamically biases each CPU toward **responsiveness** or **throughput** depending on the observed workload mix.

The design is based on three stages:

- **per-task behavioral characterization**
- **per-core workload aggregation**
- **adaptive scaling of scheduling parameters**

Each fair-class task is classified dynamically using EWMA-smoothed estimates of:

- CPU burst duration
- sleep duration

At the runqueue level, per-task classifications are aggregated into a per-core estimate of I/O intensity, which is then used to tune:

- **minimum granularity**
- **wakeup granularity**

This enables different cores to react independently to heterogeneous workload phases.

## 📊 Main Outcomes

| **Experiment** | **Focus** | **Outcome** |
|---|---|---|
| **Static vruntime boosting** | Bias task placement through manual vruntime reduction | **Limited impact** on wake-up latency metrics |
| **Short block preempt** | Improve responsiveness for recent sleepers | **Parameter-sensitive behavior**, with no consistent gain across tested configurations |
| **Adaptive per-core scheduling** | Tune responsiveness vs throughput dynamically on each CPU | **Task classification accuracy above 98%**, with **per-core granularity parameters** tracking workload phases |

## 🔬 Methodology

The measurement flow is fully automated and identical for baseline and modified kernels, except for the scheduler logic under test.

The pipeline includes:

1. kernel and userspace image construction  
2. automated boot inside QEMU  
3. benchmark execution inside the guest  
4. scheduler tracing with **sched_wakeup** and **sched_switch**  
5. raw trace extraction through the serial console  
6. host-side parsing of wake-up latency samples  
7. statistical aggregation of **p50**, **p95**, and **p99**

The benchmark suite uses controlled mixtures of:

- **CPU-bound tasks**
- **I/O-bound ping-pong processes**
- phase-controlled workloads for the adaptive policy

## 📄 Repository Content

- **`Linux_Scheduler_Report.pdf`** — full technical report

## 📘 Report

[**Open the report**](./Linux_Scheduler_Report.pdf)

## 🔒 Source Code Availability

The source code is **not publicly available** for academic reasons.  
It can be shared upon request for **recruiting purposes**.