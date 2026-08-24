

# 💊 MedReview Insight

**AI-powered analysis of patient-written medication reviews**

MedReview Insight is a research and educational NLP application that analyzes the satisfaction expressed in medication reviews. It predicts whether a review reflects **low**, **medium**, or **high** satisfaction, explains important language signals, and retrieves semantically similar patient experiences.

> **Research and educational use only.** This project summarizes patient-reported experiences and model predictions. It does not diagnose, prescribe, or recommend treatment.


https://github.com/user-attachments/assets/4c9b55f7-fa41-49c7-891f-36042b129c99






## Project Overview

Patient reviews contain useful information about effectiveness, side effects, symptoms, and overall experience, but this information is stored as unstructured text. MedReview Insight converts those reviews into a more transparent and searchable workflow:

1. **Analyze** — predict the satisfaction expressed in a medication review.
2. **Explain** — display class probabilities, evidence terms, and experience themes.
3. **Retrieve** — find semantically similar patient reviews.
4. **Ground** — support the result with retrieved evidence and safety-aware responses.

The user provides:

- Drug name
- Medical condition
- Patient-written review
- Number of evidence sources to retrieve

## Main Features

- Three-class satisfaction prediction: `low`, `medium`, or `high`
- Hybrid word- and character-level TF-IDF features
- Balanced averaged SGD logistic classifier
- Class probabilities and confidence visualization
- Interpretable evidence terms and patient-experience themes
- MiniLM semantic retrieval using `all-MiniLM-L6-v2`
- Similar-review evidence from the Drugs.com dataset
- Interactive Streamlit dashboard with example inputs
- Safety guardrails for dosage, prescribing, and treatment questions
- Professional navy-and-teal interface designed for presentation use

## Dataset

The project uses the **Drugs.com Medication Reviews Dataset**.

| Data | Reviews |
|---|---:|
| Original training file | 161,297 |
| Original test file | 53,766 |
| Classifier training sample | 90,000 |
| Evaluation sample | 15,000 |
| Semantic retrieval library | 35,000 |

The original numerical ratings are converted into satisfaction labels:

| Rating | Label |
|---|---|
| 1–4 | Low satisfaction |
| 5–6 | Medium satisfaction |
| 7–10 | High satisfaction |

The dataset is not included in this repository. Download the two Drugs.com CSV files separately and upload them when running the notebook.

Accepted filenames:

```text
drugsComTrain_raw.csv
drugsComTest_raw.csv
```

The notebook also accepts filenames containing `(1)`, such as `drugsComTrain_raw(1).csv`.

## Model Architecture

```text
Drug + Condition + Review
          │
          ▼
Text cleaning and preprocessing
          │
          ├──► Word TF-IDF (1–2 grams)
          │
          └──► Character TF-IDF (3–5 grams)
                       │
                       ▼
       Balanced averaged SGD classifier
                       │
                       ▼
   Satisfaction label + probabilities + evidence

Review query ──► MiniLM embedding ──► nearest-neighbor search
                                      │
                                      ▼
                           Similar patient reviews
```

TF-IDF is used for the final classification decision because it performed best in the experiment. MiniLM is retained as the semantic retrieval model because it can identify reviews with similar meaning even when they use different words.

## Model Performance

Results on the **15,000-review evaluation sample**:

| Model | Accuracy | Macro-F1 |
|---|---:|---:|
| Majority-class baseline | 65.9% | 0.265 |
| **Hybrid word + character TF-IDF** | **83.3%** | **0.710** |
| MiniLM embeddings + logistic regression | 61.7% | 0.524 |

The final TF-IDF classifier improved accuracy by **17.4 percentage points** over the majority-class baseline. Macro-F1 is also reported because the three satisfaction classes are imbalanced, especially the medium class.

## Repository Structure

```text
medreview-insight/
├── VP_Experiment_Eye_Catching_Final.ipynb   # Complete Colab workflow
├── streamlit_app.py                         # Generated Streamlit application
├── model_artifacts/                         # Generated model and retrieval files
│   ├── tfidf_logreg_pipeline.joblib
│   ├── train_embeddings.npy
│   ├── train_model.parquet
│   └── embed_model_name.txt
├── requirements.txt                         # Python dependencies, if included
├── data/                                    # Local dataset folder; do not commit
├── .gitignore
└── README.md
```

