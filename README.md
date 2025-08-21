# Log Classification Project

Pipeline to classify system logs using a combination of regex rules, a lightweight BERT embedder, and an LLM for fallback input input.

## Quick start

1. Create/activate your Python venv and install dependencies:

   ```sh
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

2. Run the classifier (reads `resources/test.csv`, writes `resources/output.csv`):

   ```sh
   python classify.py
   ```

   - Input sample: `resources/test.csv`  
   - Output: `resources/output.csv`

## Project layout

- `classify.py`  entry point and pipeline. Main functions:
  - `classify(logs)`  batch classify list of (source, log_message).
  - `classify_log(source, log_msg)`  single log routing logic.
  - `classify_csv(input_file)`  read CSV, classify, write output.
- Processor modules:
  - `processor_regex.py`  rule-based matching.
  - `processor_bert.py`  fallback semantic classification (BERT embeddings / model).
  - `processor_llm.py`  used for `LegacyCRM` logs.
- Training artifacts:
  - `training/training.ipynb`  clustering, model training and export steps.
  - `models/log_classifier.joblib`  trained classifier (if present).
- Project helpers:
  - `main.py`  sample script scaffold.
  - `requirements.txt`  Python dependencies.
  - `package.json`  (if any JS tooling).

## Architecture (high-level)

1. Input ingestion
   - CSV or in-memory list of tuples (source, log_message)  see `classify.py`.

2. Routing logic (`classify_log`)
   - If `source == "LegacyCRM"` -> LLM processor: `processor_llm.classify_with_llm`
   - Else try rule-based regex: `processor_regex.classify_with_regex`
   - If regex returns None -> semantic BERT-based classifier: `processor_bert.classify_with_bert`

3. Persistence / outputs
   - Classified rows written to `resources/output.csv` by `classify.classify_csv`.
   - Models are trained in `training/training.ipynb` and saved to `models/log_classifier.joblib`.

Flow (textual):

Input CSV -> `classify.py` (classify_csv)  
  ├─> For each row -> classify_log  
  │   ├─> processor_regex (fast, deterministic)  
  │   ├─> processor_bert (semantic fallback)  
  │   └─> processor_llm (legacy source)  
  └─> Append target_label -> `resources/output.csv`

## How to retrain / update model

- Open and run `training/training.ipynb`.
- The notebook trains models, clusters messages, and saves the classifier (example output: `models/log_classifier.joblib`).

## Notes & tips

- Regex rules are quick and deterministic; keep them in `processor_regex.py`.
- BERT/embedding-based classification is heavier but better for unseen variations — see `processor_bert.py`.
- LLM usage is limited to legacy data to control cost/latency — see `processor_llm.py`.
- Use the notebook to inspect clusters and tune regex or balance training data.
- Keep secrets out of the repo. You exposed a GROQ API key in `.env`  revoke/rotate it now and keep `.env` in `.gitignore`. Use `.env.example` for templates.




