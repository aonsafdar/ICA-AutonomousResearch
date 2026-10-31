# Supplementary materials

This repository accompanies the paper "Autonomous Vision-Architecture Research
with a Language-Model Agent: A Longitudinal Case Study" by Aon Safdar and
Mohamed Saadeldin (University College Dublin), published at IEEE ICA 2026.

It contains the implementation-and-appendix PDF (`appendix.pdf`) and everything
needed to audit the per-hypothesis trace described in the paper and to
regenerate every figure and table. The full training code
(model implementation, training driver, and per-run experiment configurations)
will be released in a future update.

## Contents

```
.
|-- README.md                                  (this file)
|-- appendix.pdf                               implementation details (App. A),
|                                              full per-hypothesis record (App. B),
|                                              and the tables/figures omitted from
|                                              the main paper for space (App. C)
|-- instructions/
|   |-- RESEARCH_PROGRAM.md                    full research-programme document
|   |-- RESEARCH_BRIEF.md                      shorter agent-facing brief that
|   |                                          points to RESEARCH_PROGRAM.md
|   |                                          and adds per-phase constraints
|   |-- AGENT_INSTRUCTIONS.md                  one-page workflow companion
|   |-- TOOLING.md                             tooling-configuration document
|   |                                          (MCP servers, GitHub CLI, etc.)
|   `-- empirical_laws_synthesis.md            agent-authored synthesis read
|                                              alongside the log in long sessions
|                                              (Section 3.3 of the paper)
|-- log/
|   |-- research_log.md                        full per-hypothesis log
|   |                                          (one entry per hypothesis in the
|   |                                          seven-field template described
|   |                                          in Appendix A of the paper)
|   `-- hypotheses.csv                         parsed projection of the log
|                                              used to generate every figure
|-- outputs/
|   `-- architecture_evolution.md              agent-authored architectural
|                                              specification, named in
|                                              Section 4.4 as one of the
|                                              by-products of the loop
|-- figures/
|   |-- parse_notes.py                         log -> CSV
|   `-- build_figures.py                       CSV -> six PDF figures
|-- configs/
|   |-- phase1_champion.yaml                   architecture spec, end of Phase 1
|   |-- phase2_champion.yaml                   architecture spec, end of Phase 2
|   `-- phase3_baseline.yaml                   architecture spec, Phase 3 baseline
                                               (P2-H26 arch on ImageNet-1K)
`-- slurm/
    |-- train_1gpu.slurm.template              single-GPU submission template
    |-- train_2gpu.slurm.template              two-GPU submission template
    |-- job_watcher.sh.template                control-hand-off watcher
    |                                          (described in Appendix A.4)
    `-- submit_and_wait.sh.template            alternative submission utility
                                               (kept for completeness; the
                                               watcher is the recommended
                                               control-hand-off mechanism)
```

## Reproducing the analysis

To regenerate `log/hypotheses.csv` from the research log:

```
cd figures
python parse_notes.py
```

To regenerate every figure in Section 5 of the paper:

```
cd figures
python build_figures.py
```

This writes `fig_trajectory.pdf`, `fig_success_rate.pdf`, `fig_attribution.pdf`,
`fig_hypothesis_mix.pdf`, `fig_cross_scale.pdf`, and `fig_bias_decomp.pdf` to
the same directory. The `tables/full_hyp_log.tex` longtable in Appendix B is
similarly regenerable from the CSV; the generation snippet is documented in the
paper's source-control history.

## Reproducing a training run

The training code is not yet included in this bundle. The configurations
in `configs/` and the Slurm templates in `slurm/` document the exact
hyperparameters and submission protocol used for each phase. With the training
code (released upon acceptance), the workflow to reproduce a hypothesis is:

```
sbatch --export=ALL,CFG=configs/phaseX_champion.yaml,RUN_TAG=repro \
       slurm/train_2gpu.slurm.template
```

The Slurm template expects to be customised at the top for the target site
(partition name, accelerator constraint, conda environment path, data path).

## Map from supplementary files to paper sections

| File | Referenced in |
| --- | --- |
| `instructions/RESEARCH_PROGRAM.md`, `RESEARCH_BRIEF.md`, `AGENT_INSTRUCTIONS.md` | Section 3.3, Appendix A.5 |
| `instructions/TOOLING.md`                                  | Appendix A.1, A.2 |
| `instructions/empirical_laws_synthesis.md`                 | Sections 3.3, 4.4 |
| `log/research_log.md`                                      | Sections 3.1, 4.4, Appendix A.6, B |
| `log/hypotheses.csv`                                       | All figures, Appendix B |
| `outputs/architecture_evolution.md`                        | Section 4.4 |
| `figures/parse_notes.py`, `build_figures.py`               | All figures |
| `configs/phase*_*.yaml`                                    | Section 4.3, Appendix A.10 |
| `slurm/train_*.template`                                   | Section 3.2, Appendix A.2 |
| `slurm/job_watcher.sh.template`                            | Appendix A.4 |
| `slurm/submit_and_wait.sh.template`                        | Appendix A.4 |

## Notes

Some site-specific identifiers (user IDs, cluster paths, partition names,
accelerator generations, and experiment-tracking project names) are shown as
neutral placeholders in this bundle. The full training code, with identifiers
restored, will be released in a future update.
