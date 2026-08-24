# 🌫️ Karachi AQI Predictor

### AI-Powered 3-Day Air Quality Forecasting for Karachi, Pakistan

**Live Demo:** [AQI Predictor — Karachi](https://aqipredictor-kashaf.streamlit.app/)

---

## 📌 Project Overview

**Karachi AQI Predictor** is an end-to-end machine learning system designed to forecast Karachi's air quality for the upcoming **96 hours (3 days)**.

The system continuously collects air-quality and weather information, processes and transforms the data into meaningful time-series features, trains multiple regression models, and generates future AQI predictions through an interactive **Streamlit dashboard**.

The project is designed around real-world air pollution patterns in Karachi, where factors such as traffic emissions, industrial activity, particulate matter, weather conditions, and seasonal dust can significantly influence air quality.

### 🎯 Key Objectives

* Forecast Karachi's AQI for the next **3 days**
* Provide residents with an easy-to-understand pollution outlook
* Automate data collection and model retraining
* Maintain historical engineered data in a cloud database
* Compare multiple ML algorithms and select the strongest model
* Provide model interpretability through **SHAP**
* Make the complete system deployable and reproducible

---

## 🧠 Forecasting Strategy

Rather than directly training separate models to predict AQI 24, 48, or 72 hours into the future, this project uses a **recursive forecasting strategy**.

The model is trained to predict:

```text
Next Hour AQI = AQI(t + 1)
```

The predicted value is then used as an input for the following prediction:

```text
t+1 → prediction
t+2 → prediction based on t+1
t+3 → prediction based on t+2
...
t+96 → final prediction
```

This approach takes advantage of the strong temporal relationship between consecutive AQI measurements.

The single-step model achieves approximately **R² ≈ 0.90** on real Karachi air-quality data, while the recursive forecasting pipeline extends this capability to produce a complete **96-hour outlook**.

---

# ⚙️ System Pipeline

The project follows an automated data-to-prediction workflow:

```text
Open-Meteo APIs
      ↓
Data Collection
      ↓
Cleaning & Preprocessing
      ↓
Time-Series Feature Engineering
      ↓
MongoDB Feature Store
      ↓
Model Training & Comparison
      ↓
Best Model Selection
      ↓
Model Registry / GridFS
      ↓
Recursive 96-Hour Forecast
      ↓
Streamlit Dashboard / REST API
```

---

## 🌍 1. Data Collection

Environmental information is obtained from **Open-Meteo's Air Quality and Weather APIs**.

The pipeline works with variables including:

### Air Quality

* PM2.5
* PM10
* CO
* CO₂
* NO₂
* SO₂
* O₃
* Dust
* US AQI

### Weather

* Temperature
* Relative humidity
* Wind speed and direction
* Cloud cover
* UV index
* Precipitation-related information

Hourly observations are used to preserve the temporal behavior of Karachi's pollution levels.

---

## 🧹 2. Data Processing & Feature Engineering

Raw environmental data is transformed into model-ready time-series features.

The preprocessing pipeline includes:

* Missing-value handling using forward filling and interpolation
* IQR-based outlier capping
* Hour-based cyclical encoding
* Wind direction conversion into U/V components
* Rolling statistics
* AQI rate-of-change features
* PM2.5-to-PM10 ratio
* Rush-hour indicators
* Precipitation likelihood features

The model also incorporates:

* **36-hour rolling averages**
* **72-hour rolling standard deviations**

These features allow the model to capture both short-term fluctuations and longer pollution trends.

---

## 🗄️ 3. Feature Store & Model Registry

Processed hourly observations are stored in **MongoDB Atlas**.

The primary feature collection is:

```text
aqi_hourly_v1
```

Each record is identified by its:

```text
timestamp
```

A unique timestamp constraint prevents duplicate hourly observations.

Trained models and their associated preprocessing information are stored using **MongoDB GridFS**.

The model registry keeps track of:

```text
Model
Scaler
Feature List
```

This makes it possible for the deployed application to retrieve the latest trained model without requiring the model file to be manually bundled with the application.

---

# 🤖 4. Machine Learning

Several regression algorithms are evaluated before selecting the final forecasting model.

### Models Compared

| Model             | Purpose                                  |
| ----------------- | ---------------------------------------- |
| Random Forest     | Ensemble-based nonlinear regression      |
| Gradient Boosting | Sequential boosting for complex patterns |
| XGBoost           | High-performance gradient boosting       |
| SVR               | Support Vector Regression                |
| Ridge Regression  | Regularized linear baseline              |

A **MinMaxScaler** is applied where required, and the dataset is split chronologically using an **80/20 time-based split**.

This avoids randomly mixing future observations into the training data.

The model with the strongest evaluation performance is selected and then retrained using the complete historical dataset.

---

# 🔮 5. 96-Hour Forecasting

Once the latest model is available, the serving layer generates an hourly forecast for the next **96 hours**.

The forecast provides useful checkpoints such as:

```text
Current AQI
+24 Hours
+48 Hours
+72 Hours
+96 Hours
```

The dashboard converts these predictions into understandable pollution categories and highlights potentially unhealthy conditions.

An alert is displayed whenever the current or predicted AQI crosses **150**.

---

# 📊 Interactive Dashboard

The Streamlit interface provides users with an interactive view of Karachi's expected air quality.

The dashboard includes:

* Current AQI information
* 3-day forecast
* Hourly prediction trends
* AQI category indicators
* Pollution alerts
* Model information
* SHAP-based feature explanations
* Forecast visualizations

A local-model option is also available for demonstrations where MongoDB access is not required.

---

# 🔍 Model Explainability

The project uses **SHAP (SHapley Additive exPlanations)** to make model predictions easier to interpret.

Instead of showing only an AQI number, the dashboard can highlight which environmental features contributed most to the prediction.

This provides additional insight into relationships between variables such as:

* PM2.5
* PM10
* Temperature
* Humidity
* Wind
* Previous AQI values
* Time-based features

---

# 🛠️ Technology Stack

| Component            | Technology            |
| -------------------- | --------------------- |
| Data Source          | Open-Meteo APIs       |
| Programming Language | Python                |
| Machine Learning     | Scikit-learn, XGBoost |
| Explainability       | SHAP                  |
| Database             | MongoDB Atlas         |
| Model Storage        | MongoDB GridFS        |
| Dashboard            | Streamlit             |
| API                  | FastAPI               |
| Visualization        | Plotly                |
| Automation           | GitHub Actions        |
| Configuration        | YAML + `.env`         |

---

# 📁 Repository Structure

```text
AQI Predictor/
│
├── run_pipeline.py
├── config/
│   └── settings.yaml
│
├── src/
│   ├── dashboard.py
│   │
│   ├── data/
│   │   └── openmeteo_client.py
│   │
│   ├── features/
│   │   └── build_features.py
│   │
│   ├── models/
│   │   └── sklearn_trainer.py
│   │
│   ├── pipelines/
│   │
│   ├── serving/
│   │   └── predict.py
│   │
│   └── utils/
│       └── mongo_store.py
│
├── notebooks/
│   └── eda_quick.ipynb
│
└── .github/
    └── workflows/
```

---

# 🚀 Getting Started

## Requirements

Before running the project locally, make sure you have:

* Python **3.11+**
* MongoDB Atlas account
* GitHub account for automated workflows

Create a MongoDB Atlas database, configure the database user and network access, and place the connection string inside your environment file.

---

## 🔧 Installation

Clone the repository and create a virtual environment:

```bash
git clone <your-repo-url>
cd "AQI Predictor"

py -3.12 -m venv .venv311
.\.venv311\Scripts\Activate.ps1
```

For macOS/Linux:

```bash
source .venv311/bin/activate
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

Create the environment configuration:

```bash
cp .env.example .env
```

Then configure:

```text
MONGODB_URI=<your-mongodb-uri>
MONGODB_DB=aqi_predictor
```

---

# 🔄 Running the Pipeline

### Generate Historical Data

Create a historical dataset using the previous 365 days:

```bash
python run_pipeline.py backfill --days 365
```

To generate the dataset without MongoDB:

```bash
python run_pipeline.py backfill --days 365 --csv-only
```

### Train the Model

Using MongoDB:

```bash
python run_pipeline.py train
```

Using a local CSV:

```bash
python run_pipeline.py train --csv data/backfill.csv
```

### Run Hourly Feature Ingestion

```bash
python run_pipeline.py feature
```

---

# 🖥️ Launch the Application

### Streamlit Dashboard

```bash
streamlit run src/dashboard.py
```

The dashboard can also be configured to use a locally stored model for UI-only demonstrations.

### FastAPI Service

```bash
uvicorn src.serving.predict:app --reload
```

Available endpoints include:

```text
GET /health
GET /predict
GET /predict/local
```

---

# 📓 Exploratory Data Analysis

Install the notebook dependencies:

```powershell
pip install -r requirements-notebooks.txt
```

Register the environment with Jupyter:

```powershell
python -m ipykernel install --user --name aqi-predictor --display-name "AQI Predictor (.venv311)"
```

Launch Jupyter:

```bash
jupyter notebook notebooks/eda_quick.ipynb
```

Generated visualizations are stored under:

```text
notebooks/visuals/
```

---

# 📈 Model Metrics

To inspect locally stored model performance:

```bash
python show_model_metrics.py --local
```

Evaluation is based on a chronological train/test split rather than random sampling, making the validation setup more appropriate for time-series forecasting.

---

# ☁️ Automated CI/CD

The project uses **GitHub Actions** to reduce manual maintenance.

### Hourly Workflow

The feature pipeline periodically executes:

```bash
python run_pipeline.py feature
```

This retrieves and processes the latest environmental data.

### Daily Training Workflow

The training workflow executes:

```bash
python run_pipeline.py train
```

This allows the forecasting model to be periodically refreshed as new historical data becomes available.

Workflows can also be manually triggered from the GitHub Actions interface.

---

# 🔐 Environment Configuration

The application requires the following environment variables:

| Variable      | Description                                |
| ------------- | ------------------------------------------ |
| `MONGODB_URI` | MongoDB Atlas connection string            |
| `MONGODB_DB`  | Database name; defaults to `aqi_predictor` |

### MongoDB Collections

```text
aqi_hourly_v1
model_registry
GridFS
```

---

# 🚦 US AQI Classification

The dashboard follows the **US EPA AQI classification system**:

| AQI Range | Classification                 |
| --------: | ------------------------------ |
|      0–50 | Good                           |
|    51–100 | Moderate                       |
|   101–150 | Unhealthy for Sensitive Groups |
|   151–200 | Unhealthy                      |
|   201–300 | Very Unhealthy                 |
|      301+ | Hazardous                      |

The application highlights conditions requiring attention whenever the current or forecast AQI rises above **150**.

---

# 🌱 Why This Project Matters

Karachi's air pollution can change considerably over relatively short periods. A current AQI reading only describes the present situation, whereas a short-term forecast can help people make better decisions about upcoming outdoor activities.

This project combines:

**Real-time environmental data + Time-series feature engineering + Machine learning + Automated pipelines + Explainable AI + Interactive visualization**

into a single deployable forecasting system.

---

## 🌐 Live Application

### **Karachi AQI Predictor**

[Open the deployed Streamlit dashboard →](https://aqipredictor-kashaf.streamlit.app/)

---

## 📌 Project Highlights

* 🌍 Karachi-specific air-quality forecasting
* ⏱️ Hourly environmental data pipeline
* 🔮 96-hour recursive AQI prediction
* 🤖 Multiple ML models evaluated automatically
* 🗄️ MongoDB-based feature and model storage
* 📊 Interactive Streamlit dashboard
* 🔍 SHAP model explainability
* ⚡ FastAPI prediction endpoints
* 🔄 Automated hourly ingestion
* 🧠 Automated daily model retraining
* ☁️ Cloud-deployable architecture

---


---
