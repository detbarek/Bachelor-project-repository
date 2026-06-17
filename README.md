# Characterisation of Alpha Activity in EEG

Notebook pipeline for **eyes-open (EO) / eyes-closed (EC)** classification of resting EEG and the downstream analysis of alpha activity across age: **alpha-band connectivity** (ordinary and imaginary coherence) and the **spectral decomposition** of the alpha rhythm (FOOOF aperiodic background and aperiodic-corrected alpha power).

A logistic classifier is trained on FOOOF spectral features from a small clinically labelled dataset, deployed on a large unlabelled clinical dataset to recover the eye state, and the most-confident epochs are used to compute connectivity against age. A healthy adult cohort (the Dortmund Vital Study) is the reference for normal ageing.

## Notebooks

- **`CHAMPS_LAST.ipynb`** — the full pipeline, run top to bottom: preprocessing, training, evaluation, deployment, connectivity, the Dortmund baseline and the cross-dataset comparison, diagnostics, and the consolidated statistics. Stage headings mark the sections.
- **`Aperiodic_nb.ipynb`** — the spectral-decomposition analysis. Fits the FOOOF aperiodic exponent and offset and the aperiodic-corrected alpha power per subject for the large clinical and Dortmund datasets, plots all three against age (clinical full range, clinical 20–70, and Dortmund), and runs the EC-versus-EO aperiodic contrast and the mixed models on the cached CSVs. It reuses the paths, the trained model and the analysis cohort from the main pipeline, so run `CHAMPS_LAST.ipynb` first.

## Datasets

Four datasets, each with a fixed role. The clinical datasets are not redistributed here. The Dortmund data is openly available from OpenNeuro (`ds005385`).

- **30-subject clinical (training)** — EEGLAB `.set` epochs split into EO and EC files, with a clinician's manual labels and rejection vector. Trains the classifier.
- **100-subject clinical (evaluation)** — raw EDF, two raters per recording. The labelled hold-out test set.
- **Large clinical (deployment, ~7020 subjects)** — continuous MATLAB `.mat` recordings, unlabelled. The classifier labels these and connectivity is computed on them.
- **Dortmund Vital Study (healthy reference, ~608 adults)** — from OpenNeuro, eye state known from the filenames. The healthy-ageing reference for the cross-dataset comparison.

## Preprocessing and channels

All recordings are reduced to a common **19-channel 10–20 montage** (Fp1, Fp2, F3, F4, C3, C4, P3, P4, O1, O2, F7, F8, T7, T8, P7, P8, Fz, Cz, Pz), with channel names normalised across providers (for example stripping `EEG ` and `-REF`). The target sampling rate throughout is **200 Hz**, the band-pass is **1–70 Hz** and a **50 Hz notch** suppresses mains interference.

The three clinical datasets enter the pipeline at different stages of preprocessing, handled inside the notebook. The 30-subject set is already epoched at the source, the 100-subject EDF is converted in Stage 1, and the large `.mat` set is filtered and epoched during deployment in Stage 4.

**The Dortmund recordings are preprocessed separately, before they enter this pipeline.** The raw release is 64-channel data sampled at 1000 Hz and referenced to FCz. Before the pipeline reads them they are **resampled to 200 Hz**, **band-pass filtered 1–70 Hz**, **reduced to the 19-channel 10–20 montage** and **segmented into 10-second epochs**, with the original FCz reference retained. The notebook then reads the already-preprocessed Dortmund `.fif` files directly. The common-average reference used for the cross-dataset comparison is applied inside the notebook (the "Dortmund under the common-average reference" cell), not in the external preprocessing.

## Pipeline stages (run order)

`CHAMPS_LAST.ipynb` runs in order:

1. **Configuration and helpers** — paths and all shared functions. Edit the paths first.
2. **Stage 1** — convert the 100-subject EDF recordings to epoched `.fif`.
3. **Stage 2** — train the logistic classifier on the 30-subject set under subject-grouped, age-stratified cross-validation.
4. **Stage 3** — evaluate on the 100-subject set.
5. **Stage 4** — deploy on the large clinical set, select the five most-confident EC and EO epochs per subject, and compute ordinary and imaginary coherence at the posterior ROI and 19-channel scopes.
6. **Stages 5–8** — cohort filter, clinical connectivity plots, the Dortmund baseline (simple relative-alpha model) and the classifier deployed on Dortmund for the cross-dataset comparison.
7. **Stage 9** — diagnostics: ROC, classifier confidence, alpha centre frequency, lowest-performing subjects, and the epoch-length comparison.
8. **Stage 10** — consolidated statistics: paired Wilcoxon with rank-biserial effect sizes, age regressions, the eye-by-age mixed model, and the clinical-versus-Dortmund Mann-Whitney and likelihood-ratio slope tests.

Cells marked **optional** in the notebook are robustness checks, superseded earlier versions, or one-off development cells, and are not needed to reproduce the reported results.

## Configuration (paths)

All paths live in the configuration cell at the top of each notebook. Set these to your local data before running:

- `EO_DIR`, `EC_DIR` — the 30-subject EO and EC `.set` folders
- `MEDIUM_EDF_DIR` — the 100-subject EDF folder
- `LARGE_MAT_DIR` — the large clinical `.mat` folder
- `DORTMUND_ROOT` — the preprocessed Dortmund root
- `OUT_ROOT` — where models, connectivity CSVs and figures are written

Set `MAX_SUBJECTS_LARGE` or `MAX_SUBJECTS_DORTMUND` to an integer for a quick subset run, or `None` for the full datasets.

## Requirements

Python 3 with MNE-Python, FOOOF (specparam), scikit-learn, statsmodels, scipy, numpy, pandas, matplotlib, seaborn and joblib.
