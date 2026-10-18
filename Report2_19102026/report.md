# BEGAD - Week 2 Report: ROI-level classification on T5 and T1235

## 1. Dataset overview

This week I moves from the subject-level last week to an **ROI-level design**: features are computed **per region of interest (ROI)** of each task, and each ROI row contributes **one sample**. The label of every sample is the diagnostic group of its participant (CO / MCI / AD). This is **same with Ali setup**.

Both datasets contain **64 participants** (CO 32 / MCI 17 / AD 15).

### T5 - reading only

T5 is the reading task. The text stimulus is divided into **158 ROIs**, one ROI per letter.

- `T5.xlsx`: 6,691 rows; rows per participant: min 5, median 115, max 146.

### T1235 - King–Devick and reading

T1235 combines the three King–Devick slides (T1, T2, T3) with the reading task (T5):

| Task | Stimulus | ROIs | ROI unit |
|---|---|---|---|
| T1 | KD slide 2 | 39 | one ROI per number |
| T2 | KD slide 3 | 39 | one ROI per number |
| T3 | KD slide 4 | 39 | one ROI per number |
| T5 | Reading | 158 | one ROI per letter |
| **Total** | | **275** | |

- `T1235.xlsx`: 13,139 rows; rows per participant: min 39, median 220, max 268.

### Features per ROI row

One sample is one (participant, task, ROI) row with **19 features** (T1235), grouped by modality:

- **Visual (6):** `nFixations`, `nTobiiSaccades`, `regSaccades`, `longSaccades`, `tinySaccades`, `saccadeTotLength`.
- **Speech (4):** `totalFixTime`, `totalSpokenTime`, `speechDelay`, `endSpeechDelay`.
- **Pupil (9):** `startPupLB`, `startPupRB`, `endPupLB`, `endPupRB`, `diffPupL`, `diffPupR`, `maxPupLB`, `minPupLB`, `diffXpupL`.


**Note - ROI level vs subject level.** In subject level design, one sample represented one participant, so samples were independent and ordinary cross-validation was valid. In the ROI-level design one participant produces up to 158 samples in T5 and up to 275 samples in T1235 that share the same label, the same eye physiology, and the same recording session. These samples are **not independent**. Splitting, folding, or hypothesis testing at the ROI level without accounting for participants is pseudoreplication and can inflate significance and apparent performance.

## 2. Experiment design (shared by T1235 and T5)

The protocol below is identical for T1235 and T5. The only differences are the data file and the pupil branch columns of the Ali's Multimodal model.

**Problems.** co_vs_mci (positive class MCI), co_vs_ad (positive class AD), co_mci_ad (3-class, metrics are macro averages).

**Models.**

| Model | Specification |
|---|---|
| Logistic Regression | `max_iter=2000` |
| SVM RBF | `C=1, gamma='scale', probability=True` |
| Random Forest | `n_estimators=100` |
| LightGBM | `n_estimators=100` |
| FNN (MLP 64-32) | relu, adam, alpha 1e-4, batch 32, lr 1e-3, max_iter 300 |
| Torch Multimodal | three branches with `ROI` appended last: visual 7d (6 + ROI), speech 5d (4 + ROI), pupil 10d (9 + ROI; T5: 7d). L2 1e-3, dropout 0.2, Adam lr 1e-3, batch 32, max 300 epochs, early stopping (patience 15) on a validation split (20%) carved from the training folds. |


**Evaluation.** 10 folds, split seed 42, model seeds [42, 2026, 1, 0, 7]. 

**Two split protocols**:

1. **Group split (participant-aware).** Participant-level stratified: all rows of one participant stay in a single fold, and stratification follows the diagnostic group. This split estimates generalization to **unseen participants**.
2. **Random split.** Row-level `StratifiedKFold` stratified by y, ignoring participant identity. Rows of the same participant can land in both the training and the test fold. This follows the split used by Ali.

**Note - group split vs random split.** The two splits answer different questions. The group split asks whether a model can classify a **new participant** from ROI-level features; it is the leakage-free estimate of generalization. The random split allows training and test samples to come from the same participant, so a model can learn individual eye-movement signatures (an identity shortcut) instead of disease-related patterns. Scores under the random split are therefore expected to be **optimistic** and must not be read as generalization to unseen participants.

**Metrics.** Acc, Sen, Spec, AUC.

## 3. Results with T1235


### Group split (participant-level)

