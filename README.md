# Verifiable rating aggregation

Replication package for **"Privacy-Preserving and Verifiable Product Rating
Aggregation Using Threshold Homomorphic Encryption with Distributed Key
Generation"** — S. S. Vakil, Y. Farjami, University of Qom.

Threshold Paillier with dealerless key generation, blind-RSA tokens, an
OR-based range proof over `{1..5}`, and publicly verifiable partial
decryption. Everything here is implemented, self-tested and measured.

## Quick start

```
pip install -r requirements.txt        # numpy, pandas, scipy, matplotlib, gmpy2
python3 code/ratingagg.py --self-test  # run this first
python3 code/dkg.py      --self-test
```

Nothing else in this repository means anything if the self-tests fail. They
check homomorphic addition, threshold reconstruction against centralised
decryption, rejection of a tampered partial decryption, rejection of an
out-of-domain rating, blind-signature round-trip, and — in `dkg.py` — that the
distributed product reconstructs and that the biprimality test accepts a
biprime and rejects a non-biprime.

## Layout

```
code/       implementation and measurement scripts
data/       every JSON the paper's numbers come from
figures/    the five figures, vector PDF and 600-dpi PNG
paper/      LaTeX source, bibliography, compiled PDF
proverif/   symbolic model  (SEE THE WARNING BELOW)
```

## Reproducing the results

| Result in the paper | Command | Data |
|---|---|---|
| Per-operation costs (Table 3) | `python3 code/ratingagg.py --bits 2048 3072` | `data/bench.json` |
| Committee-size scaling (Table 5) | `python3 code/auth_sweep.py` | `data/auth_sweep.json` |
| Dealerless setup (Table 6) | `python3 code/dkg.py --parties 3 5 9 --prime-bits 128 256 --helper-bits 1024` | `data/dkg_small.json` |
| " (512-bit row) | `python3 code/dkg.py --parties 3 9 --prime-bits 512 --helper-bits 2048` | `data/dkg_512.json` |
| End-to-end validation (Table 7) | `python3 code/e2e.py` | `data/e2e.json` |
| Figures | `python3 code/pfigs.py && python3 code/dkgfig.py` | `figures/` |

Build the paper with `bash paper/build.sh`. A single `pdflatex` pass always
reports undefined citations; four steps are required and the script runs them.

## Environment

All timings were taken on an Intel Xeon at 2.10 GHz with **one** available
core, 3.9 GB RAM, Linux 6.18, Python 3.12.3, gmpy2 2.3.1. Absolute numbers
will differ on your machine; the composition of a round should not.

## Known limitations, stated because they bound the claims

- **Single core.** Multi-core figures in the paper are arithmetic projections,
  not measurements, and are labelled as such.
- **Dealerless generation is semi-honest.** The zero-knowledge layer that
  makes Boneh–Franklin robust against a malicious party is not implemented, so
  the reported setup cost is a lower bound.
- **The 2048-bit generation was not run to completion.** Its cost is a
  projection from a measured per-candidate cost and an analytic candidate
  count of about 1.26e5.
- **Python.** Absolute constants would change in C; the 99.9% share of the
  round taken by proof verification is a ratio and should not.
- **`data/` records what each run measured but earlier versions of `dkg.py`
  did not record the helper modulus used.** It does now. If you compare
  against an older JSON, check which helper modulus produced it — two runs at
  different helper sizes are not comparable, and mixing them once put two
  contradictory tables into a draft of this paper.

## ⚠ The ProVerif model is out of date

`proverif/` is a placeholder. The model in the submitted analysis encodes
**blind Schnorr**, and the protocol now uses **blind RSA with a one-time
key**, after the ROS attack made the former unsound. Update the model before
publishing this repository, or remove the directory and say in the paper that
the model accompanies the earlier design.

## Three flaws the symbolic analysis did not find

Recorded because they are the paper's methodological point and because anyone
building on this should look for the same class of problem:

1. **Blind Schnorr under concurrency.** Broken by the ROS attack
   (Benhamouda et al., EUROCRYPT 2021). Abe–Okamoto is broken by the same
   result and is not a repair; we use blind RSA.
2. **A long-term signing key on the submission.** Verifying it requires the
   buyer's public key to travel with the rating, which destroys the
   unlinkability the blind token exists to provide. Replaced by a one-time key
   certified by the token.
3. **Per-authority ledgers.** With a freely chosen authority, one token can be
   spent once per authority. The handling authority is now fixed by
   `k = H(sigma) mod n`, so a local ledger suffices and no consensus is needed.

All three turn on multiplicity — of sessions, of identifiers, of replicas of a
state the model treats as one — which is exactly what a symbolic abstraction
discards.

## Citation

See `CITATION.cff`.

## License

MIT, see `LICENSE`.
