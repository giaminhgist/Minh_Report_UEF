# BEGAD — Week 1 Report: Group Comparison (CO / MCI / AD)

## 1. Dataset overview

| Group | Participants |
|---|---|
| CO | 100 |
| MCI | 41 |
| AD | 16 |
| **Total** | **157** |

**Protocol**: 11 tasks per participant.

| Task | Media (`MediaName`) | Description |
|---|---|---|
| 0 | `Instruction Element` | Instruction screen |
| 1 | `KD_demo_highres.png` | King–Devick demo card (practice) |
| 2 | `Instruction Element 2` | Instruction screen |
| 3 | `KD_1_highres.png` | King–Devick card 1 |
| 4 | `KD_2_highres.png` | King–Devick card 2 |
| 5 | `KD_3_highres.png` | King–Devick card 3 |
| 6 | `Instruction Element` | Instruction screen |
| 7 | `lukeminen_harj.png` | Reading practice |
| 8 | `Instruction Element` | Instruction screen |
| 9 | `lukeminen_1.png` | Reading task |
| 10 | `Instruction Element 2` | Closing instruction screen |


## 2. 25 selected features

### Definitions of the 25 selected features

Each feature is one scalar per (participant, task). Notation: `n` = number of
clean fixations; `x, y` = fixation centers (`FixationPointX/Y (MCSpx)`, px,
media-relative); `dur` = fixation duration (`GazeEventDuration`, ms); `pup` =
binocular mean pupil `(PupilLeft + PupilRight) / 2` (mm); `(cx, cy) = (W/2,
H/2)` = media center from `TASK_GEOMETRY` (config.py). All stds use `ddof=1`
(sample SD). The full 72-feature dictionary: `EDA/features_dictionary.md`.

| Feature | Meaning | How it is computed | Unit |
|---|---|---|---|
| **Pupil — pupil size and dynamics** | | | |
| `pup_mean` | Average pupil size during the task | `mean(pup_i)` with `pup_i = (PupilLeft_i + PupilRight_i) / 2` over the clean fixations | mm |
| `pup_median` | Median pupil size | `median(pup_i)` | mm |
| `pup_max` / `pup_min` | Largest / smallest pupil size | `max` / `min` of the series | mm |
| `pup_std` | Variability of pupil size | sample SD (`ddof=1`) of the series | mm |
| `pup_slope` | Pupil trend over the task (positive = dilation, negative = constriction) | slope of the least-squares line (`np.polyfit`, degree 1) of the series over fixation order `0…n−1` | mm per fixation index |
| `pupbl_std` | Variability after baseline correction | as `pup_std` on the baseline-corrected series `((PupilLeft_i − PupilLeftBaseline) + (PupilRight_i − PupilRightBaseline)) / 2` (baselines constant per recording) | mm |
| `pupbl_slope` | Baseline-corrected pupil trend | as `pup_slope` on the baseline-corrected series | mm per fixation index |
| **Saccade — measured saccade events** | | | |
| `sac_count` | Number of saccades | `m` = number of saccade runs | count |
| `sac_dur_total` | Total time spent in saccades | sum of run durations | ms |
| `sac_dur_mean` | Average saccade duration | mean of the run durations | ms |
| `sac_dur_max` | Longest saccade | max of the run durations | ms |
| `sac_amp_mean` | Average saccade amplitude | mean of the per-fixation `SaccadicAmplitude`| deg (visual angle) |
| `sac_amp_std` | Variability of saccade amplitude | sample SD (`ddof=1`) of those amplitudes | deg |
| `sac_turn_std` | Variability of direction changes between consecutive saccades | sample SD (`ddof=1`) of `min(reldir, 360 − reldir)` from `RelativeSaccadicDirection`, folded to `[0, 180]` | deg |
| **Spatial — fixation positions and spread** | | | |
| `spa_fix_count` | Number of fixations | `n` = clean fixations remaining after cleaning | count |
| `spa_center_dist_std` | Spread of the fixations around the media center | sample SD (`ddof=1`) of the distances `sqrt((x−cx)² + (y−cy)²)` | px |
| `spa_dispersion` | Average distance of the fixations from their centroid | `mean(sqrt((x−mean(x))² + (y−mean(y))²))` | px |
| `spa_q2` | Share of fixations in quadrant q2 (right of center, upper half) | fraction of fixations with `x ≥ cx` and `y < cy` | fraction 0–1 |
| **Geometry — scanpath shape** | | | |
| `geo_scanpath_len` | Total scanpath length | sum of step lengths `amp = sqrt(dx² + dy²)`, `dx = diff(x)`, `dy = diff(y)` | px |
| `geo_sacc_amp_mean` | Average step (inferred saccade) amplitude | `mean(amp)` | px |
| `geo_nn_dist_mean` | Spacing of the fixations (how dense the scanpath is) | mean over fixations of the distance to the nearest *other* fixation | px |
| `geo_revisit_rate` | Tendency to return to already visited locations | fraction of fixations with a *later* fixation within 60 px (`REVISIT_RADIUS`; forward-looking, `triu(dist < 60, k = 1)`) | fraction 0–1 |
| **Temporal — timing** | | | |
| `tem_dur_total` | Total fixation time | sum of the fixation durations | ms |
| `tem_velocity_mean` | Average scanpath velocity | mean of `amp / half_gap` over the finite terms, `half_gap = (dur_i + dur_{i+1}) / 2` (half-dwell inter-fixation interval approximation; onset timestamps are not recorded) | px/ms |


