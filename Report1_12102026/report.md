# BEGAD - Week 1 Report: Group Comparison (CO / MCI / AD)


## 1. Dataset overview

| Group | Participants |
|---|---|
| CO | 100 |
| MCI | 41 |
| AD | 16 |
| **Total** | **157** |

**Protocol.** 11 tasks per participant (0–10). Media names are read from the `MediaName` column of the raw exports; geometry from `TASK_GEOMETRY` in `src/begad_norm/config.py`. Tasks 0, 2, 6, 8, 10 are full-screen instruction screens, not test stimuli; tasks 3–5 are the King–Devick cards and task 9 the reading task.

| Task | Media (`MediaName`) | Geometry | Description |
|---|---|---|---|
| 0 | `Instruction Element` | 1920×1080 | Instruction screen |
| 1 | `KD_demo_highres.png` | 748×564 | King–Devick demo card (practice) |
| 2 | `Instruction Element 2` | 1920×1080 | Instruction screen |
| 3 | `KD_1_highres.png` | 746×566 | King–Devick card 1 |
| 4 | `KD_2_highres.png` | 744×525 | King–Devick card 2 |
| 5 | `KD_3_highres.png` | 737×508 | King–Devick card 3 |
| 6 | `Instruction Element` | 1920×1080 | Instruction screen |
| 7 | `lukeminen_harj.png` | 950×417 | Reading practice |
| 8 | `Instruction Element` | 1920×1080 | Instruction screen |
| 9 | `lukeminen_1.png` | 918×917 | Reading task |
| 10 | `Instruction Element 2` | 1920×1080 | Closing instruction screen |


<img src="figures/fig1_begad_dataset_summary.svg" alt="BEGAD dataset summary" width="100%">

**Figure 1.** The BEGAD dataset. (a) Participant counts by diagnostic group: healthy controls (HC, n = 100), mild cognitive impairment (MCI, n = 41), and Alzheimer's disease (AD, n = 16). (b) SOB scores (raw export field) by group (HC n = 100, MCI n = 41, AD n = 16).

## 2. 25 selected features

### Definitions of the 25 selected features

Each feature is one scalar per (participant, task).
Notation:
- `n` = number of clean fixations;
- `x, y` = fixation centers (`FixationPointX/Y (MCSpx)`, px, media-relative);
- `dur` = fixation duration (`GazeEventDuration`, ms);
- `pup` = binocular mean pupil `(PupilLeft + PupilRight) / 2` (mm);
- `(cx, cy) = (W/2, H/2)` = media center from `TASK_GEOMETRY` (config.py).
All stds use `ddof=1` (sample SD).

