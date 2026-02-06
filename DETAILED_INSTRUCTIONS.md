# 🐱🐶 Cat/Dog MLOps Demo - DETAILED Setup & Demo Guide

This is a **step-by-step** guide for the instructor to set up the demo repository, and for students to reproduce it.

---

# PART A: INSTRUCTOR SETUP (You Do This First)

This section creates the repository that students will clone.

---

## A1. Create the Project Folder

```bash
# Navigate to your slides folder
cd /home/gvsr/Downloads/course/teaching/weeks/slides

# Create the project folder
mkdir cat_dog_mlops
cd cat_dog_mlops
```

---

## A2. Initialize Git Repository

```bash
# Initialize Git
git init

# Verify
ls -la
# You should see: .git/
```

---

## A3. Initialize DVC

```bash
# Initialize DVC (creates .dvc/ folder)
dvc init

# Verify
ls -la
# You should see: .git/ .dvc/ .dvcignore

# First commit
git add .dvc .dvcignore
git commit -m "Initialize DVC"
```

---

## A4. Configure Local DVC Remote

```bash
# Create local storage folder (simulates S3/GCS)
mkdir dvc_storage

# Tell DVC to use this as the remote
dvc remote add -d local_remote ./dvc_storage

# The -d flag makes this the default remote
# Verify configuration
cat .dvc/config
# Should show: [core] remote = local_remote

# Commit DVC config
git add .dvc/config
git commit -m "Configure local DVC remote"
```

---

## A5. Create Project Files

Create these files (already created for you in the folder):

```bash
# Verify files exist
ls -la
# Should see:
# - train.ipynb       (training notebook)
# - requirements.txt  (dependencies)
# - README.md         (instructions)
# - models/           (empty folder for outputs)
# - dvc_storage/      (local remote)
```

If not already committed:
```bash
git add train.ipynb requirements.txt README.md
git commit -m "Add training notebook and requirements"
```

---

## A6. Add Initial Data (Phase 1: 50 Cat + 50 Dog)

```bash
# Create data folders
mkdir -p data/Cat data/Dog

# Copy FIRST 50 Cat images from source
# (Adjust source path as needed)
ls /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Cat/ | head -50 | xargs -I {} cp /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Cat/{} data/Cat/

# Copy FIRST 50 Dog images from source
ls /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Dog/ | head -50 | xargs -I {} cp /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Dog/{} data/Dog/

# Verify counts
ls data/Cat | wc -l   # Should show: 50
ls data/Dog | wc -l   # Should show: 50
```

---

## A7. Track Data with DVC

```bash
# Add data folder to DVC tracking
dvc add data/

# This creates:
# - data.dvc (pointer file, ~100 bytes)
# - Updates .gitignore to ignore data/

# Verify
ls -la data.dvc
cat data.dvc
# Shows MD5 hash of the data folder

# Commit the pointer file to Git
git add data.dvc .gitignore
git commit -m "Add dataset v1: 100 images (50 Cat + 50 Dog)"

# Tag this version for easy reference
git tag data-v1
```

---

## A8. Push Data to DVC Remote

```bash
# Push data to the local remote storage
dvc push

# Verify - data is now in dvc_storage/
ls dvc_storage/files/md5/
# Should show folders with cached data
```

---

## A9. Push to GitHub

```bash
# Create repo on GitHub first (via web interface)
# Then:
git remote add origin https://github.com/YOUR_USERNAME/cat_dog_mlops.git
git branch -M main
git push -u origin main
git push origin data-v1  # Push the tag
```

---

## A10. Test Clone & Pull (Verify Student Experience)

```bash
# Go to a different folder
cd /tmp

# Clone (data folder will be EMPTY)
git clone https://github.com/YOUR_USERNAME/cat_dog_mlops.git test_clone
cd test_clone

# Verify data is NOT there yet
ls data/
# Should be empty or not exist

# Pull data from DVC remote
dvc pull

# Now data appears!
ls data/Cat | wc -l   # Should show: 50
ls data/Dog | wc -l   # Should show: 50
```

---

# PART B: STUDENT EXPERIENCE (What Students Do)

Students clone the repo and run experiments.

---

## B1. Clone Repository

```bash
# Clone the repo (small download - no data yet!)
git clone https://github.com/INSTRUCTOR/cat_dog_mlops.git
cd cat_dog_mlops
```

---

## B2. Set Up Environment

```bash
# Create virtual environment
python -m venv venv

# Activate it
source venv/bin/activate        # Linux/Mac
# venv\Scripts\activate         # Windows

# Install dependencies
pip install -r requirements.txt
```

---

## B3. Pull Data with DVC

```bash
# Download the data (from DVC remote)
dvc pull

# Verify data is present
ls data/Cat | wc -l   # Should show: 50
ls data/Dog | wc -l   # Should show: 50
```

---

## B4. Run Training (Phase 1)

```bash
# Start Jupyter
jupyter notebook train.ipynb
```

In the notebook:
1. Verify config cell shows:
   ```python
   LEARNING_RATE = 0.001
   EPOCHS = 5
   UNFREEZE_LAYER4 = False
   DATA_VERSION = "v1"
   ```
2. **Run all cells** (Cell → Run All)
3. Training will complete and log to MLflow

---

## B5. View Experiment in MLflow

