# Noise-robust, exactly lossless RDHEI by multi-key selection

Code and result files for the paper

> S. Singh and U. Singh, **"Noise-Robust and Exactly Lossless Reversible Data Hiding in Encrypted Images via Scalable Multi-Key Selection"**, submitted to *Information Sciences*.

The scheme embeds a message while it encrypts a grey-scale image. Each $B\times B$ block is encrypted with the AES-CTR key stream of one of $K$ keys, and the choice of key carries $\log_2 K$ message bits. The receiver holds all keys and identifies the key of every block from the regularity of natural images, using either a training-free iterative rule (Rule-it) or a gradient-boosted classifier trained with noise (HGB-it). A Reed–Solomon layer, sized by a closed-loop test at the sender, corrects the remaining decision errors, so that image and message are recovered bit-exactly. An HMAC-SHA256 tag over the marked ciphertext detects tampering.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR-GITHUB-USERNAME/rdhei-multikey-selection/blob/main/notebooks/RDHEI_Colab.ipynb)

## Contents

```
rdhei/core.py                 the scheme: key streams (repeated RC4 / AES-CTR), embedding, features, receivers
                              (KNN, DT, RF, RF noise-aug, Rule-TV, Rule-it, HGB-it), Reed–Solomon layer, test images
experiments/                  one script per experiment of the paper (table below) and run_all.sh
notebooks/RDHEI_Colab.ipynb   Colab companion: set-up, demonstration, experiments, tables
notebooks/Exp8_public_datasets_Colab.ipynb   Experiment 8 on BOSSbase, BOWS-2 and UCID
scripts/make_*.py             regenerate every table (tables/*.tex) and figure (figures/*.pdf) of the paper from data/
scripts/compare_with_paper.py compare a new run with the result files of the paper
data/                         the result files behind every number of the paper, and the test-image checksums
```

## Quick start

**Colab.** Open `notebooks/RDHEI_Colab.ipynb` (badge above), set `REPO_URL` in the first cell and run the cells in order.

