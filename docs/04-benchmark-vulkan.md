# Vulkan Benchmark – Intel Arc Pro B70

## Benchmark results for llama.cpp
**Hardware:** Intel Arc Pro B70 (32 GB VRAM)

These benchmarks serve primarily to track the performance development of various models as well as changes to `llama.cpp` and configuration.

> **Important:** Values from different benchmark runs are not always 1:1 comparable. In some cases, llama.cpp versions, build parameters, model quantization, or other settings were changed between measurements.

---

## Test System

* **GPU:** Intel Arc Pro B70 (32 GB VRAM)
* **OS:** Ubuntu Linux
* **Backend:** Vulkan
* **Server:** `llama-server`
* **API:** OpenAI-compatible (`/v1/chat/completions`)

---

## Benchmark Prompt

The following prompt was used for subsequent standardized tests:
*"Erkläre kurz, was ein MOSFET ist."*

### Request
```bash
curl http://localhost:8082/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Erkläre kurz, was ein MOSFET ist."
      }
    ],
    "max_tokens": 200,
    "temperature": 0.7
  }'
```

### Measured Metrics
Two values in particular are being considered:
1. **prompt tok/s**: Speed of processing the input prompt (Prefill).
2. **gen tok/s**: Speed of generating the response (Decoding).

---

## Benchmark History

### Overview

| Date / Build | llama.cpp | Model | Quantization | Prompt tok/s | Gen tok/s | Remarks |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| older build | b9436 / d6588daa | Qwen3-Coder 30B A3B | Q5_K_M | 18.87 | 57.32 | Basic |
| older build | b9436 / d6588daa | Qwen3-Coder 30B A3B | Q5_K_M | 51.16 | 69.37 | Optimized |
| older build | b9436 / d6588daa | Gemma 4 26B A4B | MXFP4 | 74.10 | 29.91 | Basic |
| older build | b9436 / d6588daa | Gemma 4 26B A4B | MXFP4 | 74.61 | 33.50 | Optimized |
| older build | b9436 / d6588daa | Gemma 4 12B | Q5_K_M | 37.05 | 30.95 | Optimized |
| older build | b9436 | Qwen3.8 27B UD | Q6_K | 18.24 | 7.37 | older test |
| older build | b9436 | Qwen3.8 27B UD | Q4_K_M | 18.07 | 14.11 | older test |
| 24.08.2026 | b10516 / b95502ba9 | Qwen3.8 27B UD | Q4_K_M | 33.85 | 20.86 | new llama.cpp build |
| 17.09.2026 | b10516 / b95502ba9 | Qwen2.5 1.5B-Instruct | Q4_K_M | 227.17 | 242.78 | MOSFET test |
| 17.09.2026 | b10516 / b95502ba9 | Gemma 4 26B A4B | MXFP4_MOE | 89.81 | 56.29 | MOSFET test |
| 17.09.2026 | b10516 / b95502ba9 | Qwen3-Coder 30B A3B-Instruct | Q5_K_M | 98.33 | 100.48 | MOSFET test |
| 17.09.2026 | b10516 / b95502ba9 | Gemma 4 12B | Q5_K_M | 78.10 | 44.04 | MOSFET test |
| 17.09.2026 | b10516 / b95502ba9 | Qwen3 14B | Q4_K_M | 51.06 | 52.10 | MOSFET test |
| 17.09.2026 | b10516 / b95502ba9 | Qwen3.8 27B UD | Q4_K_M | 21.15 | 20.64 | MOSFET test |
| 17.09.2026 | b10516 / b95502ba9 | Gemma 4 26B | Q4_0 | 97.89 | 60.49 | MOSFET test |

---

## Current Benchmark – llama.cpp Build 10516

All measurements since **24.08.2026** were performed with the following `llama-server`:

```text
version: 0.1.2-dev
build: 10516
commit: b95502ba9
built with IntelLLVM 2026.0.0 for Linux x86_64
```

The version used can be checked with the following command:
`llama-server --version`

### Test Parameters
* **Prompt:** Explain briefly what a MOSFET is.
* **max_tokens:** 200
* **temperature:** 0.7

### Results