| Feature | Meaning | How it is computed | Unit |
|---|---|---|---|
| **Pupil - pupil size and dynamics** | | | |
| `pup_mean` | Average pupil size during the task | `mean(pup_i)` with `pup_i = (PupilLeft_i + PupilRight_i) / 2` over the clean fixations | mm |
| `pup_median` | Median pupil size | `median(pup_i)` | mm |
| `pup_max` / `pup_min` | Largest / smallest pupil size | `max` / `min` of the series | mm |
| `pup_std` | Variability of pupil size | sample SD (`ddof=1`) of the series | mm |
| `pup_slope` | Pupil trend over the task (positive = dilation, negative = constriction) | slope of the least-squares line (`np.polyfit`, degree 1) of the series over fixation order `0…n−1` | mm per fixation index |
| `pupbl_std` | Variability after baseline correction | as `pup_std` on the baseline-corrected series `((PupilLeft_i − PupilLeftBaseline) + (PupilRight_i − PupilRightBaseline)) / 2` (baselines constant per recording) | mm |
| `pupbl_slope` | Baseline-corrected pupil trend | as `pup_slope` on the baseline-corrected series | mm per fixation index |
| **Saccade - measured saccade events** | | | |
| `sac_count` | Number of saccades | `m` = number of saccade runs | count |
| `sac_dur_total` | Total time spent in saccades | sum of run durations | ms |
| `sac_dur_mean` | Average saccade duration | mean of the run durations | ms |
| `sac_dur_max` | Longest saccade | max of the run durations | ms |
| `sac_amp_mean` | Average saccade amplitude | mean of the per-fixation `SaccadicAmplitude` | deg (visual angle) |
| `sac_amp_std` | Variability of saccade amplitude | sample SD (`ddof=1`) of those amplitudes | deg |
| `sac_turn_std` | Variability of direction changes between consecutive saccades | sample SD (`ddof=1`) of `min(reldir, 360 − reldir)` from `RelativeSaccadicDirection`, folded to `[0, 180]` | deg |
| **Spatial - fixation positions and spread** | | | |
| `spa_fix_count` | Number of fixations | `n` = clean fixations remaining after cleaning | count |
| `spa_center_dist_std` | Spread of the fixations around the media center | sample SD (`ddof=1`) of the distances `sqrt((x−cx)² + (y−cy)²)` | px |
| `spa_dispersion` | Average distance of the fixations from their centroid | `mean(sqrt((x−mean(x))² + (y−mean(y))²))` | px |
| `spa_q2` | Share of fixations in quadrant q2 (right of center, upper half) | fraction of fixations with `x ≥ cx` and `y < cy` | fraction 0–1 |
| **Geometry - scanpath shape** | | | |
| `geo_scanpath_len` | Total scanpath length | sum of step lengths `amp = sqrt(dx² + dy²)`, `dx = diff(x)`, `dy = diff(y)` | px |
| `geo_sacc_amp_mean` | Average step (inferred saccade) amplitude | `mean(amp)` | px |
| `geo_nn_dist_mean` | Spacing of the fixations (how dense the scanpath is) | mean over fixations of the distance to the nearest *other* fixation | px |
| `geo_revisit_rate` | Tendency to return to already visited locations | fraction of fixations with a *later* fixation within 60 px (`REVISIT_RADIUS`; forward-looking, `triu(dist < 60, k = 1)`) | fraction 0–1 |
| **Temporal - timing** | | | |
| `tem_dur_total` | Total fixation time | sum of the fixation durations | ms |
| `tem_velocity_mean` | Average scanpath velocity | mean of `amp / half_gap` over the finite terms, `half_gap = (dur_i + dur_{i+1}) / 2` (half-dwell inter-fixation interval approximation; onset timestamps are not recorded) | px/ms |

`pupbl_std` and `pupbl_slope` are NaN for the 16 participants whose pupil baselines are missing in the raw CSVs.


Among the 25 selected features, **8** are significant (p < 0.1) in all three King–Devick cards (tasks 3, 4, 5): `spa_fix_count`, `geo_sacc_amp_mean`, `geo_revisit_rate`, `geo_nn_dist_mean`, `pup_mean`, `pup_median`, `sac_count`, `sac_dur_total`.

### Figures - features significant in all three King–Devick cards (tasks 3, 4, 5)

Each figure shows one of the 8 features across the three cards.

<img src="figures/fig_geo_nn_dist_mean.svg" alt="geo_nn_dist_mean" width="100%">

**Figure 2.** `geo_nn_dist_mean` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=11.34, p=0.0034 (CO n=100, MCI n=41, AD n=16); Task 4: H=8.21, p=0.0165 (CO n=100, MCI n=41, AD n=16); Task 5: H=7.26, p=0.0265 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_geo_revisit_rate.svg" alt="geo_revisit_rate" width="100%">

**Figure 3.** `geo_revisit_rate` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=11.58, p=0.0031 (CO n=100, MCI n=41, AD n=16); Task 4: H=10.91, p=0.0043 (CO n=100, MCI n=41, AD n=16); Task 5: H=7.52, p=0.0233 (CO n=100, MCI n=41, AD n=16)).

<img src="figures/fig_geo_sacc_amp_mean.svg" alt="geo_sacc_amp_mean" width="100%">

