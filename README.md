Zepto Data & AI Platform — Capstone Project
Overview
This repository contains the three connected modules required for the Masai School Certificate Program in Artificial Intelligence and Machine Learning capstone:
`data_pipeline/` — web scraping, cleaning, currency conversion, SQLite storage, SQL querying, and pandas-based validation.
`analytics/` — Titanic data profiling, EDA, preprocessing, classification, imbalance handling, Random Forest tuning, regression, and model persistence.
`support_assistant/` — a grounded Zepto policy assistant using local embeddings, ChromaDB, LangGraph, Pydantic structured output, and FastAPI.
The project description explicitly requires one public repository containing these three module folders plus one root `README.md`. It also requires the repository to document setup, end-to-end execution, and module design decisions.
> **Current archive note:** the supplied archive currently contains `support-assistant/` (hyphen) rather than the assignment's required `support_assistant/` (underscore). Rename the folder before submission if you are keeping the repository aligned exactly with the specification. The current support-assistant code also contains a syntax error (`return {"intent": intent}s`) that must be corrected before the API can run.
Repository Structure
```text
.
├── README.md
├── data_pipeline/
│   ├── README.md
│   ├── scrape_books.ipynb
│   └── books.db
├── analytics/
│   ├── README.md
│   ├── 01_eda.ipynb
│   ├── 02_modelling.ipynb
│   ├── titanic.csv
│   ├── clean_df.csv
│   └── best_titanic_pipeline.pkl
└── support_assistant/
    ├── README.md
    ├── support_assistant.py
    └── docs/
        └── ... policy documents ...
```
The archive supplied for review does not currently include a root README or module README files; this set supplies them.
Setup
The supplied project does not include a `requirements.txt`, so the environment must be created from the imports used by the notebooks/application.
A practical consolidated environment is:
```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate

pip install pandas numpy requests beautifulsoup4 matplotlib seaborn scikit-learn imbalanced-learn joblib jupyter
pip install sentence-transformers chromadb langgraph fastapi uvicorn pydantic python-dotenv groq
```
The analytics module uses `sns.load_dataset("titanic")` once to obtain the original Titanic data. The project description requires the resulting `titanic.csv` to be committed as the offline fallback.
The support assistant uses `all-MiniLM-L6-v2` through `sentence-transformers`. The first model initialization may download model files if they are not already cached locally.
Running the Modules
1. Data Pipeline
Open and run:
```bash
jupyter notebook data_pipeline/scrape_books.ipynb
```
The notebook:
scrapes five book categories from `books.toscrape.com`;
cleans price, rating, category, and availability fields;
converts GBP to INR using the required fixed rate of 1 GBP = 105.50 INR;
creates normalized `categories` and `books` SQLite tables;
executes five SQL queries;
reads SQL results with pandas;
reproduces a SQL join using `pd.merge()`.
The notebook output in the supplied archive shows 90 books across 5 categories, satisfying the project's minimum row/category scope.
2. Analytics
Run the notebooks in order:
```bash
jupyter notebook analytics/01_eda.ipynb
jupyter notebook analytics/02_modelling.ipynb
```
`01_eda.ipynb` loads the Titanic dataset, saves the offline copy, profiles and cleans the data, performs EDA, computes correlations/outliers, creates the data-story charts, and demonstrates standardization.
`02_modelling.ipynb` reads `clean_df.csv`, performs the stratified train/test split and train-only preprocessing pipeline, trains the three classifiers, compares imbalance strategies, tunes Random Forest, performs the regression side task, saves the fitted pipeline, and reloads it for prediction.
3. Support Assistant
After renaming the supplied `support-assistant/` directory to `support_assistant/` and correcting the syntax error noted above:
```bash
cd support_assistant
set MOCK_LLM=1
uvicorn support_assistant:app --host 0.0.0.0 --port 7860
```
On macOS/Linux:
```bash
export MOCK_LLM=1
uvicorn support_assistant:app --host 0.0.0.0 --port 7860
```
Then test:
```bash
curl -X POST http://127.0.0.1:7860/ask ^
  -H "Content-Type: application/json" ^
  -d "{"query":"What is the delivery fee for an order below INR 149?"}"
```
and a non-policy query:
```bash
curl -X POST http://127.0.0.1:7860/ask ^
  -H "Content-Type: application/json" ^
  -d "{"query":"What is the capital of India?"}"
```
Use the equivalent single-line commands for your shell if required.
Design Decisions
Data Pipeline
`requests` + BeautifulSoup are used for scraping.
Five predefined categories are scraped, producing 90 records in the supplied notebook output.
Rating words are mapped to integers 1–5.
Availability is converted to a boolean.
The required fixed conversion rate of 105.50 INR per GBP is used rather than a live currency API.
SQLite is normalized into a `categories` parent table and `books` child table connected by a foreign key.
SQL queries demonstrate filtering, ordering, limiting, distinct selection, range filtering, and joins.
Analytics
The Titanic dataset is loaded once in the EDA notebook and saved to `titanic.csv`.
The EDA stage removes redundant/derived columns and handles missing values before saving `clean_df.csv`.
The modeling notebook uses a stratified split and a scikit-learn `ColumnTransformer`/`Pipeline` so scaling and categorical encoding are learned from the training split.
Logistic Regression, Decision Tree, and Random Forest are evaluated.
Logistic Regression is additionally evaluated with class weighting and SMOTE.
Random Forest is tuned with `GridSearchCV` and an OOB-enabled estimator.
A separate linear regression predicts fare.
The complete Random Forest preprocessing + estimator pipeline is saved with joblib.
Support Assistant
Policy text is embedded locally with `all-MiniLM-L6-v2`.
ChromaDB stores the embeddings with cosine similarity.
LangGraph routes queries between policy retrieval and direct/general handling.
Mock mode is the intended offline baseline.
Pydantic validates the final `answer`, `sources`, and `confidence` structure.
FastAPI exposes the graph through `POST /ask`.
Important Submission Checks
Before submitting the repository, verify the assignment-specific acceptance criteria rather than relying only on the presence of these README files.
In particular:
Use the exact required directory names: `data_pipeline`, `analytics`, and `support_assistant`.
Include a `requirements.txt` (module-specific or consolidated) and document it here.
Ensure the support assistant contains the eight required corpus documents as separate files, or otherwise follows the assignment's exact corpus requirement.
Add and test the required Dockerfile for the support assistant.
Record the two required FastAPI example responses in the support-assistant README.
Ensure the support assistant runs with `MOCK_LLM` at its default value.
Ensure the repository history contains a feature branch with at least two commits followed by a merge into `main`.
Review the module READMEs for implementation-specific gaps identified from the supplied code.
