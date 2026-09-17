Mastering Statistical Analysis
===============================

Laboration 2: Modelling Wine Quality — 4-Hour Session
-------------------------------------------------------

## Learning goals

By the end of this session you will be able to:

- Curate a real, imperfect dataset before analysing it (duplicates, scaling, class balance)
- Use PCA to get an honest first look at a multivariate dataset, and read a scree plot, score plot, and loading plot correctly
- Build and validate **both** a PLS (continuous response) and an LDA (categorical response) model of the same target, and compare what each one tells you
- Judge whether a finding **transfers** to a related but different dataset, or is specific to the one you built it on
- Report your understanding of the data and the models concisely, in direct answers, rather than in a long-form report

## Background

Once viewed as a luxury good, wine is nowadays enjoyed by a much wider range of consumers. Portugal is a top-ten wine-exporting country, and exports of its *vinho verde* (northwest region) have grown substantially over the last two decades. Wine certification and quality assessment are central to this industry: certification prevents illegal adulteration (protecting public health) and assures the market of quality; quality evaluation also feeds back into the winemaking process itself, by identifying which production factors actually matter, and into pricing, by stratifying premium brands.

Wine certification is generally assessed by **physicochemical** and **sensory** tests. Physicochemical laboratory tests — density, alcohol content, pH, and so on — are objective and routine. Sensory tests rely on human tasting panels, and taste is the least understood of the human senses: the relationship between what the lab measures and what a taster scores is complex and only partly understood. That gap is exactly what this laboration explores: **can the physicochemical profile of a wine predict how it will be scored?**

The data come from the *vinho verde* certification entity (CVRVV), collected between May 2004 and February 2007. Each row is one wine sample, described by 11 physicochemical measurements. Each sample's `quality` score is the **median of at least three independent blind sensory assessments**, on a scale from 0 (very bad) to 10 (excellent) — in practice, scores cluster tightly around 5–7, as you will see for yourself in Step 1. Because red and white *vinho verde* have quite different taste profiles, they were kept as two separate datasets (1599 red, 4898 white samples) — and used as two separate parts of today's session for exactly that reason.

## The 4-hour workflow

This laboration follows one fixed workflow, in four timed steps. Stick to the timings loosely — the goal is a complete, honest analysis, not a perfect one. If you're still polishing a step when its window ends, move on anyway; you can always note in your answers what you would have checked with more time.

| Time | Step | What you produce |
|---|---|---|
| 0:00–0:30 | **1. Data curation** | A cleaned, documented dataset ready for analysis |
| 0:30–1:30 | **2. PCA** | Scree plot, score plot, loading plot, and a read of what they show |
| 1:30–3:00 | **3. Model quality: PLS *and* LDA** | Both models, validated, compared |
| 3:00–3:45 | **4. Does it transfer? White wine** | The same analysis repeated on the second dataset, compared to red |
| 3:45–4:00 | **Wrap-up** | Your answer sheet, finalised (see "What to hand in") |

## Dataset

Two files are provided in this folder, and — unlike a 2-hour version of this lab — **both are required today**, not just the first:

- `winequality-red.csv` — 1599 samples — your **primary** dataset for Steps 1–3
- `winequality-white.csv` — 4898 samples — your **transfer-check** dataset for Step 4

Both are semicolon-separated (`sep=';'` in `pd.read_csv`), with 11 physicochemical predictors and one response column, `quality`:

```
1  fixed acidity        5  chlorides              9  pH
2  volatile acidity      6  free sulfur dioxide   10  sulphates
3  citric acid           7  total sulfur dioxide  11  alcohol
4  residual sugar        8  density               12  quality (response, 0-10)
```

Do not combine the two files into one analysis. Red and white wine were characterised separately for a reason (the background text above explains why); mixing them would let a model "cheat" by partly learning to separate wine colour instead of quality. Step 4 asks you to compare your two *sets of findings*, not merge the two datasets.

---

## Step 1 — Data Curation (0:00–0:30)

Load the dataset and get it into a state you'd trust before running any analysis on it. At minimum:

1. Load the CSV and check its shape, column names, and dtypes.
2. Check for missing values.
3. **Check for duplicate rows.** Look carefully at what you find before deciding what to do about it — this dataset has a real, non-trivial answer here, not just a "yes/no" check.
4. Look at the distribution of `quality`. A simple `value_counts()` or histogram is enough.
5. Decide on, and apply, a scaling/standardisation strategy for the 11 predictors, ready for Step 2. (Think about *why* this matters before you do it — the variables are not remotely on the same scale.)

> **Checkpoint 1a**: How many duplicate rows did you find? A duplicate here means two samples with *identical* values on all 11 physicochemical measurements *and* quality. Give one plausible reason such duplicates could genuinely occur in this dataset (not just data-entry error), and one reason they could be a real problem for Steps 2–3 if left in. State what you decided to do, and why.

> **Checkpoint 1b**: Describe the distribution of `quality` in one or two sentences (where does it concentrate, how rare are the extremes?). Keep this answer in mind — you'll need it again in Step 3.

---