**Figure 4.** `geo_sacc_amp_mean` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=12.28, p=0.0022 (CO n=100, MCI n=41, AD n=16); Task 4: H=5.04, p=0.0804 (CO n=100, MCI n=41, AD n=16); Task 5: H=13.15, p=0.0014 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_pup_mean.svg" alt="pup_mean" width="100%">

**Figure 5.** `pup_mean` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=4.73, p=0.0938 (CO n=100, MCI n=41, AD n=16); Task 4: H=5.27, p=0.0718 (CO n=100, MCI n=41, AD n=16); Task 5: H=6.52, p=0.0384 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_pup_median.svg" alt="pup_median" width="100%">

**Figure 6.** `pup_median` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=5.15, p=0.0761 (CO n=100, MCI n=41, AD n=16); Task 4: H=5.50, p=0.0639 (CO n=100, MCI n=41, AD n=16); Task 5: H=6.56, p=0.0377 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_sac_count.svg" alt="sac_count" width="100%">

**Figure 7.** `sac_count` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=6.66, p=0.0357 (CO n=100, MCI n=41, AD n=16); Task 4: H=11.17, p=0.0037 (CO n=100, MCI n=41, AD n=16); Task 5: H=15.91, p=0.0004 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_sac_dur_total.svg" alt="sac_dur_total" width="100%">

**Figure 8.** `sac_dur_total` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=6.52, p=0.0385 (CO n=100, MCI n=41, AD n=16); Task 4: H=19.63, p=5.47e-05 (CO n=100, MCI n=41, AD n=16); Task 5: H=13.71, p=0.0011 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_spa_fix_count.svg" alt="spa_fix_count" width="100%">

**Figure 9.** `spa_fix_count` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=7.70, p=0.0213 (CO n=100, MCI n=41, AD n=16); Task 4: H=12.38, p=0.0021 (CO n=100, MCI n=41, AD n=16); Task 5: H=17.12, p=0.0002 (CO n=100, MCI n=41, AD n=16)).

## 4. Baseline classification experiments

Six classical baselines (Logistic Regression, SVM RBF, SVM Linear, FNN MLP 64-32, QDA, GNB) on the 25 selected features, run for three problems (CO vs MCI, CO vs AD, CO/MCI/AD) with class-imbalance handling. 

**Protocol**
- **Input design**: one sample per participant; the 25 features × 11 tasks flattened in feature-major order to **275 features** (`{feature}_t{task}`). Sample counts: CO vs MCI 141 (CO 100 / MCI 41), CO vs AD 116 (CO 100 / AD 16), CO/MCI/AD 157 (100 / 41 / 16).
- **CV**: participant-level, stratified by group, 5 folds × 20 seeds (fixed before results were seen) = **100 train/test evaluations** per (problem, model); 1800 evaluations total. Test folds have 23–33 participants depending on the problem.
- **Missing data**: 402 / 43,175 cells (0.93%), 45 / 275 columns affected (`pupbl_*` of the 16 participants without a pupil baseline, and the 2 all-NaN task-10 rows). **Median imputation per fold on the training split only**, then `StandardScaler` fitted per fold on the training split only.
- **Class weights**: Logistic Regression and both SVMs get scikit-learn `class_weight: balanced`; the FNN gets balanced `sample_weight` at fit time (MLPClassifier has no `class_weight` argument); QDA and GNB get **inverse-frequency priors** computed from the training split of each fold.
- **Decision threshold** (`threshold: 0.5`): binary predictions use the fixed default threshold 0.5 (`model.predict`); no threshold tuning. The 3-class problem uses argmax (no threshold). AUC is unaffected by the threshold.
- **Models** (fixed hyperparameters, no tuning): Logistic Regression (`max_iter=2000`), SVM RBF (`C=1, gamma='scale', probability=True`), SVM Linear (`C=1, probability=True`), FNN (MLP 64-32, relu, adam, lr=1e-3, alpha=1e-4, batch=32, max_iter=300), QDA (`solver='eigen', shrinkage=0.5`), GNB (`var_smoothing=1e-9`).
- **Metrics** (per evaluation, on the held-out fold): Acc; Sen (sensitivity of the disease class: binary = the positive class, 3-class = macro recall); Spec (binary: TN/(TN+FP), 3-class: mean per-class TN/(TN+FP)); Precision (binary: disease class, 3-class: macro); AUC (binary: ROC-AUC, 3-class: OVR macro); F1 (binary: disease class, 3-class: macro). **Primary metric: AUC.** Disease class: MCI in CO vs MCI, AD in CO vs AD. In the 3-class problem, Acc is the overall 3-class accuracy and no class is singled out.
- **Chance references** (majority-class accuracy): CO vs MCI 0.709, CO vs AD 0.862, CO/MCI/AD 0.637; AUC chance = 0.5.