| Model | Prompt tok/s | Prompt ms/tok | Gen tok/s | Gen ms/tok | Gen total ms | Cached | Wall s | Finish |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Qwen2.5-1.5B-Instruct-Q4_K_M.gguf | 227.17 | 4.40 | 242.78 | 4.12 | 819.7 | 40 | 0.83 | length |
| Gemma-4-26B-A4B-it-MXFP4_MOE.gguf | 89.81 | 11.13 | 56.29 | 17.76 | 3535.0 | 21 | 3.62 | length |
| Qwen3-Coder-30B-A3B-Instruct-Q5_K_M.gguf | 98.33 | 10.17 | 100.48 | 9.95 | 1910.8 | 19 | 1.93 | stop |
| gemma-4-12b-it-Q5_K_M.gguf | 78.10 | 12.80 | 44.04 | 22.71 | 4518.8 | 21 | 4.62 | length |
| Qwen3-14B-Q4_K_M.gguf | 51.06 | 19.59 | 52.10 | 19.19 | 3819.4 | 19 | 3.86 | length |
| Qwen3.8-27B-UD-Q4_K_M.gguf | 21.15 | 47.29 | 20.64 | 48.45 | 5959.5 | 60 | 6.31 | stop |
| gemma-4-26B_q4_0-it.gguf | 97.89 | 10.22 | 60.49 | 16.53 | 3290.0 | 21 | 3.37 | length |

---

## Explanation of Measured Values

#### Prompt tok/s
Speed (Tokens per second) at which `llama.cpp` processes the input context.
* **Higher is better.**
* *Example:* 100 prompt tok/s $\approx$ 1,000 prompt tokens in about 10 seconds.

#### Prompt ms/tok
Time (milliseconds) required for a single prompt token.
* **Lower is better.** (Inverse of `prompt tok/s`)

#### Gen tok/s
Speed (Tokens per second) at which the model generates new tokens.
* **Higher is better.** (Particularly relevant for perceived chatting speed).

#### Gen ms/tok
Time (milliseconds) for the generation of a single token.
* **Lower is better.**

#### Gen total ms
Total time for the generation of the measured response in milliseconds.

#### Cached
Number of tokens reused from the KV cache.

#### Wall s
Actual elapsed time (Wall time) for the respective request.
* *Note:* Depends on the number of generated tokens and is therefore only suitable for limited direct comparison of differently long responses.

#### Finish
Reason for the end of generation:
* `stop`: Model ended the response itself.
* `length`: The maximum token count (`max_tokens`) was reached.
* *Note:* When comparing, it should be taken into account whether a model ended with `stop` or `length`.

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
## Comparison of llama.cpp Development

### Qwen3-Coder 30B A3B Comparison
| llama.cpp Build | Configuration | Prompt tok/s | Gen tok/s |
| :--- | :--- | :--- | :--- |
| b9436 | Basic | 18.87 | 57.32 |
| b9436 | Optimized | 51.16 | 69.37 |
| b10516 | MOSFET test | 98.33 | 100.48 |

### Qwen3.8 27B Q4_K_M Comparison
| llama.cpp Build | Prompt tok/s | Gen tok/s |
| :--- | :--- | :--- |
| older b9436 test | 18.07 | 14.11 |
| b10516 / 24.08.2026 | 33.85 | 20.86 |
| b10516 / 17.09.2026 | 21.15 | 20.64 |

**Conclusion of comparison:** In addition to the `llama.cpp` version, the respective initial configuration and test execution can also have an influence. Therefore, measurements from different benchmark runs should not be interpreted exclusively as version comparisons.

---

## Short Summary of the Current B70 Test

* **Qwen3-Coder 30B A3B (Q5_K_M):** $\approx$ 98.33 prompt / 100.48 gen tok/s
* **Gemma 4 26B (Q4_0):** $\approx$ 97.89 prompt / 60.49 gen tok/s
* **Qwen3.8 27B (Q4_K_M):** $\approx$ 21.15 prompt / 20.64 gen tok/s
* **Small Qwen2.5 1.5B:** Achieves significantly higher rates as expected ($\approx$ 227 prompt / 242 gen tok/s).

*These values are intended primarily as performance measurements of the Intel Arc Pro B70 with llama.cpp/Vulkan and do not make any statement regarding the quality of the respective models.*
