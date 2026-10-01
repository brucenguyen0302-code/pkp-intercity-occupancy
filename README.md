# Predicting passenger occupancy of PKP Intercity trains

Coursework for UTS 42172 Introduction to Artificial Intelligence (AT1, Example III): a multi-class classification problem solved with techniques from the subject.

## Problem

A long-distance train operator wants to know, before a train departs, how full it will be. PKP Intercity reports occupancy in three bands: low (under 50%), medium (50% to 80%) and high (over 80%). This project predicts the band of a train run from planning-time information only: timetable, calendar, route, weather and planned infrastructure restrictions. Delay values and disruptions logged during the run are deliberately excluded.

## Data

PKP Intercity delays dataset.

Kostrz, M. (2026). *PKP Intercity delays* [Data set]. Zenodo. https://doi.org/10.5281/zenodo.21700869

- Licence: CC BY 4.0
- Period: 1 March 2026 to 19 May 2026
- Used file: `processed_tabular/train_delays_tabular.parquet` (558,334 stop records, 33,378 train runs)
- The data is **not stored in this repository**. Download it from Zenodo.

The occupancy label is constant within each train run, so the stop records are aggregated into one row per run before modelling.

## Method

| Step | Choice |
|---|---|
| Unit of analysis | One row per train run (33,378 runs) |
| Split | Chronological by run order: 70% train, 15% validation, 15% test; test set used once |
| Models | Random forest, feedforward neural network (Keras), logistic regression baseline |
| Naive baselines | Majority class; per-service majority |
| Tuning | 20 random configurations each for the forest and the FNN, 8-value grid for logistic regression, selected on validation macro F1 |
| Comparison | Exact McNemar test with Holm correction, bootstrap 95% intervals, 5-seed stability, training and prediction time, model size |
| Analysis | Permutation importance, feature-group ablation, error by train category and day of week |

## Main results (test set, 5,007 runs)

| Model | Accuracy | Macro F1 |
|---|---|---|
| Random forest | 0.718 | 0.709 |
| Per-service majority baseline | 0.600 | 0.558 |
| Logistic regression | 0.555 | 0.553 |
| FNN | 0.576 | 0.539 |
| Majority class | 0.372 | 0.181 |

The random forest is the best model, the most stable across seeds and the cheapest to train. Calendar and timetable features carry most of the signal; weather adds nothing in this window. The high-occupancy band is the weakest, partly because its share rises from 19.6% in training to 26.2% in the test period.

## Limitations

Single operator, one spring season of about 11 weeks, occupancy bands estimated by the operator, and weather treated as if it were a forecast. Results should not be extended to winter, holiday peaks or other operators without new data.

## How to run

1. Download `train_delays_tabular.parquet` from the Zenodo record above.
2. Put it anywhere under `MyDrive/IntroToAI` in Google Drive.
3. Open `notebooks/AT1_III_pkp_occupancy.ipynb` in Google Colab and choose Runtime, Restart and run all. A GPU is not needed.
4. Figures and result tables are written to `MyDrive/IntroToAI/AT1_III/figures` and `.../results`.

Tuning and stability tables are cached in the `results` folder under names that include the budget and seed. Delete them to force a full re-run.

## Repository layout

```
notebooks/    the Colab notebook with all outputs
figures/      figures produced by the notebook (200 dpi PNG)
results/      CSV tables behind the report (tuning, scores, stability, ablation)
```

The trained random forest file (about 29 MB) is not included.