### Results - mean ± std over 100 evaluations (20 seeds × 5 folds)

**CO vs MCI** (positive = MCI)

| Model | Acc | AUC | Sen | Spec | Precision | F1 |
|---|---|---|---|---|---|---|
| Logistic Regression | 0.6572±0.0720 | 0.6361±0.0873 | 0.4472±0.1622 | 0.7430±0.0935 | 0.4222±0.1332 | 0.4244±0.1267 |
| SVM RBF | **0.7282±0.0706** | **0.7384±0.0945** | 0.4585±0.1819 | 0.8390±0.0863 | **0.5534±0.1645** | **0.4829±0.1492** |
| SVM Linear | 0.6447±0.0736 | 0.5975±0.0968 | 0.4068±0.1721 | 0.7420±0.0937 | 0.3954±0.1389 | 0.3906±0.1363 |
| FNN (MLP 64-32) | 0.7178±0.0763 | 0.6954±0.1013 | 0.4572±0.1590 | 0.8245±0.0886 | 0.5349±0.1787 | 0.4796±0.1429 |
| QDA | 0.6962±0.0460 | 0.5535±0.0943 | 0.0932±0.0957 | **0.9435±0.0563** | 0.3493±0.3688 | 0.1401±0.1378 |
| GNB | 0.6301±0.0916 | 0.6429±0.1028 | **0.5910±0.1596** | 0.6455±0.1148 | 0.4141±0.1099 | 0.4808±0.1161 |

**CO vs AD** (positive = AD)

| Model | Acc | AUC | Sen | Spec | Precision | F1 |
|---|---|---|---|---|---|---|
| Logistic Regression | 0.8588±0.0539 | 0.8571±0.0996 | 0.3758±0.2886 | 0.9365±0.0517 | 0.4455±0.3286 | 0.3839±0.2636 |
| SVM RBF | 0.8619±0.0266 | 0.8428±0.0903 | 0.0558±0.1330 | 0.9910±0.0218 | 0.1300±0.3157 | 0.0750±0.1753 |
| SVM Linear | **0.8687±0.0474** | **0.8638±0.0993** | 0.3483±0.2544 | 0.9520±0.0443 | **0.4977±0.3617** | 0.3861±0.2575 |
| FNN (MLP 64-32) | 0.8569±0.0532 | 0.7141±0.1562 | 0.3067±0.2515 | 0.9450±0.0505 | 0.4257±0.3569 | 0.3374±0.2618 |
| QDA | 0.8610±0.0158 | 0.5937±0.2230 | 0.0000±0.0000 | **0.9985±0.0086** | 0.0000±0.0000 | 0.0000±0.0000 |
| GNB | 0.8465±0.0819 | 0.7965±0.1592 | **0.5975±0.3004** | 0.8870±0.0751 | 0.4780±0.2719 | **0.5120±0.2552** |

**CO/MCI/AD** (3-class, macro metrics)

