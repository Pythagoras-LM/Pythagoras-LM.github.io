<div align="center">
  <img src="figures/pythagoras-website-logo.png" width="120" alt="Pythagoras-Prover logo"><br>
  <h1>Pythagoras-Prover</h1>
</div>

<div align="center">

[![Paper](https://img.shields.io/badge/Paper-arXiv-b31b1b?logo=arxiv&logoColor=white)](https://arxiv.org/abs/XXXX.XXXXX)
[![HuggingFace](https://img.shields.io/badge/%F0%9F%A4%97%20HuggingFace-Pythagoras--LM-ffc107?color=ffc107)](https://huggingface.co/Pythagoras-LM)
[![Code](https://img.shields.io/badge/GitHub-Pythagoras--Prover-181717?logo=github&logoColor=white)](https://github.com/Pythagoras-LM/Pythagoras-Prover)
[![Stars](https://img.shields.io/github/stars/Pythagoras-LM/Pythagoras-Prover?style=social)](https://github.com/Pythagoras-LM/Pythagoras-Prover)

</div>

<p align="center"><b>Paper Link &nbsp;|&nbsp; <a href="https://arxiv.org/abs/XXXX.XXXXX">arXiv</a></b></p>

<p align="center">
  <a href="#1-introduction">Introduction</a> |
  <a href="#2-model-summary">Model Summary</a> |
  <a href="#3-minif2f-alf-benchmark">MiniF2F-ALF</a> |
  <a href="#4-evaluation">Evaluation</a> |
  <a href="#5-models--dataset-downloads">Downloads</a> |
  <a href="#6-quick-start">Quick Start</a> |
  <a href="#7-license">License</a> |
  <a href="#8-citation--contact">Citation &amp; Contact</a>
</p>

# Pythagoras-Prover: Advancing Efficient Formal Proving via Augmented Lean Formalisation

## 1. Introduction

We introduce **Pythagoras-Prover**, a compute-efficient family of open-source large language models for formal theorem proving in Lean 4. The family comprises two autoregressive provers at 4B and 32B parameters, together with **Pythagoras-Prover-Diffusion**, the first diffusion-based theorem prover, which iteratively refines Lean proofs at inference time. All three models are artefacts of a single methodological approach: a scalable, Lean-verified synthetic data pipeline. At its centre is **Augmented Lean Formalisation** (ALF), a structured mutation scheme that expands a verified seed corpus into formal variants without per-instance Lean compilation, then re-uses them as a self-distillation signal during training. This design lets careful data construction stand in for raw scale, closing much of the gap between small open provers and their largest counterparts — without relying on inference-time self-correction.

<div align="center">
  <img src="figures/prover_fig1_hi.png" alt="Pythagoras-Prover benchmark overview" width="90%">
</div>

## 2. Model Summary

---

**A Lean-Verified Synthetic Data Pipeline**

- Natural-language problems from general-math and competition sources are autoformalised into Lean and gated on the type-checker; an auto-informalisation and alignment step discards faithful-but-wrong formalisations, yielding a seed corpus across easy, medium, and hard tiers built predominantly from sub-30B open models.

- A failure-mode-conditioned, rubric-guided distillation stage re-prompts on each rejected instance to target the specific Lean type-checker error, lifting autoformalisation success and roughly doubling the verified training set.

---

**Augmented Lean Formalisation (ALF) and Self-Distillation**

- ALF emits one structured variant per category — simplification, generalisation, lemma proposal, proof-step decomposition, and reformulation — for every seed statement, replacing per-instance Lean verification with a cheap statement-alignment check and expanding the seed roughly 3×.

- The post-RL prover then proves its own mutations; these self-distilled proofs form a corpus of ~2M instances that trains both the autoregressive and diffusion provers from a single recipe.

---

**Compute-Efficient Training**

- LoRA-only supervised fine-tuning under an 8K context, paired with a dynamic proof-reasoning filter and a difficulty-ordered easy→medium→hard curriculum, followed by reinforcement learning with a Lean-compilation reward and a final continued-SFT stage on the ALF corpus.

---

**The First Diffusion-Based Theorem Prover**

- Pythagoras-Prover-Diffusion adapts a block-diffusion formulation with a tactic-based masking objective aligned to the discrete reasoning steps of Lean — to our knowledge the first demonstration that a diffusion language model can verifiably solve Lean theorems at non-trivial rates.

---

The resulting models set a new bar for compute-efficient formal proving. **Pythagoras-Prover-32B** achieves state-of-the-art performance among open-source provers, reaching **93.03%** on MiniF2F-Test and solving **93 of 672** problems on PutnamBench, while **Pythagoras-Prover-4B** outperforms DeepSeek-Prover-V2-671B on MiniF2F-Test despite being roughly **167× smaller** — with no self-correction or test-time reinforcement learning. We additionally release **MiniF2F-ALF**, an ALF-mutated companion benchmark on which every evaluated prover degrades.

## 3. MiniF2F-ALF Benchmark

We release **MiniF2F-ALF**, a 488-statement companion benchmark constructed by applying the ALF mutation scheme to each of the 244 statements in MiniF2F-Test. For each problem we generate five candidate mutations and retain the two most divergent under embedding cosine distance, then verify that each retained statement is a well-formed Lean theorem.

MiniF2F-ALF plays a dual role:

- **Decontamination probe** — it preserves the original problem families while perturbing the statements, distinguishing genuine proof ability from recall of potentially contaminated training data.
- **Transfer probe** — it tests whether the same ALF operator used for training augmentation generalises to ALF-mutated test statements.

Because the original MiniF2F-Test is approaching saturation, MiniF2F-ALF also restores discriminative power: under mutation, the number of instances on which strong provers disagree roughly doubles.

## 4. Evaluation

All results use Lean 4.9.0-rc1 with a maximum generation length of 30,000 tokens. A proof attempt is judged correct only if it compiles with no errors and no `sorry`/`admit`, and the target statement appears verbatim in the output. We report pass@N, the fraction of problems solved by at least one of N independent samples.

### MiniF2F-Test (pass@N, %)

| Method | #Params | Pass@32 | Pass@1024 | Best (N) |
| --- | :---: | :---: | :---: | :---: |
| Goedel-Prover-SFT | 7B | 57.6 | – | 62.7 (3200) |
| STP | 7B | – | – | 67.6 (25600) |
| Kimina-Prover-Preview-72B | 72B | 68.85 | – | 80.74 (8192) |
| DeepSeek-Prover-V2-7B | 7B | 75.6 | – | 82.0 (8192) |
| DeepSeek-Prover-V2-671B | 671B | 82.4 | – | 88.9 (8192) |
| Kimina-Prover-8B-Distill | 8B | 77.86 | – | – |
| Kimina-Prover-70B | 70B | 84.0 | 87.7 | 92.2 (TTRL) |
| Goedel-Prover-V2-8B | 8B | 84.6 | 87.9 | 90.2 (8192) |
| &nbsp;&nbsp;+ Self-Correction | 8B | 86.7 | 89.3 | – |
| Goedel-Prover-V2-32B | 32B | 88.1 | 91.8 | 92.2 (8192) |
| &nbsp;&nbsp;+ Self-Correction | 32B | 90.4 | 92.6 | – |
| **Pythagoras-Prover-4B** | 4B | **86.07** | **88.11** | **89.75 (2048)** |
| **Pythagoras-Prover-32B** | 32B | **89.75** | **92.62** | **93.03 (2048)** |

> Pythagoras-Prover-4B exceeds DeepSeek-Prover-V2-671B's pass@8192 result (88.9%) at pass@2048 — a quarter of the budget and ~167× fewer parameters. Pythagoras-Prover-32B sets the strongest reported MiniF2F-Test pass rate without self-correction or test-time RL.

### PutnamBench (problems solved out of 672)

| # | Model | Open-source | Compute | Solved |
| :---: | --- | :---: | :---: | :---: |
| **1** | **Pythagoras-Prover** | ✓ | pass@2048 | **93** |
| **1** | **Pythagoras-Prover** | ✓ | pass@64 | **59** |
| **1** | **Pythagoras-Prover** | ✓ | pass@32 | **48** |
| 2 | Goedel-Prover-V2 (self-correction) | ✓ | pass@184 | 86 |
| 2 | Goedel-Prover-V2 (self-correction) | ✓ | pass@32 | 57 |
| 2 | Goedel-Prover-V2 | ✓ | pass@32 | 43 |
| 3 | DeepSeek-Prover-V2 | ✓ | pass@1024 | 47 |
| 3 | DeepSeek-Prover-V2 | ✓ | pass@32 | 22 |
| 4 | DSP+ | ✓ | pass@128 | 23 |
| 5 | Bourbaki | ✓ | pass@512 | 14 |
| 6 | Kimina-Prover-7B-Distill | ✓ | pass@192 | 10 |
| 7 | Self-play Theorem Prover | ✓ | pass@3200 | 8 |
| 8 | Goedel-Prover-SFT | ✓ | pass@512 | 7 |
| 9 | ABEL | ✗ | pass@596 | 7 |

> Pythagoras-Prover ranks 1st among open-source models, surpassing the previous best (Goedel-Prover-V2, 86 with self-correction) using independent restart sampling rather than iterative self-correction.

### MiniF2F-ALF (pass@32, %)

| Model | Pass@32 |
| --- | :---: |
| DeepSeek-Prover-V2-671B | 79.71 |
| Goedel-Prover-V2-8B | 82.58 |
| Goedel-Prover-V2-32B | 83.61 |
| **Pythagoras-Prover-4B** | **83.19** |
| **Pythagoras-Prover-32B** | **85.04** |

> Every prover loses accuracy under ALF mutation, confirming a non-trivial distribution shift. Pythagoras-Prover-32B retains the highest pass rate, and Pythagoras-Prover-4B matches the 8× larger Goedel-Prover-V2-32B while degrading less (2.9 vs. 4.5 points).

### Diffusion Theorem Proving (pass@32, %)

| Model | Pass@32 |
| --- | :---: |
| **Pythagoras-Prover** (autoregressive, 4B) | **86.07** |
| Pythagoras-Prover&#42; (autoregressive, 4K-context control) | 74.59 |
| **Pythagoras-Prover-Diffusion** (4B) | **63.25** |

> &#42; Autoregressive 4B re-trained at the matched 4,096-token context used for the diffusion model. The control isolates the diffusion gap as a context-length effect rather than a decoding-strategy one.

<!-- 
## 5. Models & Dataset Downloads

| Model | Download |
| --- | --- |
| Pythagoras-Prover-4B | [🤗 HuggingFace](https://huggingface.co/Pythagoras-LM/Pythagoras-Prover-4B) |
| Pythagoras-Prover-32B | [🤗 HuggingFace](https://huggingface.co/Pythagoras-LM/Pythagoras-Prover-32B) |
| Pythagoras-Prover-Diffusion-4B | [🤗 HuggingFace](https://huggingface.co/Pythagoras-LM/Pythagoras-Prover-Diffusion-4B) |

| Resource | Download |
| --- | --- |
| Training corpus (Lean-verified + ALF) | [🤗 HuggingFace](https://huggingface.co/Pythagoras-LM) |
| MiniF2F-ALF benchmark | [🤗 HuggingFace](https://huggingface.co/Pythagoras-LM) |

> All artefacts are released under the [Pythagoras-LM](https://huggingface.co/Pythagoras-LM) organisation. Confirm the exact repo slugs above match your uploaded repositories.


## 6. License

The code in this repository is released under the license specified in `LICENSE`. Model weights are subject to the license of their base models (Qwen3) and any additional model license included with the release. *(Fill in the concrete license terms before publishing.)* -->

## Citation & Contact

If you find Pythagoras-Prover useful, please cite:

```bibtex
@article{leang2026pythagoras,
  title   = {Pythagoras-Prover: Advancing Efficient Formal Proving via Augmented Lean Formalisation},
  author  = {Leang, Joshua Ong Jun and Zhao, Zheng and Stoian, Mihaela C{\u{a}}t{\u{a}}lina
             and Xu, Qiyuan and Li, Haonan and Li, Wenda and Cohen, Shay B. and Giunchiglia, Eleonora},
  journal = {arXiv preprint arXiv:XXXX.XXXXX},
  year    = {2026}
}
```


For questions, please open an issue on this repository.