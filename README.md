# vllm-inference-bench

Free-tier benchmarks of **vLLM** vs Hugging Face `generate` on a **Google Colab Tesla T4**.

This project shows how I measure LLM inference behavior: throughput (tokens/s), latency, and the effect of batching. It is small on purpose so anyone can reproduce it on free hardware.

## Hardware and model

| Item | Value |
|------|--------|
| GPU | NVIDIA Tesla T4 (Colab free) |
| Model | `Qwen/Qwen2.5-0.5B-Instruct` |
| Max new tokens | 64 |
| Decoding | Greedy (`temperature = 0`) |
| Prompts | 3 fixed short prompts (same for every run) |

## Results (this run)

### Summary

| Backend | Setting | Tokens/s |
|--------|---------|----------|
| Hugging Face | batch size 1 (mean of 3 prompts) | **19.4** |
| vLLM | batch size 1 (mean of 3 prompts) | **152.2** |
| vLLM | batched (3 prompts in one call) | **448.4** (total) |

- vLLM batch-1 was about **7.8×** higher tokens/s than Hugging Face on the same model and GPU.
- Batching three prompts in one vLLM call raised **total** tokens/s to **448.4** (wall time stayed ~0.43 s for 192 new tokens).

### Hugging Face raw (batch size 1)

| Prompt | New tokens | Latency (s) | Tokens/s |
|--------|------------|-------------|----------|
| 1 | 64 | 4.80 | 13.3 |
| 2 | 64 | 3.13 | 20.5 |
| 3 | 64 | 2.61 | 24.5 |
| **Mean** | — | — | **19.4** |

### vLLM raw

| Backend | Batch size | Prompts | New tokens total | Wall time (s) | Tokens/s total | Latency per prompt (s) |
|--------|------------|---------|------------------|---------------|----------------|-------------------------|
| vllm | 1 | 1 | 64 | 0.437 | 146.6 | 0.437 |
| vllm | 1 | 1 | 64 | 0.418 | 152.9 | 0.418 |
| vllm | 1 | 1 | 64 | 0.408 | 157.0 | 0.408 |
| vllm | 3 | 3 | 192 | 0.428 | **448.4** | 0.143 |

Mean vLLM batch-1 tokens/s: **152.2**

Raw CSVs from this run: `summary.csv`, `hf_raw.csv`, `vllm_raw.csv`.

## How to reproduce (free)

1. Open `vllm_bench_colab.ipynb` in [Google Colab](https://colab.research.google.com).
2. **Runtime → Change runtime type → GPU (T4)**.
3. Run all cells from top to bottom.
4. Download the CSV files the notebook writes.

## Limitations

- Single free T4, not multi-node or production traffic.
- Small model only (fits in 16 GB).
- Colab shared GPUs and cold start can change absolute numbers between runs.
- Metrics here are **throughput and wall latency** under a simple scripted load, not a full production trace (no continuous batching traffic mix, no TTFT breakdown under concurrent users).
- Goal is a clear method and fair comparison, not peak numbers on H100/B200.

## Why this exists

Inference work needs measurable baselines. This repo is a free template for that habit: fixed prompts, warm-up, timed runs, and written limits—the same discipline used when tuning serving stacks for latency, throughput, and cost.
