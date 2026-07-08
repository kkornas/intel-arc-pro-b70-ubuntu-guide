# llama.cpp Vulkan Benchmark (Intel Arc Pro B70)

This repository contains benchmark results for `llama.cpp` using the **Vulkan backend** on an **Intel Arc Pro B70 (Battlemage G31, 32 GB VRAM)**.

The goal is to compare different models and runtime configurations. This document will be updated over time with additional models and optimizations.

## Test System

| Component | Value |
|-----------|-------|
| GPU | Intel Arc Pro B70 (Battlemage G31, 32 GB) |
| Backend | Vulkan |
| llama.cpp | b9436 (d6588daa8) |
| OS | Ubuntu |
| API | OpenAI-compatible (`llama-server`) |

---

# Test Prompt

All benchmarks use the same request:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Erkläre kurz, was ein MOSFET ist."
    }
  ],
  "max_tokens": 200,
  "temperature": 0.7
}
```

---

# Configurations

## Basic

```bash
llama-server \
  -m "$MODEL" \
  --host "$HOST" \
  --port "$PORT" \
  -ngl 999 \
  --ctx-size 8192 \
  -b 1024 \
  -ub 512 \
  --flash-attn \
  --jinja \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  -t 8 \
  -tb 4
```

## Optimized

```bash
llama-server \
  -m "$MODEL" \
  --host "$HOST" \
  --port "$PORT" \
  -ngl 999 \
  --ctx-size 32768 \
  --parallel 1 \
  -b 4096 \
  -ub 1024 \
  --flash-attn \
  --jinja \
  --cache-type-k f16 \
  --cache-type-v f16 \
  -t 8 \
  -tb 4
```

> **Note:** For Gemma-4-26B the optimized benchmark used `--ctx-size 16384`. All other parameters are identical.

---

# Results

| Model | Config | Prompt tok/s | Generation tok/s |
|--------|---------|-------------:|-----------------:|
| Qwen3-Coder-30B-A3B-Instruct-Q5_K_M | Basic | 18.87 | 57.32 |
| Qwen3-Coder-30B-A3B-Instruct-Q5_K_M | Optimized | **51.16** | **69.37** |
| gemma-4-26B-A4B-it-MXFP4_MOE | Basic | 74.10 | 29.91 |
| gemma-4-26B-A4B-it-MXFP4_MOE | Optimized | **74.61** | **33.50** |

---

# Improvement

## Qwen3-Coder-30B-A3B

| Metric | Basic | Optimized | Improvement |
|--------|------:|----------:|------------:|
| Prompt | 18.87 tok/s | 51.16 tok/s | **+171%** |
| Generation | 57.32 tok/s | 69.37 tok/s | **+21%** |

## Gemma-4-26B

| Metric | Basic | Optimized | Improvement |
|--------|------:|----------:|------------:|
| Prompt | 74.10 tok/s | 74.61 tok/s | ~0% |
| Generation | 29.91 tok/s | 33.50 tok/s | **+12%** |

---

# Notes

The optimized configuration mainly improves:

- larger batch sizes (`-b`, `-ub`)
- `f16` KV cache
- single parallel slot (`--parallel 1`)
- larger context size

The impact depends on the model architecture.

For example:

- **Qwen3-Coder** benefits significantly from the optimized settings.
- **Gemma-4-26B** mainly improves during token generation, while prompt processing remains almost unchanged.

This benchmark will be extended with additional models and configurations over time.