| Model | co_vs_mci | | | | │ | co_vs_ad | | | | │ | co_mci_ad | | | |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Logistic Regression | 0.489±0.167 | 0.449±0.258 | 0.491±0.236 | 0.505±0.244 | │ | 0.537±0.135 | 0.464±0.215 | 0.361±0.264 | 0.628±0.116 | │ | 0.349±0.052 | 0.483±0.078 | 0.353±0.080 | 0.674±0.027 |
| SVM RBF | 0.497±0.163 | 0.451±0.239 | 0.374±0.175 | 0.572±0.208 | │ | 0.526±0.113 | 0.474±0.155 | 0.325±0.169 | 0.620±0.103 | │ | 0.365±0.096 | 0.482±0.092 | 0.336±0.074 | 0.674±0.041 |
| Random Forest | 0.504±0.145 | 0.442±0.232 | 0.239±0.143 | 0.659±0.172 | │ | 0.575±0.112 | 0.511±0.157 | 0.280±0.181 | 0.711±0.095 | │ | 0.373±0.074 | 0.485±0.078 | 0.318±0.053 | 0.664±0.033 |
| LightGBM | 0.508±0.160 | 0.451±0.232 | 0.338±0.178 | 0.614±0.184 | │ | 0.558±0.107 | 0.486±0.149 | 0.326±0.195 | 0.661±0.097 | │ | 0.368±0.066 | 0.475±0.069 | 0.325±0.051 | 0.666±0.032 |
| FNN (MLP 64-32) | 0.530±0.120 | 0.474±0.190 | 0.336±0.149 | 0.637±0.126 | │ | 0.550±0.084 | 0.482±0.101 | 0.341±0.139 | 0.645±0.084 | │ | 0.368±0.057 | 0.504±0.054 | 0.337±0.043 | 0.674±0.021 |
| Torch Multimodal | 0.550±0.118 | 0.431±0.250 | 0.080±0.085 | 0.806±0.161 | │ | 0.596±0.108 | 0.430±0.161 | 0.083±0.133 | 0.850±0.134 | │ | 0.429±0.096 | 0.483±0.100 | 0.314±0.063 | 0.659±0.041 |

### Random split (row-level)

| Model | co_vs_mci | | | | │ | co_vs_ad | | | | │ | co_mci_ad | | | |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Logistic Regression | 0.571±0.015 | 0.593±0.015 | 0.599±0.029 | 0.555±0.018 | │ | 0.628±0.017 | 0.609±0.026 | 0.506±0.036 | 0.688±0.021 | │ | 0.428±0.012 | 0.598±0.010 | 0.437±0.014 | 0.718±0.005 |
| SVM RBF | 0.668±0.013 | 0.741±0.017 | 0.703±0.028 | 0.650±0.014 | │ | 0.684±0.013 | 0.728±0.016 | 0.599±0.018 | 0.725±0.021 | │ | 0.528±0.015 | 0.709±0.010 | 0.522±0.014 | 0.762±0.008 |
| Random Forest | 0.780±0.014 | 0.853±0.009 | 0.679±0.021 | 0.835±0.015 | │ | 0.779±0.012 | 0.840±0.009 | 0.639±0.024 | 0.848±0.018 | │ | 0.652±0.010 | 0.822±0.006 | 0.631±0.010 | 0.816±0.005 |
| LightGBM | 0.767±0.015 | 0.848±0.010 | 0.740±0.020 | 0.782±0.020 | │ | 0.767±0.014 | 0.834±0.013 | 0.702±0.009 | 0.799±0.022 | │ | 0.639±0.013 | 0.813±0.007 | 0.629±0.011 | 0.816±0.006 |
| FNN (MLP 64-32) | 0.717±0.015 | 0.779±0.017 | 0.655±0.046 | 0.750±0.034 | │ | 0.704±0.020 | 0.740±0.017 | 0.621±0.042 | 0.745±0.042 | │ | 0.574±0.015 | 0.751±0.011 | 0.569±0.014 | 0.784±0.007 |
| Torch Multimodal | 0.758±0.014 | 0.831±0.011 | 0.544±0.055 | 0.876±0.024 | │ | 0.767±0.009 | 0.812±0.014 | 0.448±0.040 | 0.923±0.020 | │ | 0.644±0.012 | 0.810±0.010 | 0.590±0.017 | 0.798±0.008 |

## 4. Results with T5


### Group split (participant-level)