`streamlit_app.py` and `model_artifacts/` are created by the final notebook after model training and artifact export.

## Run in Google Colab

This is the recommended method.

1. Upload `VP_Experiment_Eye_Catching_Final.ipynb` to Google Colab.
2. Upload both Drugs.com CSV files to `/content/` using the Colab Files panel.
3. Select **Runtime → Run all**.
4. Wait for preprocessing, model training, MiniLM encoding, and artifact export to finish.
5. Run the final **Launch the app** cell.
6. Click the green **Open MedReview Insight** button printed by the notebook.

The launch cell starts Streamlit on port `8501`, checks the application health endpoint, and opens the app through Colab's built-in port proxy. No ngrok token is required.

### Colab Runtime Notes

- A GPU is helpful for MiniLM embedding generation but is not required.
- A T4 GPU usually makes the embedding section considerably faster.
- Depending on Colab resources, the complete first run may take approximately **10–80 minutes**.
- Do not close or disconnect the Colab runtime while the app is running.
- Colab links are temporary and stop working after the runtime disconnects.

## Run Locally

### 1. Create a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

### 2. Install dependencies

If the repository contains `requirements.txt`:

```bash
python -m pip install -r requirements.txt
```

Otherwise:

```bash
python -m pip install numpy pandas scikit-learn sentence-transformers streamlit plotly joblib pyarrow
```

### 3. Generate the model artifacts

Place the two CSV files in the project root or `data/`, then run the notebook through the artifact-export section. Confirm that `streamlit_app.py` and `model_artifacts/` were created.

### 4. Start the application

```bash
python -m streamlit run streamlit_app.py
```

Open the local URL displayed in the terminal, normally:

```text
http://localhost:8501
```

## Troubleshooting

### Dataset file not found

Upload both CSV files and confirm their names. The notebook searches the project root, `/content/`, and `data/`.

### Model artifacts not found

Run the notebook through the artifact-export section before launching Streamlit. The app requires the classifier, embeddings, retrieval corpus, and embedding-model name.

### The Colab app shows a blank page

- Return to the notebook and confirm that the launch cell reports **Streamlit is healthy**.
- Open the URL from the green button rather than an old or blank browser tab.
- Run the Streamlit log cell to view the actual error.
- If the runtime disconnected, reconnect and rerun the required notebook cells.

### MiniLM encoding is slow

- Use a T4 GPU when Colab makes one available.
- Keep the semantic retrieval library at 35,000 reviews for the reproducible project configuration.
- Do not rerun the embedding cell unless the runtime was reset or the data changed.

### Streamlit is already using port 8501

Stop the previous Streamlit process or restart the runtime before launching the app again.

## Limitations

- Patient ratings and reviews are subjective and may be noisy or incomplete.
- The medium-satisfaction class is the most difficult to predict.
- Reviews can contain misspellings, sarcasm, reporting bias, and missing context.
- Model probabilities are not fully calibrated.
- Retrieved reviews are anecdotal patient experiences, not clinical evidence.
- The application must not be used to make medical decisions.

## Future Work

- Calibrate the model's probability estimates
- Evaluate performance by medication and condition
- Investigate high-confidence classification errors
- Improve prediction of the medium-satisfaction class
- Perform a larger human evaluation of retrieval relevance
- Add stronger automated tests and deployment monitoring

## Team

**ISBA 2411 — Group 1**

- Zahra Fahimfar
- Varsha Pai
- Krystle Jozen Dario

## Technology Stack

- Python
- pandas and NumPy
- scikit-learn
- Sentence Transformers / MiniLM
- Streamlit
- Plotly
- Google Colab

## Acknowledgments

- Drugs.com Medication Reviews Dataset
- Sentence Transformers `all-MiniLM-L6-v2`
- ISBA 2411 course materials and project guidance