| Model | Acc | AUC | Sen | Spec | Precision | F1 |
|---|---|---|---|---|---|---|
| Logistic Regression | 0.5915±0.0753 | 0.6982±0.0715 | 0.4519±0.1000 | 0.7408±0.0486 | 0.4567±0.1258 | 0.4440±0.1025 |
| SVM RBF | **0.6478±0.0735** | **0.7583±0.0706** | 0.4588±0.0764 | **0.7755±0.0495** | 0.4359±0.1201 | 0.4364±0.0828 |
| SVM Linear | 0.5736±0.0751 | 0.6913±0.0757 | 0.4265±0.0937 | 0.7208±0.0492 | 0.4374±0.1316 | 0.4212±0.1010 |
| FNN (MLP 64-32) | 0.6471±0.0709 | 0.7008±0.0865 | **0.4751±0.0916** | 0.7650±0.0507 | **0.4815±0.1193** | **0.4681±0.0970** |
| QDA | 0.6203±0.0447 | 0.5645±0.1068 | 0.3411±0.0347 | 0.6735±0.0278 | 0.3105±0.1129 | 0.2955±0.0479 |
| GNB | 0.5685±0.0912 | 0.6575±0.0889 | 0.4697±0.1081 | 0.7564±0.0557 | 0.4763±0.1223 | 0.4574±0.1053 |

### Best mean AUC per problem

| Problem | Best model | AUC |
|---|---|---|
| CO vs MCI | SVM RBF | 0.7384±0.0945 |
| CO vs AD | SVM Linear | 0.8638±0.0993 |
| CO/MCI/AD | SVM RBF | **0.7583±0.0706** |

### Notes

- **With the fixed 0.5 threshold, probability-calibrated models keep useful hard predictions**: SVM RBF reaches Sen 0.459 and F1 0.483 on CO vs MCI; GNB has the highest binary Sen (0.591 on CO vs MCI, 0.598 on CO vs AD) at the cost of Spec (0.646 / 0.887). QDA's binary predictions still collapse on CO vs AD (Sen = 0).
- **3-class metrics are macro averages**: Sen/Precision/Spec/F1 are averaged over the three classes and AUC is the one-vs-rest macro AUC; no class is singled out as the disease class there (unlike the binary problems). Acc is the overall 3-class accuracy.
- **AUC is threshold-independent**; class weights and priors barely move the ranking. SVM Linear is exactly invariant to class weights here: the p > n standardized training folds are linearly separable, so the max-margin solution does not depend on class weights. QDA/GNB binary AUC is likewise unaffected by the priors, which shift scores by a constant and preserve the ROC ranking.
- Accuracy is within one standard deviation of the majority-class chance level in every problem; AUC is the informative metric. Large standard deviations reflect the small test folds (23–33 participants) and the small AD class (16 participants).
- The 25 features were screened on the same cohort (Kruskal–Wallis, p < 0.1 in ≥ 3 tasks), so CV performance carries **in-sample selection bias**.
- **The archived Youden run** (`outputs/baseline_balanced_youden/`) used per-fold Youden-threshold tuning. AUC and all 3-class metrics are identical between the two runs; only the binary hard-decision metrics (Acc/Sen/Spec/Precision/F1) changed.

**Reproducibility.** Code: `src/begad_norm/baseline/run_baselines.py` (+ `aggregate_results.py`), config `src/begad_norm/baseline/config.yaml`. Machine-readable outputs in `outputs/baseline_balanced/` (results detail, summary, missingness, provenance, run log). Archived runs: `outputs/baseline_balanced_macro/` (first run, identical metrics, `recall` column), `outputs/baseline_balanced_adpos/` (superseded intermediate run with AD-vs-rest 3-class metrics), `outputs/baseline_balanced_youden/` (superseded run with per-fold Youden threshold tuning). AUC and all 3-class metrics are identical across all four runs; the binary hard-decision metrics differ only between the Youden run and the fixed-0.5 runs. Re-run: `uv run python -m begad_norm.baseline.run_baselines --config src/begad_norm/baseline/config.yaml`.
