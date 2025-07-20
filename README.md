
````markdown
# 🧠 End-to-End Chest Cancer Classification using MLflow, DVC & AWS CI/CD 🚀

A complete MLOps-enabled Deep Learning project that classifies chest cancer images using a VGG16 model. The system integrates **data versioning**, **experiment tracking**, **model registry**, and **CI/CD deployment**.

---

## 📦 Project Structure & Core Modules

### `1. Data Ingestion (src/components/data_ingestion.py)`
Handles downloading and extracting the dataset from Google Drive using `gdown`.

- `download_file()` → Downloads dataset zip.
- `extract_zip_file()` → Extracts zip to usable format.

### `2. Prepare Base Model (src/components/prepare_base_model.py)`
- Loads pretrained **VGG16** without top layer.
- Freezes layers and adds new classification head.
- Saves the model for transfer learning.

### `3. Model Training (src/components/model_trainer.py)`
- Uses Keras `ImageDataGenerator` for training/validation.
- Supports augmentation, batch sizing, and configurable steps.
- Saves trained model to artifacts directory.

### `4. Evaluation (src/components/model_evaluation.py)`
- Loads trained model.
- Evaluates on validation data.
- Saves metrics (accuracy/loss) to `scores.json`.
- Logs run to **MLflow** and optionally registers it to the **Model Registry**.

---

## ⚙️ DVC Pipeline: End-to-End ML Workflow Automation

This project uses **DVC** (Data Version Control) to manage and reproduce the ML pipeline stages.

### 🔄 DVC Workflow Stages (Defined in `dvc.yaml`)

```yaml
stages:
  data_ingestion:
    cmd: python src/pipeline/stage_01_data_ingestion.py
    outs:
      - artifacts/data_ingestion

  prepare_base_model:
    cmd: python src/pipeline/stage_02_prepare_base_model.py
    deps:
      - artifacts/data_ingestion
    outs:
      - artifacts/prepare_base_model

  training:
    cmd: python src/pipeline/stage_03_model_training.py
    deps:
      - artifacts/prepare_base_model
      - artifacts/data_ingestion
    outs:
      - artifacts/training

  evaluation:
    cmd: python src/pipeline/stage_04_model_evaluation.py
    deps:
      - artifacts/training
    outs:
      - scores.json
```

### ✅ Each DVC stage:

| Stage                | Script                                      | Purpose                                  |
|---------------------|---------------------------------------------|------------------------------------------|
| `data_ingestion`    | `stage_01_data_ingestion.py`                | Downloads and unzips raw image dataset   |
| `prepare_base_model`| `stage_02_prepare_base_model.py`            | Loads and prepares VGG16 model           |
| `training`          | `stage_03_model_training.py`                | Trains the model on the dataset          |
| `evaluation`        | `stage_04_model_evaluation.py`              | Evaluates trained model + MLflow logging|

### 🛠️ Run Pipeline

```bash
dvc repro
```

This executes each stage in order, automatically skipping stages if no changes are detected in dependencies.

### 🔍 Visualize Pipeline

```bash
dvc dag
```

This shows a Directed Acyclic Graph (DAG) of your ML workflow, helping you understand the sequence and dependencies.

---

## 🧪 Experiment Tracking with MLflow

- Set environment variables:
```bash
export MLFLOW_TRACKING_URI=<your-mlflow-uri>
export MLFLOW_TRACKING_USERNAME=<username>
export MLFLOW_TRACKING_PASSWORD=<password>
```

- Launch MLflow UI:
```bash
mlflow ui
```

Each run logs:
- Metrics: `accuracy`, `loss`
- Parameters: `learning_rate`, `batch_size`, etc.
- Artifacts: Trained model
- Optionally: Registers model in a central registry (e.g., Dagshub-hosted MLflow)

---

## 🐳 Deployment via Docker + AWS EC2 & ECR

### 🧱 Deployment Workflow

1. **Build Docker image**
```bash
docker build -t chest-cancer-app .
```

2. **Push to ECR**
```bash
docker tag chest-cancer-app:latest <your-ecr-uri>
docker push <your-ecr-uri>
```

3. **Run on EC2**
```bash
sudo apt update && sudo apt install docker.io -y
docker pull <your-ecr-uri>
docker run -d -p 80:5000 chest-cancer-app
```

App now accessible on EC2 public IP.

---

## 🧠 Model Summary

- **Base Architecture**: VGG16
- **Trained Layers**: Custom dense layer only
- **Classification**: Binary (Cancer vs. No-Cancer)
- **Framework**: TensorFlow + Keras

---

## 📂 Directory Layout

```bash
.
├── artifacts/                # DVC-tracked outputs
├── config/                   # YAMLs for config, params
├── src/
│   ├── components/           # Core ML logic
│   ├── config/               # Config entity classes
│   └── pipeline/             # Orchestration scripts per stage
├── templates/                # HTML templates for app UI
├── app.py                    # Flask web interface
├── dvc.yaml                  # DVC workflow definition
├── Dockerfile                # For containerization
└── mlruns/                   # MLflow run logs
```

---

## 🧪 CI/CD with GitHub Actions + AWS Runner

- **Self-hosted EC2 runner** auto-pulls Docker image.
- Deploys model backend automatically when new changes pushed.

### Required GitHub Secrets

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `ECR_REPOSITORY_NAME`
- `AWS_ECR_LOGIN_URI`

---

## 🙋‍♂️ How to Contribute

1. Fork and clone this repo
2. Run `dvc repro` to execute full pipeline
3. Submit a pull request with improvements or fixes

---

## 📄 License

This project is licensed under the **MIT License**.

---

## 🙌 Acknowledgments

- VGG16 Pretrained Model — [Keras Applications](https://keras.io/api/applications/)
- DVC & MLflow — MLOps Open Source Tools
- Deployment — AWS EC2 + ECR + GitHub Actions

````


