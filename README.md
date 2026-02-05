# 🐱🐶 Cat/Dog MLOps Demo

A hands-on demonstration of **MLOps** using DVC (data versioning) and MLflow (experiment tracking).

## 📋 Demo Overview

| Phase | Data | Training | What You'll Learn |
|-------|------|----------|-------------------|
| 1 | 100 images | Frozen backbone | Baseline + DVC tracking |
| 2 | Same 100 | Better hyperparams | MLflow experiment comparison |
| 3 | 200 images | Unfreeze layer4 | Data versioning + fine-tuning |

---

## 🚀 Setup

```bash
# 1. Clone this repo
git clone <your-repo-url>
cd cat_dog_mlops

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows

# 3. Install dependencies
pip install -r requirements.txt
```

---

## 📊 Phase 1: Baseline (100 images)

### Step 1.1: Pull data with DVC
```bash
dvc pull
```

### Step 1.2: Run training
```bash
jupyter notebook train.ipynb
# Run all cells
```

### Step 1.3: View experiment
```bash
mlflow ui
# Open http://localhost:5000
```

**What to observe in MLflow:**
- Parameters: lr=0.001, epochs=5
- Metrics: accuracy, precision, recall, F1, AUC
- Artifacts: confusion_matrix.png, roc_curve.png

---

## 🔧 Phase 2: Hyperparameter Tuning

### Step 2.1: Modify notebook config
```python
LEARNING_RATE = 0.0001  # was 0.001
EPOCHS = 10             # was 5
```

### Step 2.2: Run training again
Run all cells in notebook

### Step 2.3: Compare experiments
In MLflow UI → Select both runs → Click "Compare"

**What to observe:**
- Same data, different hyperparameters
- Compare accuracy improvement
- Compare confusion matrices side-by-side

---

## 📈 Phase 3: More Data + Fine-tuning

### Step 3.1: Add more images
Copy 50 more Cat and 50 more Dog images to `data/`

### Step 3.2: Track new data version
```bash
dvc add data/
git add data.dvc
git commit -m "Data v2: 200 images"
git tag data-v2
dvc push
```

### Step 3.3: Enable fine-tuning in notebook
```python
UNFREEZE_LAYER4 = True  # was False
DATA_VERSION = "v2"     # was "v1"
```

### Step 3.4: Run training
Run all cells in notebook

### Step 3.5: Compare all experiments
In MLflow UI → Select all 3 runs → Compare

**What to observe:**
- More data + unfreeze layer4 = best accuracy
- Progressive improvement across phases

---

## 🔄 Key Commands Reference

| Task | Command |
|------|---------|
| Track data changes | `dvc add data/` |
| Push data to remote | `dvc push` |
| Pull data from remote | `dvc pull` |
| View data diff | `dvc diff` |
| View experiments | `mlflow ui` |
| Check data status | `dvc status` |

---

## 📁 Project Structure

```
cat_dog_mlops/
├── data/
│   ├── Cat/           # Cat images (DVC tracked)
│   └── Dog/           # Dog images (DVC tracked)
├── data.dvc           # DVC pointer file
├── dvc_storage/       # Local DVC remote
├── mlruns/            # MLflow data (auto-created)
├── models/            # Saved models
├── train.ipynb        # Training notebook
├── requirements.txt   # Dependencies
└── README.md          # This file
```

---

## 📊 Metrics Logged to MLflow

| Category | Metrics |
|----------|---------|
| Training | train_loss, train_acc, val_loss, val_acc |
| Final | accuracy, precision, recall, f1_score, auc_roc |
| Artifacts | confusion_matrix.png, roc_curve.png, training_curves.png, model |