## Step 2 — PCA (0:30–1:30)

Run a PCA on the (curated, standardised) 11 predictor variables — **do not include `quality` itself in the PCA**, it is the response you're trying to predict, not a predictor.

1. Fit the PCA and produce a **scree plot** (individual + cumulative explained variance).
2. Produce a **score plot** (PC1 vs PC2 at minimum). Colour the points by `quality`.
3. Produce a **loading plot** for the same two components.
4. Decide how many components you would keep, using one of the criteria from the course (cumulative variance, Kaiser, or eyeballing the scree plot's elbow), and say which one you used.

> **Checkpoint 2a**: How many components did you keep, and by which criterion? Roughly what fraction of the total variance do they capture?

> **Checkpoint 2b**: Look at the score plot coloured by quality. Is there a visible trend across PC space, or does quality look scattered across it? Be honest here — a "no visible trend" answer is a completely valid, useful finding, not a failure.

> **Checkpoint 2c**: Pick the two variables with the largest loadings (in either direction) on PC1. Using the loading plot, describe in one sentence what PC1 physically represents in terms of the original wine chemistry.

---

## Step 3 — Model Quality: PLS *and* LDA (1:30–3:00)

With four hours instead of two, build **both** models of `quality` today, not just one — the two framings answer genuinely different questions, and comparing them directly is more instructive than picking one and never seeing the other:

- Treat `quality` as a **continuous** score (0–10) → build a **PLS regression** model.
- Collapse `quality` into a small number of **ordered categories** (e.g. "low" / "medium" / "high") → build an **LDA classification** model.

**Build the PLS model:**
1. Split your data into a training and test set (or set up cross-validation).
2. Fit a PLS regression of `quality` on the 11 (standardised) predictors.
3. Choose the number of latent variables using cross-validated RMSE or Q², not just training-set fit.
4. Report test-set performance (RMSE and/or R²) and a predicted-vs-observed plot.

**Build the LDA model:**
1. Define your quality categories explicitly (state your bin edges and why you chose them — this is a real modelling decision, not a formality).
2. Check the class sizes after binning. If they are very unbalanced, say so and note what that means for how much you should trust the model's accuracy.
3. Split into training and test sets.
4. Fit LDA, and report test-set performance with a **confusion matrix**, not accuracy alone.

> **Checkpoint 3a**: Report your PLS model's validated performance (the specific RMSE and/or R² on the test set). Is this good enough to be useful to a wine producer, in your judgement? Justify briefly.

> **Checkpoint 3b**: Report your LDA model's validated performance (the confusion matrix and per-class accuracy on the test set). Is this good enough to be useful to a wine producer, in your judgement? Justify briefly — and check it against the accuracy a model would get by just always guessing the majority class.

> **Checkpoint 3c**: Compare the two models directly. Which one would you actually recommend to a wine producer, and for what decision specifically (e.g. "flag a batch for closer tasting" vs. "predict an exact score")? Neither answer is "more correct" — justify your pick.

> **Checkpoint 3d**: Which one or two original variables matter most to each model (PLS regression/VIP coefficients for PLS; LDA discriminant loadings for LDA)? Do the two models agree with each other? Does either match what you found for PC1 in Checkpoint 2c, or point somewhere different?

---

## Step 4 — Does It Transfer? White Wine (3:00–3:45)

Everything so far was built on red wine alone. Repeat the analysis on `winequality-white.csv` and find out whether your findings are a genuine property of *vinho verde* chemistry, or an artefact of the specific dataset you happened to build on — a distinction that matters every time a model trained on one dataset gets applied to another.

1. Curate the white wine dataset the same way as Step 1 (check duplicates, missing values, quality distribution — white wine has its own, different answer here).
2. Redo the PCA from Step 2 on white wine (standardise, fit, scree/score/loading plots).
3. Redo **one** of your two Step 3 models (PLS or LDA, your choice — you do not need to rebuild both again today) on white wine, using the same methodology as before.

> **Checkpoint 4a**: Does PC1 mean the same thing physically in white wine as it did in red (Checkpoint 2c)? Compare the top-loading variables directly. If it's different, say what changed and speculate briefly on why (white and red *vinho verde* are made from different grapes, with genuinely different typical chemistry — e.g. white wine's residual sugar varies far more).

> **Checkpoint 4b**: Does your chosen model's validated performance and its most important variables (Checkpoint 3a/3b/3d) transfer to white wine, or not? Report the white-wine numbers alongside the red-wine ones for direct comparison. A genuine difference is just as interesting and valid an answer as a match — the point is to actually check, not to assume transfer either way.

---

## What to hand in

**No lab report.** Hand in two things:

1. **Your working notebook** (`.ipynb`) — the code that produces everything above, in order. Light comments are enough to show what each cell does; you do not need prose explanations here, since that's what the answer sheet is for.
2. **Your answer sheet** — direct, concise answers to Checkpoints 1a through 4b (11 questions). A few sentences per answer is normal; a paragraph is too long. Include the specific numbers/plots each question asks for — "it looked fine" is not an answer, "RMSE = 0.61 on the test set" is.
