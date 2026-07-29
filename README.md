<div align="center">

# AI 2040 Verification

**A working implementation of pieces of the compute-governance verification architecture proposed in the AI Futures Project's [*AI 2040: Plan A*](https://ai-2040.com).**

Real trained model. Real server. Real red-team attacks against that real server. Real bugs, found and reported, not quietly fixed and left out.

[![Status](https://img.shields.io/badge/status-work--in--progress-orange)](#progress)
[![Progress](https://img.shields.io/badge/sections-21%2F30-blue)](#progress)
[![CI](https://github.com/AnonRish/ai-2040-verification/actions/workflows/ci.yml/badge.svg)](https://github.com/AnonRish/ai-2040-verification/actions/workflows/ci.yml)
[![Rust](https://img.shields.io/badge/rust-tap__reader-CE422B?logo=rust&logoColor=white)](./tap_reader)
[![Python](https://img.shields.io/badge/python-DiFR%20server-3776AB?logo=python&logoColor=white)](#repository-layout)
[![C](https://img.shields.io/badge/C-memory%20wipe%20daemon-00599C?logo=c&logoColor=white)](./wipe_daemon.c)
[![License](https://img.shields.io/badge/license-none%20yet-lightgrey)](#license)

</div>

---

This is not a slide deck describing what a verification system *would* do. It's an attempt to actually build the pieces that are buildable in a sandbox with no GPU and one CPU core: a small but genuinely trained character-level language model, a real FastAPI + PostgreSQL inference-verification server, a real Rust packet-capture and hashing pipeline, real LoRA backdoor adapters, real red-team scripts run against the real server above, and a real tamper-evident log with real signature verification.

**Status: work in progress, shared deliberately mid-way rather than held back.** 21 of 30 planned sections are drafted. See [Progress](#progress) below, or the authoritative table at the top of [`ai-2040-verification.md`](./ai-2040-verification.md).

## Table of Contents

- [Why This Exists](#why-this-exists)
- [What's Actually Here](#whats-actually-here)
- [Architecture](#architecture)
- [Notable Findings](#notable-findings)
- [Progress](#progress)
- [Repository Layout](#repository-layout)
- [Getting Started](#getting-started)
- [Known Limitations](#known-limitations)
- [Sources and Prior Work](#sources-and-prior-work)
- [License](#license)

## Why This Exists

The AI Futures Project's *AI 2040: Plan A* — specifically its [Verification Plan supplement](https://ai-2040.com/supplements/verification-plan) (Romeo Dean, July 2026) — sketches a compute-governance regime modeled loosely on arms-control treaty inspection: passive taps on datacenter networks, statistical recomputation of sampled inference requests, verifier-issued randomness so a prover can't game what gets checked, periodic memory wipes, and physical-security auditing modeled on IAEA safeguards concepts (Significant Quantity, Detection Probability, conversion time).

That's a design document. This repo is an attempt to find out how much of it survives contact with an actual sandbox: what's genuinely buildable and testable today, where the real bottlenecks are (a link-budget problem that has no COTS solution yet; a disk-write collapse that shows up on three independently-measured storage systems; a detection scheme with a real, quantified blind spot), and what has to stay an honest design estimate because the hardware to test it doesn't exist here.

## What's Actually Here

Every section of [`ai-2040-verification.md`](./ai-2040-verification.md) follows the same discipline: a technical write-up, a working and tested code artifact, and a **Self-check** that states plainly what was demonstrated versus what's still a design estimate. A few things that discipline actually caught, rather than papered over:

- A stream-reassembly bug where a CRLF split exactly across a TCP segment boundary hung the parser — caught by a byte-at-a-time fragmentation test, fixed, documented.
- A distillation run that used hard-label gradients where soft KL-divergence against the teacher was required — a real bug in an early training pass, not a modeling choice.
- A `-O2` optimization that silently dead-code-eliminated the very fields a "certificate" mechanism was supposed to be certifying, letting two *differently declared* workloads compile to bit-identical binaries.
- A separate AI-generated document's "512-bit RSA" implementation that, when actually run, produced `Baseline Check: False` on its very first entry — the modulus was 79 bits, not 512, and `p × q` didn't equal the stated `n`.

Sandbox constraints (no GPU, no reachable path to pretrained weights, one logical CPU, ~5GB free disk) are stated explicitly wherever they bound a result, rather than smoothed over.

## Architecture

<img src="./docs/architecture.svg" alt="System topology: storage bank, inference units, passive optical splitter, trust boundary, recomputation server" width="620">

A prover's facility (storage bank, inference units, a passive optical splitter with no logic, firmware, or state) mirrors frontend traffic across a one-way trust boundary into a verifier-controlled recomputation server, which re-runs sampled requests under a verifier-issued seed and scores deviation against a per-model calibrated threshold, logged to a hash-chained, signed record. `§` references in the diagram point to the section of `ai-2040-verification.md` that builds and tests that piece.

## Notable Findings

Six results that came out of actually running things, not from reasoning about them in the abstract:

| Finding | Section |
|---|---|
| A trigger-conditional LoRA backdoor is **completely invisible** to output-level verification while dormant (100% pass rate, indistinguishable from honest) — and collapses to the maximum deviation ceiling (~7.98 of 8.0) the instant the trigger appears. | [§9](./ai-2040-verification.md) |
| Summing the same 4,096 float32 values under 12 different reduction groupings — standing in for a GPU kernel picking different shapes per batch size — produced **9 distinct bit-level results** from one canonical mathematical sum. A single fixed reduction tree collapsed that to exactly 1, reproducing at CPU scale the mechanism Thinking Machines Lab found on real GPUs. | [§7](./ai-2040-verification.md) |
| An aggregate AUC above 0.99 across every tested temperature *looked* like clean separation — but the actually-defensible threshold showed cross-family leakage climbing from 7.5% at T=0.3 to 17.5% at T=1.6. Temperature turned out to be a real detectability lever, not just a decoding parameter. | [§11](./ai-2040-verification.md) |
| A steganographic attacker spending up to half its tolerance budget per token smuggled ~75% of a 20-bit hidden payload through a 30-token output while passing every deviation check at 100% — caught only by a secondary, per-token distribution check running at roughly 124,000× the honest baseline rate. | [§10](./ai-2040-verification.md) |
| Every attacker tested against the sampling gateway's keyed PRF — including a logistic regression trained on 480,000 labeled examples — stayed within 0.005 of an uninformative AUC of 0.5. An oracle handed the actual key scored a perfect 1.0. The unpredictability holds because of key secrecy, not algorithm secrecy, demonstrated rather than assumed. | [§3](./ai-2040-verification.md) |
| A real disk-write collapse-after-burst was independently reproduced on **three different storage systems** — this sandbox's own cloud block storage, a second AI system's separate sandbox, and cited real NVMe hardware — at three different magnitudes but the same qualitative shape, which is worth more than any single number in the comparison. | [§15](./ai-2040-verification.md) |

## Progress

<!-- Keep in sync with the table at the top of ai-2040-verification.md -->

| Part | Sections | Status |
|---|---|---|
| I — Architecture | §1 | ✅ Drafted |
| II — Correctness | §2–§14 | ✅ Drafted |
| III — Completeness | §15–§18 | ✅ Drafted |
| IV — Physical & Operational Trust | §19–§21 | ✅ Drafted |
| | §22 TAP Installation & Monitoring at Scale | 🚧 In progress |
| | §23 Workload Approval | ⏳ Pending |
| V — Beyond Inference-Only | §24–§25 | ⏳ Pending |
| VI — The Frontier | §26 | ⏳ Pending |
| VII — Holding Together | §27–§28 | ⏳ Pending |
| VIII — Other Tracks | §29–§30 | ⏳ Pending |

Full section-by-section detail (each with its own **Self-check** and **Sources used**) lives in [`ai-2040-verification.md`](./ai-2040-verification.md).

## Repository Layout

```
ai-2040-verification.md   The document. Every drafted section: write-up, self-check, sources.
docs/
  architecture.svg          The topology diagram embedded above.
*.py                       DiFR verification server, the trained character-LM, LoRA backdoor
                           adapters, the red-team suite, calibration sweeps, tamper-evident log.
*.json                     Real output from actual runs of the scripts above.
wipe_daemon.c               Memory-wipe daemon (§15) — AES-NI DAG labeling + full overwrite.
                           verifier.py independently re-checks its output with a maintained
                           crypto library rather than sharing code with the daemon.
tap_reader/                 Rust crate: passive-tap parsing, hashing, HTTP chunked/SSE stream
                           reassembly, a lock-free evidence ring buffer, the sampling gateway,
                           and Merkle/epoch commitments (§2–§4). Has its own README.
```

## Getting Started

**Rust (`tap_reader/`)** is fully self-contained:

```bash
cd tap_reader
cargo build --release
cargo test --release      # 17/17 unit tests
```

See [`tap_reader/README.md`](./tap_reader/README.md) for the individual benchmark binaries.

**Python** needs `pip install -r requirements.txt`. A few scripts depend on artifacts generated by *other* scripts rather than checked in here — consistent with not committing large binaries:

1. `train.py` needs `shakespeare.txt` (the tiny-shakespeare corpus — see [Sources](#sources-and-prior-work)) and produces `charlm.pkl` + `vocab.pkl`.
2. `train_variants.py` (needs `charlm.pkl`) produces the same-family and cross-family checkpoints used throughout the red-team and calibration sections.
3. `lora.py` (needs `charlm.pkl` + `vocab.pkl`) produces the backdoor adapters.

Run those three, in that order, before anything that loads a `.pkl` checkpoint (`server.py`, `test_suite.py`, `redteam.py`, `calibration.py`, and others). `db.py` expects a local PostgreSQL instance matching the DSN at the top of that file.

## Known Limitations

Stated here the same way they're stated throughout the document, not softened:

- **No GPU, anywhere.** The "trained language model" is a small NumPy character-level model (8-character context, 65-token vocabulary) — real weights, real backprop, real learned structure, and nowhere near a frontier model's capability. Every claim in the document is about the *verification protocol*, never about this standing in for a frontier model.
- **One logical CPU, no DPDK-capable NIC.** Rust throughput numbers are a measured software ceiling on this container, not a real-NIC/real-DMA capture number — stated explicitly at every point the document uses them.
- **A known, unclosed detection gap.** §9 found and §10 confirmed that a dormant trigger-conditional backdoor is structurally invisible to output-level statistical verification. This isn't hidden — it's the reason §10 (red-team) and §12 (anti-steganography) exist.
- **9 sections remain**, including at least one (§13's raw-instruction bypass) that the document itself names as a policy dependency rather than something a verification mechanism alone can close.

## Sources and Prior Work

- AI Futures Project, [*AI 2040: Plan A*](https://ai-2040.com) — Verification Plan supplement (Romeo Dean), July 2026.
- Karvonen, Reuter, Rinberg, Marks, Garriga-Alonso, Warr, ["DiFR: Inference Verification Despite Nondeterminism,"](https://arxiv.org/abs/2511.20621) arXiv:2511.20621.
- He et al. (Thinking Machines Lab), "Defeating Nondeterminism in LLM Inference," Sep 2025 — the batch-invariant-kernels finding §7 reproduces at CPU scale.
- Bursuc, Gil-Pons, Mauw, Trujillo-Rasua, ["Software-Based Memory Erasure with Relaxed Isolation Requirements,"](https://arxiv.org/abs/2401.06626) arXiv:2401.06626, CSF 2024 — the memory-wipe daemon's security argument.
- `karpathy/char-rnn` (tiny-shakespeare corpus) — training data for the stand-in language model.

Every drafted section names its own sources in full at the point it uses them — this list is the recurring ones, not the complete bibliography.

## License

No license has been chosen yet, which by default means **all rights reserved** — nobody else may legally copy, modify, or redistribute this without permission, regardless of the code being publicly visible. If the intent is for others to actually use or build on this, a license needs adding. This isn't legal advice, just the factual default; a couple of common, low-friction options for this kind of project:

| Option | What it does |
|---|---|
| **MIT** | Maximally permissive for the code. Anyone can use, modify, and redistribute, including commercially, as long as the license text is kept. |
| **Apache-2.0** | Similar permissiveness, plus an explicit patent grant — often preferred for anything security- or infrastructure-adjacent. |
| **CC-BY-4.0** | Fits the write-up (`ai-2040-verification.md`) better than the code if the document itself is the primary thing being shared — requires attribution, allows reuse. |

GitHub can add most of these for you: **Settings → General → Add a license file**.
