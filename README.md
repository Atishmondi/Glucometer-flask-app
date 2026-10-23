# Glucometer Glucose Forecaster

A Flask research prototype that uses a Transformer model to forecast six glucose readings at five-minute intervals (30 minutes ahead) from the previous 24 readings (two hours).

[![Watch the Glucometer demo](docs/demo-preview.jpg)](https://github.com/user-attachments/assets/d0d70548-f459-4dec-bc95-7179d46e0ad9)

*Watch the 22-second demo · XML upload → glucose forecast*

## Included

- Flask application with XML upload and drag-and-drop support.
- Pretrained Keras model in `glucometer_app/model/glucometer_transformer_model.h5`.
- Chart.js history/forecast chart, summary statistics, and PNG chart download.
- Training notebook with Transformer and LSTM experiments; saved outputs are removed.
- Synthetic XML example for trying the upload flow without patient data.

## Setup

Use Python 3.11 in a virtual environment. Run these commands from the repository root:

```bash
git clone https://github.com/Atishmondi/Glucometer-flask-app.git
cd Glucometer-flask-app
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python run.py
```

On Windows, activate with `.venv\Scripts\activate` instead. Open http://127.0.0.1:5000 and upload `examples/synthetic_patient.xml`.

The development server starts in debug mode. This repository is not configured for production hosting. Chart.js and other frontend assets loaded from CDNs require internet access.

## Repository layout

```text
config.py
run.py
requirements.txt
requirements-notebook.txt
glucometer_app/
  __init__.py
  ml_utils.py
  main/
    __init__.py
    routes.py
  model/
    glucometer_transformer_model.h5
  templates/
    index.html
    result.html
examples/
  synthetic_patient.xml
notebooks/
  Glucometer_project.ipynb
```

Styles and scripts are embedded in the templates; no separate static directory is needed.

## XML format

The parser expects event groups with `ts` and `value` attributes. Timestamps use `DD-MM-YYYY HH:MM:SS`:

```xml
<patient id="synthetic-demo">
  <glucose_level>
    <event ts="01-01-2025 08:00:00" value="120"/>
    <!-- Supply at least 24 five-minute readings; see the complete example. -->
  </glucose_level>
  <meal><event ts="01-01-2025 08:30:00" value="30"/></meal>
  <bolus><event ts="01-01-2025 08:30:00" value="2"/></bolus>
</patient>
```

Glucose, meal, and bolus values become the glucose, carbohydrates, and insulin features. Data is resampled to five-minute intervals; glucose gaps are interpolated and missing meal/bolus values are filled with zero. The last 24 processed rows feed the model.

## Training notebook

```bash
python -m pip install -r requirements-notebook.txt
jupyter lab notebooks/Glucometer_project.ipynb
```

The notebook expects an independently obtained `archive.zip` containing `559-ws-training.xml` from the Ohio dataset in its working directory. The training dataset is not distributed here. Obtain it under its applicable access and usage terms before running the notebook. The synthetic example is for the application demo, not for reproducing training results.

The notebook contains Transformer and LSTM training/evaluation experiments. It does not currently include a complete export procedure that reproduces the bundled H5 artifact.

## Status and limitations

- The source, model artifact, and demonstration files are included. A full runtime/model-loading test was not performed during repository packaging because TensorFlow was unavailable in the packaging environment.
- The application fits a new MinMaxScaler on each uploaded file; the original training scaler is not bundled. This can make inference scaling inconsistent with training.
- The notebook normalizes before splitting and creates overlapping windows before its chronological split. Evaluation needs review for leakage before making accuracy claims.
- Invalid XML, insufficient history, and model-loading failures do not yet have comprehensive user-facing error handling.
- Uploads are processed in memory by the current routes. No user accounts, database persistence, live sensor integration, or deployment configuration is included.
- The chart's green line is a fixed 100 mg/dL visual reference, not a personalized clinical target.

This is an educational/research prototype, not a clinically validated medical device. Do not use predictions for treatment or insulin-dosing decisions.

## Author and licensing

Atish Mondi — https://www.linkedin.com/in/atishmondi

The previous README mentioned MIT, but no license file accompanied the source. No license grant is asserted here pending confirmation of source and model licensing.