```bash
# Start MLflow UI (in a new terminal, same folder)
cd cat_dog_mlops
source venv/bin/activate
mlflow ui
```

Open http://localhost:5000 in browser.

**What you'll see:**
- Experiment: "Cat-Dog-Classifier"
- Run 1: resnet18-v1-XXXX
- Parameters: lr=0.001, epochs=5, etc.
- Metrics: accuracy, precision, recall, F1, AUC
- Artifacts: confusion_matrix.png, roc_curve.png, model

---

## B6. Phase 2: Change Hyperparameters

In notebook, modify the config cell:
```python
LEARNING_RATE = 0.0001   # Changed from 0.001
EPOCHS = 10              # Changed from 5
```

**Run all cells again.**

Back in MLflow UI:
- See Run 2 appear
- Select both Run 1 and Run 2
- Click "Compare"
- See how metrics changed!

---

## B7. Phase 3: More Data + Fine-tuning

This phase requires instructor to push data-v2 first (see Part C below).

Students do:
```bash
# Get latest code + data pointer
git pull

# Checkout the new data version
git checkout data-v2

# Pull the new data
dvc pull

# Verify more images
ls data/Cat | wc -l   # Should show: 100
ls data/Dog | wc -l   # Should show: 100
```

In notebook:
```python
UNFREEZE_LAYER4 = True   # Changed from False
DATA_VERSION = "v2"      # Changed from "v1"
```

**Run all cells again.**

---

# PART C: ADDING MORE DATA (Instructor Does This for Phase 3)

---

## C1. Add More Images

```bash
cd /home/gvsr/Downloads/course/teaching/weeks/slides/cat_dog_mlops

# Add next 50 Cat images (total: 100)
ls /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Cat/ | tail -50 | xargs -I {} cp /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Cat/{} data/Cat/

# Add next 50 Dog images (total: 100)  
ls /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Dog/ | tail -50 | xargs -I {} cp /home/gvsr/Downloads/course/teaching/weeks/slides/data/small_dataset/Dog/{} data/Dog/

# Verify
ls data/Cat | wc -l   # Should show: 100
ls data/Dog | wc -l   # Should show: 100
```

---

## C2. Update DVC Tracking

```bash
# Update DVC (new hash will be computed)
dvc add data/

# data.dvc now has a NEW MD5 hash
cat data.dvc

# Commit new data version
git add data.dvc
git commit -m "Add dataset v2: 200 images (100 Cat + 100 Dog)"
git tag data-v2

# Push to remotes
dvc push           # Push data to DVC remote
git push           # Push code to GitHub
git push origin data-v2  # Push tag
```

---

## C3. View Data Diff

```bash
# See what changed between versions
dvc diff data-v1 data-v2

# Shows: added files, deleted files, modified files
```

---

# PART D: SUMMARY - WHO DOES WHAT, WHEN

## Instructor Flow

| Step | Command | What Happens |
|------|---------|--------------|
| 1 | `git init` + `dvc init` | Create repo |
| 2 | `dvc remote add -d local_remote ./dvc_storage` | Configure storage |
| 3 | Copy 50+50 images | Initial dataset |
| 4 | `dvc add data/` | Track with DVC |
| 5 | `git add data.dvc` + `git commit` + `git tag data-v1` | Version in Git |
| 6 | `dvc push` | Upload data |
| 7 | `git push` | Upload code |

## Student Flow

| Step | Command | What Happens |
|------|---------|--------------|
| 1 | `git clone <url>` | Get code (data folder empty) |
| 2 | `dvc pull` | Download data |
| 3 | Run notebook | Train + log to MLflow |
| 4 | `mlflow ui` | View experiments |
| 5 | Change params in notebook | Experiment |
| 6 | Run notebook again | New MLflow run |
| 7 | `git pull` + `dvc pull` | Get new data version |

---

# PART E: Q&A - COMMON QUESTIONS

## Q: How does `dvc pull` know where to get data?
**A:** DVC reads `.dvc/config` which specifies the remote. The `data.dvc` file contains the MD5 hash of the data. DVC fetches that exact version from the remote.

## Q: What if student doesn't have DVC remote access?
**A:** For local remote (this demo), students need the `dvc_storage/` folder OR instructor needs to share it. For cloud (S3/GCS), students need credentials.

## Q: Can students use `dvc repro`?
**A:** Yes, IF you create a `dvc.yaml` pipeline. Without it, `dvc repro` does nothing. In this demo, we focused on data versioning, not pipelines.

## Q: What's the difference between `git pull` and `dvc pull`?
**A:** 
- `git pull` = Get latest code + data.dvc pointer files
- `dvc pull` = Download the actual data files that the pointers reference

## Q: How do I switch between data versions?
```bash
git checkout data-v1    # Switch to v1 code/pointers
dvc checkout            # Sync data folder to match
# Now data/ has v1 data

git checkout data-v2    # Switch to v2
dvc checkout            # Sync data
# Now data/ has v2 data
```

---

# PART F: TROUBLESHOOTING

## Error: "No DVC remote configured"
```bash
dvc remote add -d local_remote ./dvc_storage
```

## Error: "Cache does not exist"
Data hasn't been pushed yet. Run `dvc push` from the original repo.

## Error: "Unable to acquire lock"
Another DVC process is running. Wait or delete `.dvc/lock`.

## Error: "Data file missing"
Run `dvc pull` to download data.
