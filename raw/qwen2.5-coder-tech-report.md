# Qwen2.5-Coder Technical Report

source_url: https://arxiv.org/abs/2409.12186 (HTML: https://arxiv.org/html/2409.12186v3)
fetched: 2026-05-07

## Paper metadata
- arXiv: 2409.12186
- Submitted: 2024-09-18
- Latest revision: v3, 2024-11-12
- Category: cs.CL
- Authors: Binyuan Hui, Jian Yang, et al. (Qwen team)

## Models introduced
Six sizes: Qwen2.5-Coder-(0.5B / 1.5B / 3B / 7B / 14B / 32B), each with Base and Instruct variants.

## Architecture (Instruct variants used in this wiki)
| Size | Hidden | Layers | Q heads | KV heads | Intermediate |
|------|--------|--------|---------|----------|--------------|
| 7B   | 3,584  | 28     | 28      | 4        | 18,944       |
| 14B  | 5,120  | 48     | 40      | 8        | 13,824       |
| 32B  | 5,120  | 64     | 40      | 8        | 27,648       |

All models use vocab size 151,646; trained on 5.5T tokens.

## Benchmark tables (Instruct)

### Table 16 — HumanEval / MBPP / BigCodeBench / LiveCodeBench
| Model        | HE   | HE+  | MBPP | MBPP+ | BCB-C | BCB-H | BCB-I | LCB(7-9/24) |
|--------------|------|------|------|-------|-------|-------|-------|-------------|
| 7B-Instruct  | 88.4 | 84.1 | 83.5 | 71.7  | 76.9  | 16.2  | 41.0  | 18.2        |
| 14B-Instruct | 89.6 | 87.2 | 86.2 | 72.8  | 81.0  | 22.3  | 48.4  | 23.4        |
| 32B-Instruct | 92.7 | 87.2 | 90.2 | 75.1  | 83.0  | 26.4  | 49.6  | 31.4        |

### Table 17 — MultiPL-E (Pass@1 across 8 languages)
| Model        | Py   | Java | C++  | C#   | TS   | JS   | PHP  | Bash |
|--------------|------|------|------|------|------|------|------|------|
| 7B-Instruct  | 87.8 | 76.5 | 75.6 | 80.3 | 81.8 | 83.2 | 78.3 | 48.7 |
| 14B-Instruct | 89.0 | 79.7 | 85.1 | 84.2 | 86.8 | 84.5 | 80.1 | 47.5 |
| 32B-Instruct | 92.7 | 80.4 | 79.5 | 82.9 | 86.8 | 85.7 | 78.9 | 48.1 |

### Table 18 — CRUXEval (Pass@1 with CoT)
| Model        | Input-CoT | Output-CoT |
|--------------|-----------|-----------|
| 7B-Instruct  | 65.8      | 65.9      |
| 14B-Instruct | 69.5      | 79.5      |
| 32B-Instruct | 75.2      | 83.4      |

### Table 19 — Aider
| Model        | Pass@1 | Pass@2 |
|--------------|--------|--------|
| 7B-Instruct  | 55.6   | 68.4   |
| 14B-Instruct | 58.6   | 69.2   |
| 32B-Instruct | 60.9   | 73.7   |

## Benchmarks NOT covered in the tech report
- BFCL (Berkeley Function-Calling Leaderboard)
- SWE-bench / SWE-bench Verified
- Spider / BIRD numerics (figures only)

## Licensing claim from paper
"Permissive licensing" supports adoption — actual license per HF cards is Apache-2.0 (all 7B/14B/32B Instruct variants).
