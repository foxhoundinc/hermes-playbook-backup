# 02-machines — Machine-Specific Configs

> **INBOX:** Unsorted links, PDFs, and notes go here first. Sort into sub-docs after review.

## Hardware Reality Check

### Headless NUC (always-on relay)
| Component | Spec | Implication |
|-----------|------|-------------|
| CPU | Intel i3 (base) | Very limited compute — no heavy processing |
| RAM | 8GB DDR4 | OS + Hermes idling eats ~4-5GB. ~3GB free for tasks |
| Storage | 120GB SSD | Tight — Hermes + tools only. No local model storage |
| **Role** | Pure relay | Cron scheduling, Syncthing, Home Assistant, lightweight bots |

### Win11 Main PC (heavy lifter)
| Component | Spec | Implication |
|-----------|------|-------------|
| CPU | AMD Ryzen 9 9950X (16C/32T) | Enormous headroom for parallel tasks |
| GPU | RTX 5070 Ti | Local GGUF inference via llama.cpp viable |
| RAM | 64GB DDR5 6000 | Room for multiple local models in memory |
| Storage | (presumably large SSD) | Space for model files, datasets |
| **Role** | Everything heavy | Development, local inference, GPU tasks, large file work |