**Locally** (Python 3.13 and the versions in `requirements.txt` reproduce the paper's environment):

```bash
git clone https://github.com/YOUR-GITHUB-USERNAME/rdhei-multikey-selection.git
cd rdhei-multikey-selection
pip install -r requirements.txt
bash experiments/run_all.sh quick        # seconds: two checks, compared with data/
bash experiments/run_all.sh              # all test-set-A experiments (several hours on a two-core CPU)
python scripts/make_results.py           # tables/ and figures/ from data/ (likewise the other make_*.py)
```

Each experiment can also be run on its own from any working folder, e.g. `mkdir work && cd work && python ../experiments/exp_hgbabl.py`. The results are written to `results/` in that folder, and `python scripts/compare_with_paper.py work/results` compares them with `data/`.

## Experiments and the paper

| Paper | Script | Result files (`data/`) |
|---|---|---|
| Exp. 1, Table 6 (clean images, receivers) | `exp_final.py`, `exp_e1runs.py`, `exp_k2.py` | `E1_main_summary`, `E2_ablation`, `E2_ablation_runs`, `E1b_k2_trained` |
| Exp. 2, 3, 4, 5; Tables 8, 10, 12, 13; Figs. 7–9 | `exp_p2.py` | `A_decoders`, `A_decoders_runs`, `B_blocksize`, `C_ecc_frontier`, `D_noise_ecc`, `C3_ladder_time` |
| Exp. 2b, Table 9 (receiver ablation) | `exp_hgbabl.py` | `A2_receiver_ablation_runs` |
| Exp. 5b, Table 11 (noise, smaller blocks) | `exp_noise_small.py` | `D2_noise_ecc_small` |
| Exp. 6, Fig. 8(a) (number of keys) | `exp_final.py` | `E3a_keys` |
| Exp. 7, Table 7 (test set B) | `exp7_generalisation.py` | `E4_generalisation` |
| Exp. 8, Tables 14–15, Figs. 10–11 (BOSSbase, BOWS-2, UCID) | `notebooks/Exp8_public_datasets_Colab.ipynb` | `D1_*`, `D2_lossless_*`, `D3_*` (saved by the notebook as `R1_*`, `R2_*`, `R3_*`) |
| Exp. 9, Section 6.7 (ciphertext corruption) | `exp_final.py` | `E5_corruption` |
| Section 4.6 (interleaver check) | `exp_abl.py` | `A_interleaver_ablation` |
| Section 6.7, Table 17 (timing); sender cost | `exp_final.py`, `exp_k2.py`, `exp_p2.py` | `E7_timing`, `C3b_ladder_time_precise`, `C3_ladder_time`, `HGB_n_iter` |
| Section 7.1, Table 18 (statistics); χ² over 900 key sets | `exp_final.py`, `chi2_mc.py` | `S1_S2_security`, `S6_chi2_keysets` |
| Section 7.1, Table 19 (NIST SP 800-22) | `setup_nist.sh`, `nist_all.py`, `nist_control.py` | `S2_nist_official`, `S2_nist_control` |
| Section 7.2, Table 20, Fig. 13 (attacks) | `exp_final.py`, `attack_chance.py` | `S4_attacks`, `S5_attack_chance` |
| Section 7.3, Table 21 (key sensitivity) | `exp_final.py` | `S3_npcr_uaci` |
| Section 7.4, Table 22 (integrity tag) | `integrity_new.py` | `I1_integrity` |
| Section 8 (compress-then-encrypt baseline) | `compress_baseline.py` | `B0_compress_baseline` |
| Fig. 1 (block gradient) | `scripts/make_gradient.py` | `jetplane_gray_512.png` |

All seeds are fixed in the scripts. Statistical tests (paired Wilcoxon, over images) are computed by `scripts/make_results.py` and `scripts/make_datasets.py`.

## Images

The images are downloaded automatically and are not part of this repository.

* **Test set A** (nine native 512×512 images): Jetplane, Boat, Peppers, Woman (blonde), Baboon, Walkbridge, Pirate and Lake from the [standard test images collection](https://github.com/mohammadimtiazz/standard-test-images-for-Image-Processing), and Barbara from the Set12 benchmark ([KAIR](https://github.com/cszn/KAIR)). `rdhei/core.py` checks every file against the SHA-256 checksums in `data/test_set_A_sources.csv`. The Lena image is not used.
* **Training** (disjoint from every test image): Living room and Woman (dark hair) from the same collection, Fruits, Tulips, the Middlebury 2014 Motorcycle image and the astronaut, coffee, chelsea, rocket and camera images of scikit-image.
* **Test set B** (Experiment 7): scikit-image textures, retina, immunohistochemistry, Hubble deep field and Moon, and the 24 Kodak images (downloaded from r0k.us if reachable).
* **Experiment 8**: BOSSbase 1.01, BOWS-2 and UCID, downloaded by the notebook; 30 images per data set are drawn with a fixed seed.

## Notes on the implementation

* Experiments 1, 6, 7 and 9 use the repeated per-key RC4 stream of earlier key-selection schemes; Experiments 2–5 and 8 use the AES-CTR stream.
* Experiments 7 and 8 were run with an earlier eleven-image training set (`RDHEI_TRAIN_SET=exp7`, set automatically by `exp7_generalisation.py`). The Experiment 8 notebook forms the AES-CTR counter block as ⟨i⟩₆₄‖⟨q⟩₆₄ instead of ⟨ν⟩₆₄‖⟨i⟩₃₂‖⟨q⟩₃₂. Both layouts give a distinct counter for every block and word (Section 5.3 of the paper).
* The experiments use the nonce ν = 0 and one fixed interleaver permutation. A deployment must use a fresh nonce per image and a keyed permutation.
* The NIST SP 800-22 reference implementation is built by `experiments/setup_nist.sh` from the [GitHub mirror](https://github.com/terrillmoore/NIST-Statistical-Test-Suite) used for the paper.
* Learned receivers (KNN, DT, RF, HGB) depend on the numpy and scikit-learn versions. With the versions in `requirements.txt` the results reproduce `data/` exactly (checked for Experiment 1 and the quick checks). With other versions the error counts of the learned receivers can differ slightly; for example, with scikit-learn 1.9.1 and numpy 2.5.3 the noise-augmented RF of Table 6 makes 26 instead of 24 errors. The training-free receivers and the cryptographic tests are unaffected.

## Citation

See `CITATION.cff`. Until the paper is published, please cite it as submitted to *Information Sciences*.

## Licence

The code is released under the MIT licence (`LICENSE`). The test images, data sets and the NIST test suite are third-party material under their own terms and are not redistributed here.

## Contact

Sudhir Singh, Indira Gandhi Delhi Technical University for Women, Delhi, India (sudhirsingh@igdtuw.ac.in).
