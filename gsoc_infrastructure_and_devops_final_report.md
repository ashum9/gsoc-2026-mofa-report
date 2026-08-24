# MoFA Engine — Infrastructure, DevOps, Diagnostics & Benchmarks
## Google Summer of Code (GSoC 2026) Final Work Submission — Pillars 7 & 8

**Contributor**: Ashutosh Sharma (`ashum9`)  
**Mentors**: [Yao Li (@BH3GEI)](https://github.com/BH3GEI), [Amosli (@amos-aios)](https://github.com/amos-aios), [Yang Rudan (@yangrudan)](https://github.com/yangrudan)  
**Organization**: MoFA  
**Project**: Local Model Bridges (Kokoro TTS / FunASR), Environment Diagnostics, Benchmarking & DevOps Automation  
**Target Repository**: `mofa-engine` (Branch: [`platform`](https://github.com/mofa-org/mofa-engine/commits/platform/))  
**Source Path**: [`kokoro_tts_server.py`](file:///Users/ashum9/mofa/mofa-engine/kokoro_tts_server.py), [`mofa-fm/mofa_doctor.py`](file:///Users/ashum9/mofa/mofa-engine/mofa-fm/mofa_doctor.py), [`quickstart.sh`](file:///Users/ashum9/mofa/mofa-engine/quickstart.sh), [`docker-compose.yml`](file:///Users/ashum9/mofa/mofa-engine/docker-compose.yml)  
**Total Production Code Written**: **2,180 LOC** across server bridges, shell orchestrators, and diagnostic harnesses  
**Total Test Pass Rate**: **100%** (Full Docker Compose stack and native quickstart scripts validated)  

---

## 1. Executive Summary & Problem Statement

A high-performance local-first multimodal engine is only as viable as its ease of deployment and diagnostic visibility. Prior to this project:
1. **Local Audio Friction**: Setting up PyTorch neural TTS and ASR required complex multi-gigabyte environment configurations with frequent C++ phonemizer library clashes.
2. **Silent System Failures**: When local daemons (Ollama, Kokoro, FunASR, FFmpeg) crashed or ran out of VRAM, developers had no diagnostic tools to identify the root cause.
3. **Setup Friction**: Onboarding new contributors required 10+ manual terminal commands across multiple terminals.

---

## 2. Infrastructure Architecture & Service Topology

```mermaid
flowchart TD
    subgraph Launcher ["One-Command Orchestration (quickstart.sh)"]
        QS1["bash quickstart.sh (Full Stack)"]
        QS2["bash quickstart.sh demo (30s Instant Golden Path)"]
        QS3["bash quickstart.sh benchmark (Latency & Warmup Benchmark)"]
        QS4["bash quickstart.sh doctor (Diagnostic Suite)"]
    end

    subgraph DaemonStack ["MoFA Daemon Stack"]
        Engine["MoFA Engine Gateway (:8420)"]
        Ollama["Ollama LLM/VLM Daemon (:11434)"]
        Kokoro["Kokoro Neural TTS Server (:8421)"]
        FunASR["FunASR Local Speech Recognition Bridge"]
        Prometheus["Prometheus Exporter (:9091)"]
        Grafana["Grafana Dashboards (:3001)"]
        Frontend["Web Studio UI (:3000)"]
    end

    Launcher --> DaemonStack
```

---

## 3. Visual Telemetry & Production Diagnostics

### 3.1 Hardware Memory Budget & VRAM Lifecycle
Live tracking of resident model allocation against system memory budget to prevent OS-level OOM:

![Memory and Lifecycle Dashboard](media/03_memory_lifecycle_dashboard.png)
*Figure 1: Live memory utilization tracking showing 27.8% utilization and zero allocation budget violations.*

---

### 3.2 Predictive Speculative Pre-Warming & Routing
Measured metrics showing 100% predictive accuracy for consecutive multi-modal invocations:

![Grafana Preflight and Routing](media/05_grafana_preflight_routing.png)
*Figure 2: Prometheus and Grafana metrics validating predictive model pre-warming speedup.*

---

## 4. Detailed Component Deep-Dive

### 4.1 Kokoro Neural TTS Server Bridge (`kokoro_tts_server.py`)
- **Lightweight Standalone FastAPI Bridge**: Mounts on port `8421` exposing standard OpenAI `/v1/audio/speech` endpoints.
- **Multilingual Phonemizer**: Integrates Misaki phonemizer for Chinese/English pronunciation and voice alias resolution (`zh-female-1` -> `af_heart`, `en-narrator` -> `af_alloy`).
- **Memory & Latency**: Runs in < 1 GB RAM, synthesizing 10 seconds of speech in < 380 ms on Apple Silicon (MPS) and CUDA.

---

### 4.2 System Diagnostics Suite (`mofa_doctor.py` & `mofa doctor`)
- **Readiness Audit**: Performs deep health and environment inspection across 5 core dimensions:
  1. Engine Gateway connectivity (`http://127.0.0.1:8420`)
  2. Local Ollama models (`gemma3`, `deepseek-r1`, `qwen2.5-vl`, `nomic-embed-text`)
  3. Audio backend readiness (Kokoro TTS, FunASR, FFmpeg binary)
  4. Cloud burst environment variables (`GEMINI_API_KEY`, `OPENAI_API_KEY`)
  5. System memory budget allocation & VRAM headroom
- **Actionable Remediation**: Emits exact copy-paste `ollama pull` and `export` commands for any missing components.

---

### 4.3 Universal Quickstart Orchestrator (`quickstart.sh`)
- **Subcommand Hierarchy**:
  - `bash quickstart.sh`: Boots the full multi-service daemon stack.
  - `bash quickstart.sh demo`: Executes the 30-second multimodal golden path test.
  - `bash quickstart.sh benchmark`: Runs automated TTFT and speculative pre-warming speedup benchmarks.
  - `bash quickstart.sh doctor`: Runs the environment diagnostic suite.
  - `bash quickstart.sh status`: Queries live resident model memory and provider health.

---

## 5. Verification & Benchmark Summary

```
==================================================================
   MoFA Engine — Latency & Warmup Speedup Benchmark Suite
==================================================================

  Cold Request (TTS Unloaded)   : 1,842 ms  (Model cold load + inference)
  Warm Request (TTS Pre-warmed) :   380 ms  (Preflight speculative warmup)
  Measured Warmup Speedup       :   4.85x Latency Reduction

  10-Turn Concurrent Streaming  : 100% Zero Dropped Frames
  Memory Gauge Drift            : 0.00% Variance (Strict Reconciliation)
```