## 3. Figures — features significant in all three King–Devick cards (tasks 3, 4, 5)

Of the 25 selected features, **8** have p < 0.1 in all three King–Devick
cards. Each figure shows one feature across the three cards.

<img src="figures/fig_geo_nn_dist_mean.svg" alt="geo_nn_dist_mean" width="100%">

**Figure 1.** `geo_nn_dist_mean` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=11.34, p=0.0034 (CO n=100, MCI n=41, AD n=16); Task 4: H=8.21, p=0.0165 (CO n=100, MCI n=41, AD n=16); Task 5: H=7.26, p=0.0265 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_geo_revisit_rate.svg" alt="geo_revisit_rate" width="100%">

**Figure 2.** `geo_revisit_rate` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=11.58, p=0.0031 (CO n=100, MCI n=41, AD n=16); Task 4: H=10.91, p=0.0043 (CO n=100, MCI n=41, AD n=16); Task 5: H=7.52, p=0.0233 (CO n=100, MCI n=41, AD n=16)).

<img src="figures/fig_geo_sacc_amp_mean.svg" alt="geo_sacc_amp_mean" width="100%">

**Figure 3.** `geo_sacc_amp_mean` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=12.28, p=0.0022 (CO n=100, MCI n=41, AD n=16); Task 4: H=5.04, p=0.0804 (CO n=100, MCI n=41, AD n=16); Task 5: H=13.15, p=0.0014 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_pup_mean.svg" alt="pup_mean" width="100%">

**Figure 4.** `pup_mean` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=4.73, p=0.0938 (CO n=100, MCI n=41, AD n=16); Task 4: H=5.27, p=0.0718 (CO n=100, MCI n=41, AD n=16); Task 5: H=6.52, p=0.0384 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_pup_median.svg" alt="pup_median" width="100%">

**Figure 5.** `pup_median` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=5.15, p=0.0761 (CO n=100, MCI n=41, AD n=16); Task 4: H=5.50, p=0.0639 (CO n=100, MCI n=41, AD n=16); Task 5: H=6.56, p=0.0377 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_sac_count.svg" alt="sac_count" width="100%">

**Figure 6.** `sac_count` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=6.66, p=0.0357 (CO n=100, MCI n=41, AD n=16); Task 4: H=11.17, p=0.0037 (CO n=100, MCI n=41, AD n=16); Task 5: H=15.91, p=0.0004 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_sac_dur_total.svg" alt="sac_dur_total" width="100%">

**Figure 7.** `sac_dur_total` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=6.52, p=0.0385 (CO n=100, MCI n=41, AD n=16); Task 4: H=19.63, p=5.47e-05 (CO n=100, MCI n=41, AD n=16); Task 5: H=13.71, p=0.0011 (CO n=100, MCI n=41, AD n=16)). 

<img src="figures/fig_spa_fix_count.svg" alt="spa_fix_count" width="100%">

**Figure 8.** `spa_fix_count` differs across CO / MCI / AD in all three King–Devick cards (Kruskal–Wallis: Task 3: H=7.70, p=0.0213 (CO n=100, MCI n=41, AD n=16); Task 4: H=12.38, p=0.0021 (CO n=100, MCI n=41, AD n=16); Task 5: H=17.12, p=0.0002 (CO n=100, MCI n=41, AD n=16)).

## 4. Baseline classification experiments

