# ML-NIDS: Machine Learning-Based Network Intrusion Detection System

**طراحی و پیاده‌سازی سیستم تشخیص نفوذ شبکه مبتنی بر یادگیری ماشین بر روی کلان‌داده‌های ترافیک شبکه**

| | |
|---|---|
| **Course** | Big Data Analytics (Master's), Fall 2026 / پاییز ۱۴۰۵ |
| **University** | Islamic Azad University, Central Tehran Branch |
| **Instructor** | Dr. Reza Ravanmehr |
| **Teaching Assistant** | Ms. Mina Mashayekhi |
| **Team** | Iman Nazer Masouleh, Taha Afshari |
| **Final deadline** | Tuesday, December 29, 2026 |

## 1. Abstract

This project builds a scalable PySpark pipeline that analyzes the BCCC-CSE-CIC-IDS2018 network-flow dataset (48,182,782 flows, 323 columns, 14 labels) and trains tree-based machine-learning models to detect malicious traffic. We solve two tasks: binary detection (Benign vs. Attack) and multi-class attack classification. The pipeline runs on Google Colab with PySpark MLlib. It covers leakage-free preprocessing, feature selection that ranks and marks the most useful flow features, and imbalance-aware training of Decision Tree, Random Forest and XGBoost models. Models are compared on per-class Recall, macro-F1 and False Positive Rate rather than accuracy.

---

## 2. Objectives and Success Metrics

### 2.1 Objectives

| ID | Objective |
|---|---|
| O1 | **Binary detection:** classify each flow as `Benign` (0) or `Attack` (1). |
| O2 | **Multi-class classification:** identify the attack family of each flow. |
| O3 | **Feature selection:** produce a marked list of the features most useful for detecting attacks, overall and per attack. |
| O4 | **Scalability:** process the full 48M-row dataset with Spark and record runtimes for each pipeline stage. |
| O5 | **Interpretability:** extract If-Then rules (Decision Tree) and feature importances (RF, XGBoost). |

### 2.2 Label Distribution (measured)

| Label | Flows | Share |
|---|---:|---:|
| Benign | 43,092,133 | 89.43% |
| DDoS_HOIC | 2,164,586 | 4.49% |
| DoS_HULK | 1,824,450 | 3.79% |
| DDoS_LOIC_HTTP | 289,527 | 0.60% |
| Bot | 286,099 | 0.59% |
| Brute_Force_FTP | 193,360 | 0.40% |
| Brute_Force_SSH | 188,444 | 0.39% |
| DoS_SlowHTTP | 105,550 | 0.22% |
| DoS_Golden_Eye | 29,485 | 0.06% |
| DoS_Slowloris | 8,269 | 0.02% |
| Brute_Force_Web | 536 | <0.01% |
| Brute_Force_XSS | 232 | <0.01% |
| SQL_Injection | 106 | <0.01% |
| Infiltration | 5 | <0.01% |

**Class decisions for multi-class:**
- Merge `Brute_Force_Web`, `Brute_Force_XSS` and `SQL_Injection` into `Web_Attack` (874 flows).
- Drop `Infiltration`: 5 samples is too few to learn from or to evaluate.
- Result: **11 classes**.

### 2.3 Metrics

| Metric | Task | Why we use it |
|---|---|---|
| Precision (per class) | Both | Shows how many of the alerts raised for a class are correct. |
| Recall (per class) | Both | Shows how many of the real attacks were caught. This is the main security metric. |
| F1 (per class) | Both | Balances Precision and Recall for each class. |
| **Macro-F1** | Both | Weights every class equally, so rare attacks count as much as Benign. **Primary model-selection metric.** |
| False Positive Rate (Benign) | Both | Benign flows flagged as attacks, i.e. false alarms. Must be measured on the natural-distribution test set. |
| ROC-AUC | Binary | Ranking quality across all thresholds. |
| PR-AUC | Binary | More informative than ROC-AUC when positives are rare. |

### 2.4 Why Accuracy Misleads

A model that predicts `Benign` for every flow scores **89.4% accuracy** and detects zero attacks. Accuracy is dominated by the majority class, so it hides failure on exactly the classes that matter. We report accuracy only for completeness. Model selection uses macro-F1, and we also report per-class Recall and FPR.

### 2.5 Success Targets

| Target | Binary | Multi-class |
|---|---|---|
| Macro-F1 on balanced test set | ≥ 0.95 | ≥ 0.85 |
| Recall per class on balanced test set | ≥ 0.95 | ≥ 0.80 for classes with ≥ 1,000 test flows |
| FPR on natural-distribution test set | ≤ 1% | ≤ 1% |

---

## 3. Tech Stack

| Tool | Purpose | Version note |
|---|---|---|
| Google Colab (CPU runtime) | Compute: 2 vCPU, ~12 GB RAM, ~108 GB disk | Free tier. Sessions are temporary. |
| Java (OpenJDK) | Runtime for Spark | Colab ships OpenJDK 21. |
| PySpark (MLlib, Spark SQL) | Distributed processing and ML | `pyspark==4.0.*`. Spark 4 officially supports Java 21. Do **not** use 3.5 on Java 21. |
| xgboost (`xgboost.spark`) | XGBoost estimator inside Spark | Latest; `SparkXGBClassifier`. |
| pandas, matplotlib, seaborn | Small result tables and charts | Use only on aggregated or sampled data. |
| Google Drive | Persistent storage: raw chunks, processed data, models, reports | Path: `MyDrive/Big Data/Project/ML-NIDS`. |
| GitHub (private repo `ml-nids`) | Version control for notebooks, code and reports | Notebooks committed with outputs. |
| Hugging Face Hub (`huggingface_hub`) | Backup source of the parquet chunks | `hf download` CLI. |

---

## 4. Repository Structure

```text
ml-nids/
├── README.md                  # project summary + how to run
├── PROJECT_PLAN.md            # this file
├── .gitignore                 # ignores data/, models/*, *.parquet
├── notebooks/
│   ├── 00_setup.ipynb
│   ├── 01_load.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_clean.ipynb
│   ├── 04_features.ipynb
│   ├── 05_sampling.ipynb
│   ├── 06_models.ipynb
│   ├── 07_evaluation.ipynb
│   └── 08_interpretation.ipynb
├── src/
│   ├── config.py              # paths, seed, label maps, column lists
│   ├── spark_session.py       # get_spark()
│   ├── data_io.py             # verified copy from Drive, read/write helpers
│   ├── cleaning.py
│   ├── features.py
│   └── evaluate.py
├── data/                      # NOT committed (raw + processed live on Drive)
│   └── README.md              # where the data comes from and how to fetch it
├── reports/                   # CSV summaries: quality, feature report, metrics
├── figures/                   # PNG charts for report and slides
└── models/                    # NOT committed (saved to Drive); README only
```

`.gitignore` additions:

```text
data/
*.parquet
models/*
!models/README.md
```

**Branching:**
- `main` must always be runnable.
- Do each phase on a branch named `phase-N-<name>` (e.g. `phase-3-clean`).
- Merge to `main` through a Pull Request reviewed by the other teammate.
- In Colab, use *File → Save a copy in GitHub* and pick the branch.

---

## 5. Phases

> Week numbering: W1 = Nov 22–28, 2026. Status as of Nov 28 is marked with [x].

> **Scope for the Dec 29 deadline**
> - Tuning: `TrainValidationSplit` only, ≤ 6 parameter combinations per model. No `CrossValidator`.
> - MLlib `GBTClassifier` baseline: dropped.
> - Experiment B (all vs. selected features): keep. Experiment A (`dst_port`): only if time allows.
> - Feature selection runs on the train split (~2.5M rows), not the full data.
> - Each owner writes their report section the same week they finish the phase.
> - Code freeze Dec 24: after that, only fixes, no new experiments.

### Phase 0: Setup
**Goal:** a working, reproducible Colab + Spark environment and a shared workspace.
**Owner:** Iman | **Weeks:** W1 (done)

- [x] Create private GitHub repo `ml-nids`, add folders and `.gitignore`
- [ ] Add Taha as a collaborator and confirm his access
- [x] Create the Drive folder `ML-NIDS/` with `dataset/`, `processed/`, `models/`, `reports/`; share it with Taha as Editor
- [ ] Taha: add a shortcut to the shared folder in his own My Drive; set `BASE = "/content/drive/MyDrive/ML-NIDS"`
- [x] Verify the runtime (`nproc`, `free -h`, `df -h`, `java -version`)
- [x] Install `pyspark==4.0.*`, `xgboost`, `huggingface_hub`
- [ ] Move setup code into `src/spark_session.py` and `src/data_io.py`
- [ ] Create branch protection on `main` (PR required)

```python
from pyspark.sql import SparkSession

spark = (SparkSession.builder
         .appName("ML-NIDS")
         .master("local[*]")
         .config("spark.driver.memory", "10g")
         .config("spark.sql.shuffle.partitions", "32")
         .config("spark.driver.maxResultSize", "2g")
         .getOrCreate())
```

**Deliverables:** `00_setup.ipynb`, working repo, shared Drive folder.

### Phase 1: Data Acquisition and Loading
**Goal:** load all 12 parquet chunks into Spark reliably every session.
**Owner:** Iman | **Weeks:** W1 (done)

- [x] Upload the 12 chunks (~16.8 GB) to `ML-NIDS/dataset/` on Drive
- [x] Copy the chunks to local disk `/content/data` at session start; never read from the Drive mount directly
- [x] Verify each copy: size must match the Drive file and the file must end with the `PAR1` footer; re-copy on mismatch
- [x] Read with Spark; confirm 48,182,782 rows and 323 columns
- [x] Save `label_counts.csv` to `reports/`
- [ ] Document the Hugging Face fallback (`hf download init5iv3/BCCC-CSE-CIC-IDS2018 ...`) in `data/README.md`

```python
import os, shutil
def verified_copy(src_dir, dst_dir="/content/data"):
    os.makedirs(dst_dir, exist_ok=True)
    for f in sorted(x for x in os.listdir(src_dir) if x.endswith(".parquet")):
        s, d = f"{src_dir}/{f}", f"{dst_dir}/{f}"
        good = os.path.exists(d) and os.path.getsize(d) == os.path.getsize(s)
        if good:
            with open(d, "rb") as fh: fh.seek(-4, 2); good = fh.read() == b"PAR1"
        if not good: shutil.copy(s, d)

df = spark.read.parquet("/content/data/*.parquet")
```

**Deliverables:** `01_load.ipynb`, `reports/label_counts.csv`.

### Phase 2: Exploratory Data Analysis
**Goal:** understand data quality and class structure before transforming anything.
**Owner:** Taha (Iman ran the first pass) | **Weeks:** W1–W2, finish by **Dec 2**

- [x] Class distribution bar chart (log scale) and Benign-vs-Attack pie chart
- [x] Single-pass quality scan: non-numeric strings, NaN, ±inf and nulls per column → `quality_report.csv`
- [x] Incomplete-handshake share per label → `handshake_by_label.csv`
- [x] Medians of key features per label and box plots from a stratified sample (~2,000 per class)
- [ ] Review all charts and tables; write 5–8 findings in the notebook
- [ ] Count exact duplicate rows (excluding ID columns), overall and per label
- [ ] Correlation heatmap of numeric features on a 500k-row stratified sample
- [ ] Note which columns are constant or near-constant

**Deliverables:** `02_eda.ipynb`, `reports/*.csv`, `figures/*.png`, written findings.

### Phase 3: Cleaning and Preprocessing
**Goal:** a clean, numeric, leakage-free dataset with binary and multi-class labels.
**Owner:** Iman | **Weeks:** W2, cleaned parquet on Drive by **Dec 4**

- [ ] Drop leakage/ID columns: `flow_id`, `timestamp`, `src_ip`, `dst_ip`, `src_port`
- [ ] Decide on `dst_port`: drop it by default; run the with/without experiment in Phase 7 if time allows
- [ ] Add a flag `incomplete_handshake` = 1 where `handshake_duration == "not a complete handshake"`
- [ ] Convert the 18 string-typed numeric columns to double with `try_cast` (non-numeric text becomes null)
- [ ] Replace ±inf and NaN with null, then fill with `-1` (a sentinel that tree models handle fine)
- [ ] Encode `protocol` with `StringIndexer`
- [ ] Map labels: merge web attacks into `Web_Attack`, drop `Infiltration`
- [ ] Create `is_attack` (binary) and `label_idx` (multi-class, via `StringIndexer`)
- [ ] Remove exact duplicates on feature columns; record rows before and after
- [ ] Write the cleaned data to parquet; save a row-count summary to `reports/cleaning_summary.csv`

```python
from pyspark.sql import functions as F
IDS = ["flow_id", "timestamp", "src_ip", "dst_ip", "src_port", "dst_port"]
WEB = ["Brute_Force_Web", "Brute_Force_XSS", "SQL_Injection"]
df = (df.drop(*IDS)
        .filter(F.col("label") != "Infiltration")
        .withColumn("label", F.when(F.col("label").isin(WEB), "Web_Attack")
                              .otherwise(F.col("label")))
        .withColumn("is_attack", (F.col("label") != "Benign").cast("int"))
        .withColumn("incomplete_handshake",
                    (F.col("handshake_duration") == "not a complete handshake").cast("int")))
```

**Deliverables:** `03_clean.ipynb`, `src/cleaning.py`, `reports/cleaning_summary.csv`.

### Phase 4: Feature Selection
**Goal:** cut 323 columns down to ~30–50 informative features and mark which features detect which attacks.
**Owner:** Taha | **Weeks:** W3, feature list frozen by **Dec 11**

- [ ] Build the code on a 1% dev sample (`df.sample(0.01, seed=42)`) while the train split is being prepared; re-run on the train split on Dec 9
- [ ] Variance filter: drop columns with `stddev == 0`, and near-constant columns where >99.9% of values are identical
- [ ] Correlation filter: on a 500k stratified sample, compute `pyspark.ml.stat.Correlation`; for each pair with |r| > 0.95, drop one
- [ ] Rank the remaining features with Random Forest `featureImportances` on the balanced train split
- [ ] Cross-check the ranking with `UnivariateFeatureSelector` (continuous features vs. categorical label, ANOVA F-test)
- [ ] Per-attack relevance: train one-vs-Benign RFs and record the top 10 features for each attack class
- [ ] Final marked list → `reports/feature_report.csv` with columns `column | status | reason | importance_rank | top_attacks`
- [ ] Charts: top-20 feature importances; heatmap of features × attack classes

**Deliverables:** `04_features.ipynb`, `src/features.py`, `reports/feature_report.csv`, `figures/feature_importance.png`, `figures/feature_attack_heatmap.png`.

### Phase 5: Handling Imbalance and Splitting
**Goal:** training data that fits in Colab memory and does not let Benign dominate, plus honest test sets.
**Owner:** Iman | **Weeks:** W2–W3, splits on Drive by **Dec 9**

- [ ] Hold out a natural-distribution test set first (2–5M rows, ~89/11 mix) and never touch it before Phase 7
- [ ] Build the working set with `sampleBy("label", fractions, seed=42)`:
  - keep all rows of the small classes
  - cap `DDoS_HOIC` and `DoS_HULK` at ~300k each
  - Benign ~1–1.5M
- [ ] Split the working set 70/15/15 within each class → `train`, `val`, `test_balanced`
- [ ] Add a class-weight column (`weightCol`): `total / (k × count_c)`; also test a square-root-softened version
- [ ] Save all splits to `ML-NIDS/processed/` on Drive as parquet
- [ ] Record class counts for every split in `reports/split_summary.csv`
- [ ] SMOTE is **not used**:
  - Spark has no native implementation
  - synthetic flows in a ~40-dimensional space built from a few hundred real samples are unreliable
  - the dataset is large enough that undersampling plus class weights covers the need

```python
counts = {r["label"]: r["count"] for r in train.groupBy("label").count().collect()}
total, k = sum(counts.values()), len(counts)
w = {c: total / (k * n) for c, n in counts.items()}
mapping = F.create_map([F.lit(x) for kv in w.items() for x in kv])
train = train.withColumn("weight", mapping[F.col("label")])
```

**Deliverables:** `05_sampling.ipynb`, parquet splits on Drive, `reports/split_summary.csv`.

### Phase 6: Modeling
**Goal:** train and tune DT → RF → XGBoost for both tasks.
**Owner:** Both (Iman: binary, Taha: multi-class) | **Weeks:** W4, all models saved by **Dec 19**

- [ ] Assemble the selected features with `VectorAssembler` inside a `Pipeline`
- [ ] **Decision Tree** (`DecisionTreeClassifier`, `weightCol`): baseline; tune `maxDepth` and `minInstancesPerNode`
- [ ] **Random Forest** (`RandomForestClassifier`, `weightCol`): tune `numTrees`, `maxDepth` and `featureSubsetStrategy`
- [ ] **XGBoost** (`SparkXGBClassifier`):
  - binary: use `scale_pos_weight = n_benign / n_attack`
  - multi-class: use `weight_col="weight"` (`scale_pos_weight` applies to binary only)
- [ ] Tune with `TrainValidationSplit` on `val`, ≤ 6 combinations per model
- [ ] Start long training runs overnight; save each model to Drive in the same cell
- [ ] Log the training time of every model
- [ ] Save the fitted pipelines to `ML-NIDS/models/` on Drive

```python
from xgboost.spark import SparkXGBClassifier
xgb = SparkXGBClassifier(features_col="features", label_col="label_idx",
                         weight_col="weight", num_workers=2,
                         tree_method="hist", max_depth=8, n_estimators=300)
model = xgb.fit(train_vec)
model.write().overwrite().save(f"{BASE}/models/xgb_multiclass")
```

**Deliverables:** `06_models.ipynb`, saved models on Drive, `reports/training_times.csv`.

### Phase 7: Evaluation and Comparison
**Goal:** compare all models fairly and measure realistic false-alarm rates.
**Owner:** Iman | **Weeks:** W5, done by **Dec 24**

- [ ] Score every model on `test_balanced` and on `test_natural`
- [ ] Report per class: Precision, Recall, F1; overall: macro-F1, weighted-F1, FPR (Benign); binary only: ROC-AUC, PR-AUC
- [ ] Confusion-matrix heatmaps (normalized by row) for the best binary and the best multi-class model
- [ ] Build the comparison table: model × task × macro-F1 × min class Recall × FPR × train time
- [ ] Experiment B: all cleaned features vs. the selected feature set (metrics and runtime)
- [ ] Experiment A (if time allows): with vs. without `dst_port`
- [ ] Scalability notes: runtime of each pipeline stage; how it would change on a cluster (only `master` and resource configs change)
- [ ] Error analysis: which classes get confused, and why

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator as MCE
ev = MCE(labelCol="label_idx", predictionCol="prediction")
f1_per_class = [ev.setMetricName("fMeasureByLabel").setMetricLabel(float(i))
                  .evaluate(pred) for i in range(11)]
macro_f1 = sum(f1_per_class) / len(f1_per_class)
recall_per_class = [ev.setMetricName("recallByLabel").setMetricLabel(float(i))
                      .evaluate(pred) for i in range(11)]
benign_fpr = ev.setMetricName("falsePositiveRateByLabel") \
               .setMetricLabel(0.0).evaluate(pred)
```

> Note: MLlib's `"f1"` metric is weighted, not macro. Use the mean of `fMeasureByLabel` as above. Check which index `StringIndexer` gave to `Benign` before using `setMetricLabel(0.0)`.

**Deliverables:** `07_evaluation.ipynb`, `src/evaluate.py`, `reports/model_comparison.csv`, confusion-matrix figures.

### Phase 8: Interpretation
**Goal:** explain why the models make their decisions.
**Owner:** Taha | **Weeks:** W5 (alongside Phase 7), done by **Dec 24**

- [ ] Export the Decision Tree rules (`toDebugString`) and turn the top branches into readable If-Then rules
- [ ] Feature importance plots for RF and XGBoost; compare them with the Phase 4 ranking
- [ ] Link the important features to attack behavior (e.g. SYN/IAT features for DoS, payload features for Web_Attack)
- [ ] Summarize interpretability vs. performance trade-offs across the three models

**Deliverables:** `08_interpretation.ipynb`, `reports/dt_rules.txt`, `figures/importance_rf.png`, `figures/importance_xgb.png`.

### Phase 9: Final Report, Presentation and Repo Cleanup
**Goal:** submit a complete, reproducible project.
**Owner:** Both | **Weeks:** written during W3–W5, final work Dec 25–28

- [ ] Each owner writes their report section in the same week they finish the phase
- [ ] Final report sections:
  - introduction
  - dataset and data challenges
  - architecture diagram
  - preprocessing
  - feature selection
  - imbalance handling
  - models
  - results
  - interpretation
  - limitations
  - future work
- [ ] Slides (12–15) built from the `figures/` charts; one backup slide set with notebook screenshots
- [ ] Rehearse twice (Dec 27–28); split speaking time evenly; prepare answers to likely questions (accuracy, leakage, sampling validity, single-node Spark)
- [ ] Repo cleanup:
  - `README.md` with run order
  - notebooks committed with outputs
  - `src/` modules imported by the notebooks
  - no data or model files in Git
- [ ] Tag the final commit `v1.0`

**Deliverables:** final report (PDF), slides, tagged repo.

---

## 6. Timeline

| Week | Dates | Iman | Taha | Milestone |
|---|---|---|---|---|
| W1 | Nov 22–28 | Phases 0, 1, 2 (first pass) | Repo/Drive access | Data loaded, EDA outputs saved ✅ |
| W2 | Nov 29–Dec 5 | Phase 3 cleaning → cleaned parquet on Drive by Dec 4 | Phase 2 findings, duplicates, correlations (by Dec 2); Phase 4 code on 1% dev sample | **Dec 5: cleaned dataset** |
| W3 | Dec 6–12 | Phase 5 splits + weights (by Dec 9); write report: Data + Cleaning | Phase 4 feature report (by Dec 11) on Iman's train split | **Dec 12: splits + feature list frozen** |
| W4 | Dec 13–19 | Phase 6 binary: DT, RF, XGBoost | Phase 6 multi-class: DT, RF, XGBoost | **Dec 19: all models saved** |
| W5 | Dec 20–24 | Phase 7 evaluation, comparison table, Experiment B | Phase 8 rules + importance plots; write report: Features, Interpretation | **Dec 24: code freeze** |
| — | Dec 25–26 | Report: Results + Conclusion | Slides 1–15 from `figures/` | Full drafts |
| — | Dec 27–28 | Review Taha's sections, repo cleanup, tag `v1.0` | Review Iman's sections | Rehearse twice |
| — | **Dec 29** | — | — | **Submit + present** |

**Critical path:** cleaning (Dec 5) → splits (Dec 9) → feature list (Dec 11) → models (Dec 19) → evaluation (Dec 24). A delay in any of these moves every later date.

---

## 7. Task Split

| Area | Iman | Taha |
|---|---|---|
| Environment and data loading (Phases 0–1) | Lead | Review, reproduce on own account |
| EDA (Phase 2) | First pass | Lead: findings, duplicates, correlations |
| Cleaning (Phase 3) | Lead | Review PR |
| Feature selection (Phase 4) | Review PR | Lead |
| Sampling and splits (Phase 5) | Lead | Review PR |
| Modeling (Phase 6) | Binary task | Multi-class task |
| Evaluation (Phase 7) | Lead | Error analysis |
| Interpretation (Phase 8) | Review | Lead |
| Report (Phase 9) | Data, preprocessing, evaluation, results sections | Intro, features, interpretation sections |
| Slides (Phase 9) | Review | Build slides 1–15 |

---

## 8. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Tight schedule (31 days from Nov 29) | Unfinished phases | Parallel tracks, fixed milestones (Dec 5 / 9 / 12 / 19 / 24), scope cuts in Section 5 |
| Colab RAM (~12 GB) | Out-of-memory crashes | `driver.memory=10g`; sample the data; select columns early; never call `toPandas()` on large data |
| Colab session limits and disconnects | Lost work mid-run | Save results to Drive after each step; split work into short notebooks; keep processed splits on Drive |
| Data size (16.8 GB parquet) | Slow scans | Compute many aggregates in one pass; rely on column pruning; use a 1% dev sample while writing code |
| Corrupted copy from Drive | Read errors (e.g. truncated `chunk_05`) | Verified copy (size + `PAR1` footer check); Hugging Face fallback |
| Severe class imbalance | Model ignores rare attacks | Merge/drop micro-classes, undersample Benign, use class weights, select models by macro-F1 |
| Data leakage (IPs, ports, IDs, timestamps) | Inflated, fake results | Drop ID columns; deduplicate before splitting; `dst_port` ablation if time allows |
| Label noise in simulated data | Wrong labels limit max performance | Inspect confused classes in error analysis; report it as a limitation |
| Spark/Java version mismatch | Hard-to-debug failures | Pin `pyspark==4.0.*` on Colab's Java 21 |

---

## 9. Colab Practical Tips

- **Start every notebook the same way:** install, mount Drive, verified copy, start Spark. Keep this in `src/`.
- **Dev sample first:** build and debug code on `df.sample(0.01, seed=42)`, then run once on the full data.
- **Cache only what you reuse and what fits:** `train.cache()` after sampling is fine; caching the full 48M × 323 table is not.
- **Checkpoint to Drive:** after each expensive step, `write.mode("overwrite").parquet(f"{BASE}/processed/...")`, then read it back in the next notebook.
- **Save models:** `model.write().overwrite().save(f"{BASE}/models/<name>")`; load them with the matching `*Model.load()`.
- **Keep outputs:** turn off *Edit → Notebook settings → Omit code cell output*; commit notebooks with outputs.
- **Reproducibility:** use `seed=42` in every `sample`, `sampleBy`, `randomSplit` and every model.
- **Overnight runs:** start long training before sleeping, with saving to Drive in the same cell.
- **Free resources:** call `spark.stop()` at the end, then *Runtime → Disconnect and delete runtime*.
- **Approximate statistics:** use `percentile_approx` instead of exact medians at this scale.

---

## 10. Final Deliverables Checklist

- [ ] `PROJECT_PLAN.md` and `README.md` (run instructions, notebook order)
- [ ] Notebooks `00`–`08`, committed with outputs
- [ ] `src/` modules used by the notebooks
- [ ] `reports/`:
  - `label_counts.csv`
  - `quality_report.csv`
  - `cleaning_summary.csv`
  - `feature_report.csv`
  - `split_summary.csv`
  - `model_comparison.csv`
  - `training_times.csv`
  - `dt_rules.txt`
- [ ] `figures/`:
  - class distribution
  - feature importance
  - feature × attack heatmap
  - confusion matrices
  - comparison chart
- [ ] Saved models on Drive (DT, RF, XGBoost; binary and multi-class)
- [ ] Final report (PDF)
- [ ] Presentation slides plus a backup slide set
- [ ] Repo tagged `v1.0`

**Future work** (out of scope): run on a multi-node Spark cluster; test deep-learning models; real-time scoring on streaming flows.

---

## 11. References

1. Shafi, M., Lashkari, A. H., & Roudsari, A. H. (2025). *Toward Generating a Large Scale Intrusion Detection Dataset and Intruders Behavioral Profiling Using Network and Transportation Layers Traffic Flow Analyzer (NTLFlowLyzer).* Journal of Network and Systems Management, 33, 44.
2. BCCC-CSE-CIC-IDS2018 dataset, Behaviour-Centric Cybersecurity Center (BCCC), York University: https://www.yorku.ca/research/bccc/ucs-technical/cybersecurity-datasets-cds/large-scale-intrusion-detection-dataset-bccc-cse-cic-ids2018/
3. Hugging Face mirror (parquet chunks): https://huggingface.co/datasets/init5iv3/BCCC-CSE-CIC-IDS2018
4. Sharafaldin, I., Lashkari, A. H., & Ghorbani, A. A. (2018). *Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization.* Proc. 4th International Conference on Information Systems Security and Privacy (ICISSP), 108–116. This is the methodology paper cited for CSE-CIC-IDS2018 [verify page numbers].
5. CSE-CIC-IDS2018 original dataset page (Canadian Institute for Cybersecurity, UNB): https://www.unb.ca/cic/datasets/ids-2018.html
6. NTLFlowLyzer (feature extractor): https://github.com/ahlashkari/NTLFlowLyzer
7. Apache Spark MLlib Guide: https://spark.apache.org/docs/latest/ml-guide.html
8. XGBoost PySpark estimator documentation: https://xgboost.readthedocs.io/en/stable/tutorials/spark_estimator.html
