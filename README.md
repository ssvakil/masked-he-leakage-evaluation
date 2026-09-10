# Masked HE leakage evaluation

Replication package for **"Reconsidering Peak Correlation as a Security Metric
for Masked Homomorphic Encryption Implementations"** — S. S. Vakil,
Y. Farjami, University of Qom.

The paper argues that a peak correlation reported at a fixed trace budget is
insufficient as a standalone security metric, and supplies the missing numbers
for BFV. This repository contains the harness that produced them.

## Quick start

```
pip install -r requirements.txt
python3 code/sca_bfv.py --self-test
```

The self-test checks the NTT against schoolbook negacyclic multiplication, the
analytic order-d predictor against a Monte-Carlo estimate over 2e5 mask draws,
and BFV aggregation correctness. No figure in this repository is meaningful if
it fails.

## Layout

```
code/       leakage simulation, campaign, sweeps
elmo/       C project and scripts for the ELMO calibrated leakage simulator
data/       every CSV the paper's numbers come from
figures/    the four figures
paper/      LaTeX source, bibliography, compiled PDF
```

## Reproducing the results

| Result | Command | Data |
|---|---|---|
| Main table, 20 seeds | `for o in 0 5 10 15; do python3 code/sca_bfv.py --seeds 5 --seed-offset $o --traces 500 2000 8000 20000 --out-csv b_$o.csv; done` | `data/full20*.csv` |
| Threshold crossing vs masking order | `python3 code/campaign.py` | `data/campaign_tvla.csv` |
| Noise sweep | `python3 code/sigma_sweep.py` | `data/campaign_sigma.csv` |
| Degenerate-input failure mode | `python3 code/seeds20.py` | `data/seeds20.csv` |
| ELMO validation | see `elmo/README.md` | `elmo/` |

Build the paper with `bash paper/build.sh`.

## Known limitations

- **Simulation, not measurement.** No physical traces. The ELMO results use a
  simulator calibrated against real Cortex-M0 and M3 measurements, which
  removes the circularity of evaluating our own leakage model but is still not
  a device.
- **Multi-trace adversary only.** The single-trace attacks that have actually
  broken HE libraries, including against masked implementations, are outside
  the threat model, and traces-to-detection says nothing about them.
- **One compilation, one core, `-Os`.** Results are specific to it.

## Read `elmo/README.md` before trusting an ELMO number

It documents four harness faults, each of which produced a confident and wrong
answer before the data contradicted it: partitioning the TVLA on the public
input, trusting the tool's built-in leaky-instruction count, unequal execution
length between the two groups, and compiler constant folding across the two
builds. Two of them independently produced the conclusion that masking fails
at first order. It does not.

`code/sca_bfv.py` carries a fifth: a degenerate TVLA fixed input whose NTT has
a zero coefficient makes one column carry no leakage at all and the statistic
explodes. Probability 3.8% per seed, invisible at five seeds, one occurrence in
twenty. The code now rejects such inputs at draw time.

## Citation

See `CITATION.cff`.

## License

MIT, see `LICENSE`.