| Model | co_vs_mci | | | | │ | co_vs_ad | | | | │ | co_mci_ad | | | |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Logistic Regression | 0.448±0.088 | 0.395±0.169 | 0.375±0.183 | 0.490±0.140 | │ | 0.525±0.170 | 0.521±0.268 | 0.444±0.320 | 0.574±0.191 | │ | 0.319±0.120 | 0.470±0.141 | 0.319±0.132 | 0.655±0.059 |
| SVM RBF | 0.576±0.127 | 0.462±0.162 | 0.318±0.209 | 0.693±0.158 | │ | 0.561±0.154 | 0.537±0.239 | 0.417±0.265 | 0.635±0.145 | │ | 0.416±0.065 | 0.519±0.075 | 0.379±0.070 | 0.685±0.031 |
| Random Forest | 0.562±0.114 | 0.436±0.161 | 0.299±0.260 | 0.702±0.138 | │ | 0.605±0.161 | 0.520±0.232 | 0.323±0.269 | 0.746±0.125 | │ | 0.395±0.088 | 0.495±0.088 | 0.363±0.114 | 0.664±0.044 |
| LightGBM | 0.534±0.115 | 0.409±0.161 | 0.314±0.246 | 0.652±0.137 | │ | 0.579±0.156 | 0.498±0.216 | 0.352±0.262 | 0.695±0.134 | │ | 0.375±0.082 | 0.472±0.082 | 0.350±0.107 | 0.658±0.040 |
| FNN (MLP 64-32) | 0.500±0.088 | 0.410±0.141 | 0.322±0.212 | 0.595±0.113 | │ | 0.547±0.122 | 0.474±0.171 | 0.340±0.205 | 0.652±0.114 | │ | 0.332±0.068 | 0.471±0.084 | 0.325±0.083 | 0.646±0.036 |
| Torch Multimodal | 0.561±0.133 | 0.392±0.193 | 0.105±0.174 | 0.796±0.207 | │ | 0.644±0.146 | 0.486±0.277 | 0.164±0.264 | 0.867±0.144 | │ | 0.458±0.120 | 0.463±0.118 | 0.341±0.102 | 0.660±0.042 |

### Random split (row-level)

| Model | co_vs_mci | | | | │ | co_vs_ad | | | | │ | co_mci_ad | | | |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec | │ | Acc | AUC | Sen | Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Logistic Regression | 0.542±0.018 | 0.569±0.030 | 0.549±0.047 | 0.538±0.026 | │ | 0.617±0.015 | 0.654±0.027 | 0.573±0.044 | 0.636±0.017 | │ | 0.406±0.016 | 0.597±0.018 | 0.412±0.020 | 0.701±0.012 |
| SVM RBF | 0.700±0.023 | 0.723±0.025 | 0.565±0.050 | 0.770±0.025 | │ | 0.717±0.020 | 0.766±0.028 | 0.677±0.045 | 0.734±0.019 | │ | 0.590±0.025 | 0.733±0.028 | 0.572±0.029 | 0.780±0.015 |
| Random Forest | 0.801±0.019 | 0.860±0.021 | 0.658±0.044 | 0.876±0.021 | │ | 0.834±0.017 | 0.882±0.014 | 0.692±0.043 | 0.894±0.012 | │ | 0.717±0.023 | 0.865±0.013 | 0.696±0.025 | 0.843±0.012 |
| LightGBM | 0.788±0.024 | 0.844±0.026 | 0.687±0.051 | 0.840±0.025 | │ | 0.834±0.013 | 0.882±0.018 | 0.734±0.035 | 0.875±0.012 | │ | 0.704±0.029 | 0.857±0.016 | 0.693±0.029 | 0.840±0.015 |
| FNN (MLP 64-32) | 0.728±0.019 | 0.772±0.019 | 0.640±0.053 | 0.773±0.039 | │ | 0.769±0.023 | 0.794±0.025 | 0.643±0.053 | 0.821±0.032 | │ | 0.623±0.021 | 0.787±0.016 | 0.623±0.020 | 0.803±0.011 |
| Torch Multimodal | 0.796±0.021 | 0.862±0.021 | 0.629±0.070 | 0.882±0.030 | │ | 0.853±0.014 | 0.893±0.020 | 0.676±0.051 | 0.927±0.015 | │ | 0.748±0.020 | 0.886±0.014 | 0.716±0.022 | 0.852±0.011 |