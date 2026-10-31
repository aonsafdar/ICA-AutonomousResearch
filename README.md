# Autonomous Vision-Architecture Research with a Language-Model Agent: A Longitudinal Case Study

**Aon Safdar** and **Mohamed Saadeldin**
University College Dublin, Ireland

Published at the **2026 IEEE International Conference on Agentic AI (ICA 2026)**.
A companion version, *Behavioural Signatures of a Language-Model Agent in Autonomous
Vision-Architecture Research*, was accepted at the NeurIPS 2026 Workshop for Autonomous
Machine Learning Research (AutoMLR). An extended preprint with full appendices is available
on arXiv: [arXiv:2608.01995](https://arxiv.org/abs/2608.01995).

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## What this repository is

This is the companion artefact to the paper. It contains the complete, auditable trace of a
single general-purpose language-model agent (Anthropic Claude, Opus 4-series, operated through
Claude Code) that was given the full role of researcher on one open-ended problem: designing a
vision transformer that mixes information purely through channel attention, with no spatial
self-attention at any stage.

Over roughly ten weeks and about 2.4k GPU-hours the agent submitted **111 sequential
hypotheses** (35 accepted), driving the design from a 69.67% CIFAR-10 baseline to 97.25%
at 5.8M parameters, 85.07% on CIFAR-100 at 22M, and 79.83% on ImageNet-1K at 3.7 GFLOPs.
The paper's contribution is not the architecture but the **behavioural trace**: a visible
phase structure (rapid early gains, a long saturation plateau, recovery triggered by expanding
the agent's tools rather than by a stronger model), a greedy-incremental bias that is largely
workflow-induced, independent rediscovery and one overturning of established design choices,
and silent carry-over of small-scale decisions into larger regimes.

Everything needed to verify the claims in the paper and to regenerate every figure and table
is here: the human-authored instruction documents that governed the agent, the agent's
per-hypothesis research log, the parsed hypothesis table, the figure-generation scripts, the
champion configurations for each phase, and the Slurm templates used for the control hand-off.

## Contents

```
.
|-- README.md                          this file
|-- CITATION.cff                       citation metadata
|-- LICENSE                            MIT
|-- appendix.pdf                       supplementary appendix (14 pp.):
|                                        A. implementation details
|                                        B. per-hypothesis record (all 111 hypotheses)
|                                        C. tables and figures omitted from the paper for space
|-- instructions/                      human-authored documents that governed the agent
|   |-- RESEARCH_PROGRAM.md              full research-programme document
|   |-- RESEARCH_BRIEF.md                agent-facing brief with per-phase constraints
|   |-- AGENT_INSTRUCTIONS.md            one-page workflow companion (the per-hypothesis loop,
|   |                                    log template, and prohibitions)
|   |-- TOOLING.md                       tool configuration (MCP servers, GitHub CLI, W&B)
|   `-- empirical_laws_synthesis.md      agent-authored synthesis, read once per session
|-- log/                               the agent's trace
|   |-- research_log.md                  full per-hypothesis log, one entry per hypothesis in
|   |                                    the fixed template (motivation, literature context,
|   |                                    exact change, budget, result, decision)
|   `-- hypotheses.csv                   parsed projection of the log; source of every figure
|-- outputs/
|   `-- architecture_evolution.md        agent-authored architectural specification
|-- figures/
|   |-- parse_notes.py                   research_log.md -> hypotheses.csv
|   `-- build_figures.py                 hypotheses.csv -> six PDF figures
|-- configs/                           champion architecture specifications
|   |-- phase1_champion.yaml             CIFAR-10, 5.8M (end of Phase 1b)
|   |-- phase2_champion.yaml             CIFAR-100, 22M (end of Phase 2)
|   `-- phase3_baseline.yaml             ImageNet-1K baseline (Phase-2 champion architecture)
`-- slurm/                             scheduler templates
    |-- train_1gpu.slurm.template        single-GPU submission
    |-- train_2gpu.slurm.template        two-GPU submission (Phase 3)
    |-- job_watcher.sh.template          control hand-off: returns control to the agent on
    |                                    job completion (Appendix A.4)
    `-- submit_and_wait.sh.template      alternative submission utility
```

## Where each file is used in the paper

Section numbers refer to the ICA 2026 paper; "A.x", "B", "C.x" refer to `appendix.pdf`.

| File | Paper |
| --- | --- |
| `instructions/RESEARCH_PROGRAM.md`, `RESEARCH_BRIEF.md`, `AGENT_INSTRUCTIONS.md` | Sec. III (human interface), Sec. VII; App. A.5, A.7 |
| `instructions/TOOLING.md` | Sec. III (affordances); App. A.1, A.2 |
| `instructions/empirical_laws_synthesis.md` | Sec. III, Sec. VII (recommendation v); App. A.5 |
| `log/research_log.md` | Secs. III–VI (primary unit of observation); App. A.6, B |
| `log/hypotheses.csv` | Table I, Figs. 2–3; App. B, C |
| `outputs/architecture_evolution.md` | Sec. IV |
| `figures/parse_notes.py`, `build_figures.py` | Figs. 2–3; App. C |
| `configs/phase*_*.yaml` | Sec. IV, Table I; App. A.10 |
| `slurm/train_*.template` | Sec. III; App. A.2 |
| `slurm/job_watcher.sh.template`, `submit_and_wait.sh.template` | Sec. III (end-of-turn after submission); App. A.4 |

## Reproducing the analysis

Requires Python 3.10+ with `numpy` and `matplotlib`.

Regenerate the hypothesis table from the research log:

```bash
cd figures
python parse_notes.py          # writes ../log/hypotheses.csv
```

Regenerate every figure:

```bash
cd figures
python build_figures.py
```

This writes `fig_trajectory.pdf` (Fig. 2), `fig_success_rate.pdf` (Fig. 3),
`fig_attribution.pdf`, `fig_hypothesis_mix.pdf`, `fig_cross_scale.pdf`, and
`fig_bias_decomp.pdf` (Appendix C) to the same directory. The per-hypothesis longtable in
Appendix B is generated from the same CSV.

## Reproducing a training run

The configurations in `configs/` and the Slurm templates in `slurm/` document the exact
architecture hyper-parameters and the submission protocol for each phase. The training code
(model implementation, training driver, and per-run experiment configurations) is being
prepared for release and will be added to this repository; until then the configurations are
provided for inspection and audit. With the code in place, a hypothesis is reproduced with

```bash
sbatch --export=ALL,CFG=configs/phase2_champion.yaml,RUN_TAG=repro \
       slurm/train_2gpu.slurm.template
```

after setting the site-specific values at the top of the template (partition, accelerator
constraint, conda environment, data path).

## A note on site-specific values

Values that identify our compute site (cluster paths, Slurm partition and accelerator
constraint names, conda environment paths, and the experiment-tracking project name) have
been replaced with neutral placeholders such as `gpu-partition`, `ACCELERATOR_CONSTRAINT`,
`/path/to/...`, and `anon-project`. None of these affect the architecture, the training
recipe, or the analysis; they are the only edits made to the released files. The research
log, instruction documents, and hypothesis table are otherwise verbatim.

## Citation

```bibtex
@inproceedings{safdar2026autonomous,
  title     = {Autonomous Vision-Architecture Research with a Language-Model Agent:
               A Longitudinal Case Study},
  author    = {Safdar, Aon and Saadeldin, Mohamed},
  booktitle = {2026 IEEE International Conference on Agentic AI (ICA)},
  year      = {2026},
  publisher = {IEEE}
}
```

Extended preprint:

```bibtex
@article{safdar2026longhorizon,
  title   = {Long-Horizon Autonomous Architecture Research with a Language-Model Agent:
             A Behavioural Case Study},
  author  = {Safdar, Aon and Saadeldin, Mohamed},
  journal = {arXiv preprint arXiv:2608.01995},
  year    = {2026}
}
```

## Acknowledgement

This publication has emanated from research conducted with the financial support of
Taighde Éireann – Research Ireland under Grant number 18/CRT/6183.

## Contact

Aon Safdar, aon.safdar@ucdconnect.ie