Six classical baselines (Logistic Regression, SVM RBF, SVM Linear, FNN MLP
64-32, QDA, GNB) on the 25 selected features, run for three problems
(CO vs MCI, CO vs AD, CO/MCI/AD). Protocol: participant-level stratified
4-fold CV × 10 seeds (1–10) = 40 train/test evaluations per (problem, model);
`StandardScaler` fitted per fold on the training split; metrics
Acc / Sen / Spec / AUC / F1 with **AUC as the primary metric**; mean ± std
over the 40 evaluations.

**Input design**: one sample per participant — the 25 selected features × 11
tasks flattened to **275 features** (`{feature}_t{task}`, feature-major);
median imputation fitted per fold on the training split (0.93% of cells
missing, 18/157 participants affected; `outputs/baseline/missingness.csv`).
Sample counts per problem: CO vs MCI 141, CO vs AD 116, CO/MCI/AD 157.


### Results — mean ± std over 40 evaluations

| Problem | Model | Acc | Sen | Spec | AUC | F1 |
|---|---|---|---|---|---|---|
| CO vs MCI | Logistic Regression | 0.6680±0.0631 | 0.4020±0.1537 | 0.7780±0.0915 | 0.6632±0.0902 | 0.4030±0.1275 |
| CO vs MCI | SVM RBF | 0.7143±0.0170 | 0.0250±0.0494 | 0.9970±0.0140 | 0.7334±0.0818 | 0.0435±0.0833 |
| CO vs MCI | SVM Linear | 0.6383±0.0713 | 0.4023±0.1566 | 0.7360±0.1000 | 0.6248±0.1028 | 0.3843±0.1249 |
| CO vs MCI | FNN (MLP 64-32) | 0.7384±0.0593 | 0.4286±0.1634 | 0.8660±0.0750 | 0.7052±0.0825 | 0.4747±0.1483 |
| CO vs MCI | QDA | 0.6981±0.0417 | 0.0857±0.0804 | 0.9490±0.0462 | 0.5678±0.0815 | 0.1345±0.1212 |
| CO vs MCI | GNB | 0.6534±0.0911 | 0.5748±0.1540 | 0.6860±0.1074 | 0.6462±0.1016 | 0.4909±0.1194 |

| Problem | Model | Acc | Sen | Spec | AUC | F1 |
| CO vs AD | Logistic Regression | 0.8698±0.0394 | 0.2687±0.2147 | 0.9660±0.0346 | 0.8518±0.0868 | 0.3309±0.2490 |
| CO vs AD | SVM RBF | 0.8621±0.0000 | 0.0000±0.0000 | 1.0000±0.0000 | 0.8588±0.1026 | 0.0000±0.0000 |
| CO vs AD | SVM Linear | 0.8733±0.0458 | 0.3563±0.2184 | 0.9560±0.0403 | 0.8578±0.0919 | 0.4166±0.2338 |
| CO vs AD | FNN (MLP 64-32) | 0.8741±0.0439 | 0.2812±0.2414 | 0.9690±0.0430 | 0.7305±0.1617 | 0.3374±0.2677 |
| CO vs AD | QDA | 0.8612±0.0055 | 0.0000±0.0000 | 0.9990±0.0063 | 0.6045±0.1742 | 0.0000±0.0000 |
| CO vs AD | GNB | 0.8586±0.0702 | 0.5625±0.2640 | 0.9060±0.0699 | 0.7925±0.1447 | 0.5189±0.2214 |

| Problem | Model | Acc | Sen | Spec | AUC | F1 |
| CO/MCI/AD | Logistic Regression | 0.6120±0.0772 | 0.4502±0.0941 | 0.7394±0.0523 | 0.6976±0.0688 | 0.4495±0.1027 |
| CO/MCI/AD | SVM RBF | 0.6415±0.0188 | 0.3429±0.0190 | 0.6746±0.0147 | 0.7560±0.0729 | 0.2810±0.0346 |
| CO/MCI/AD | SVM Linear | 0.5840±0.0682 | 0.4245±0.0899 | 0.7261±0.0477 | 0.6982±0.0807 | 0.4216±0.0952 |
| CO/MCI/AD | FNN (MLP 64-32) | 0.6561±0.0631 | 0.4558±0.0686 | 0.7568±0.0415 | 0.6807±0.0785 | 0.4525±0.0811 |
| CO/MCI/AD | QDA | 0.6300±0.0330 | 0.3448±0.0270 | 0.6754±0.0212 | 0.5677±0.0907 | 0.2971±0.0410 |
| CO/MCI/AD | GNB | 0.5593±0.0781 | 0.4485±0.1009 | 0.7450±0.0477 | 0.6560±0.0865 | 0.4397±0.1009 |

