# Plan A track integration status

This repository remains the detailed experimental engineering notebook for the AI Futures Project AI 2040 verification architecture. The current four-track coordination is distributed as follows:

- Track 1 (company auditing): AnonRish/ai-2040-plan-a-toolkit/embedded_audit provides a verifier-challenged evidence loop, hash-chain evidence and a multi-signer receipt, plus protocol-level red-team tests.
- Track 2 (initial international verification): AnonRish/frontier-verify provides /v1/recomputation/v2, which executes the bounded software path from challenge commitment through locked submission, content-addressed inputs, randomized sample selection, deterministic recomputation and signed receipt. The code in this repository remains the deeper reference implementation for passive-tap, DiFR, memory-wipe and physical-trust research.
- Track 3 (absence of secret compute): AnonRish/continental-load-registry provides the public-data/evidence layer and now includes track3_bound.py, a conservative residual-compute engine that refuses a global numeric result while the underlying accounting population remains open.
- Track 4 (full workload verification): this repository's §§23-28 plus AnonRish/ai-2040-plan-a-toolkit/plan_a_protocol/workload.py provide workload policy/certificate research and a machine-checkable manifest for code, model, data, runtime, compiler, operations and resource limits.

The common engineering contract is:

Observation -> Evidence -> Claim -> Verification procedure -> Result -> Attestation -> Receipt

Important boundary: software completion does not create physical inspection authority, globally complete transaction records, frontier-scale recomputation, hardware-rooted attestation, cryptographic proof of arbitrary frontier workloads, or independent organizational review. Those remain explicit external gates.
