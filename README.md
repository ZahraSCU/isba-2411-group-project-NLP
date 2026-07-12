# Medication Review NLP: Sentiment and Side-Effect Detection

**ISBA 2411 — Group 1 Project**
## Team Members

- Zahra Fahimfar
- Varsha Pai
- Krystle Dario
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ZahraSCU/isba-2411-group-project-NLP/blob/main/Milestone2_Baseline_and_Representation_%28Zahra%29.ipynb)

## Project Overview

This project explores how natural language processing (NLP) can analyze unstructured medication-related text and support the early identification of possible adverse drug reactions.

Patients often describe symptoms such as dizziness, nausea, fatigue, rash, or mood changes without directly connecting them to a medication. Reviewing large volumes of these messages manually can be slow and inconsistent. The long-term goal of this project is to develop a decision-support system that helps clinical staff prioritize potentially important medication-related messages for human review.

The current prototype uses medication reviews from Drugs.com to establish an initial sentiment-classification baseline and compare traditional and contextual text representations. This baseline is an early step toward more advanced side-effect and medical-entity extraction.

> **Important:** This is an educational research prototype. It is not a diagnostic system and should not replace professional medical judgment.

## Project Objectives

- Classify medication-review sentiment as **positive**, **neutral**, or **negative**.
- Compare a traditional **TF-IDF** representation with contextual **Sentence-Transformer embeddings**.
- Evaluate model performance using accuracy, precision, recall, and F1-score.
- Examine whether performance differs between mental-health-related reviews and other medication reviews.
- Extend the system toward extracting medication names, symptoms, side effects, timing, and severity.
- Prioritize recall in future adverse-event detection so potentially serious cases are less likely to be missed.

## Dataset

The current notebook uses a Drugs.com medication-review dataset stored as:

```text
drugsComTrain_raw.xlsx
```

Important fields include:

| Field | Description |
|---|---|
| `drugName` | Medication name |
| `condition` | Condition associated with the review |
| `review` | Patient-written medication review |
| `rating` | Numeric rating from 1 to 10 |
| `usefulCount` | Number of users who found the review useful |

The dataset is not included in this repository. To run the notebook, place the file in an accessible Google Drive folder and update the path in the data-loading cell.

## Label Construction

The numeric ratings are converted into three sentiment classes:

| Rating | Sentiment label |
|---|---|
| 7–10 | Positive |
| 5–6 | Neutral |
| 1–4 | Negative |

For the Milestone 2 experiment, the notebook creates a reproducible sample of **50,000 reviews**, stratified by both sentiment label and analysis group.

Current sampled distribution:

| Category | Count |
|---|---:|
| Positive | 33,150 |
| Negative | 12,411 |
| Neutral | 4,439 |
| Mental-health conditions | 10,300 |
| Other conditions | 39,700 |

Mental-health-related reviews are identified using condition keywords such as depression, anxiety, bipolar disorder, panic, ADHD, and insomnia.

## Methodology

### 1. Data Preparation

- Mount Google Drive in Colab.
- Load the Excel dataset with pandas.
- Remove records missing the review, condition, or rating.
- Clean quotation marks and extra whitespace.
- Construct sentiment labels from ratings.
- Create mental-health and non-mental-health groups.
- Draw a stratified sample using a fixed random seed.

### 2. Static Text Representation

The first representation uses **TF-IDF** with:

- English stop-word removal
- A maximum vocabulary of 5,000 features
- A resulting matrix of 50,000 documents × 5,000 features

TF-IDF provides a fast and interpretable baseline based on word importance.

### 3. Contextual Text Representation

The second representation uses the Sentence-Transformers model:

```text
all-MiniLM-L6-v2
```

Unlike TF-IDF, contextual embeddings represent each review as a dense vector intended to capture semantic meaning beyond exact word overlap.

### 4. Baseline Evaluation

The notebook compares model performance across the two text representations and reports:

- Accuracy
- Precision
- Recall
- F1-score
- Class-level classification results
- Performance for mental-health and other condition groups

The complete outputs and interpretation are available in the Milestone 2 notebook.

## Repository Contents

```text
.
├── README.md
└── Milestone2_Baseline_and_Representation_(Zahra).ipynb
```

Additional milestone documents and notebooks may be added as the project develops.

## How to Run the Notebook

1. Open the notebook using the **Open in Colab** badge at the top of this README.
2. Run the setup cell to install the required packages.
3. Mount Google Drive when prompted.
4. Update the Google Drive directory and `DATA_PATH` so they point to your copy of the dataset.
5. Run the notebook cells in order.

The main dependencies are:

```text
pandas
numpy
scikit-learn
sentence-transformers
openpyxl
```

A Colab GPU is recommended when creating embeddings for the full 50,000-review sample.

## Current Limitations

- Ratings are used as sentiment labels, so the labels may not perfectly represent the full meaning of each review.
- The classes are imbalanced, especially the neutral class.
- Reviews are not the same as real patient portal messages.
- The mental-health grouping is currently keyword-based.
- Sentiment classification alone does not identify a specific side effect or determine whether a medication caused it.
- Model predictions must not be interpreted as medical advice.

## Planned Next Steps

- Extract medication, symptom, side-effect, dosage, timing, and severity entities.
- Add aspect-based sentiment analysis for effectiveness and adverse effects.
- Develop a dedicated possible-side-effect classifier.
- Add a rule-based safety layer for high-risk symptom phrases.
- Evaluate recall carefully, especially for serious adverse-event examples.
- Test performance across writing styles and clinical subgroups.
- Explore an LLM or retrieval-augmented generation layer for explainable summaries.
- Build a simple interactive demonstration for reviewing model predictions.

## Privacy and Responsible Use

Future clinical versions of this work would require de-identified data, secure storage, restricted access, bias testing, and human oversight. The system is intended to support—not replace—pharmacists, nurses, physicians, or other qualified healthcare professionals.

## Course Information

- **Course:** ISBA 2411
- **Project:** NLP System for Medication Review and Side-Effect Analysis
- **Team:** Group 1
- **Institution:** Santa Clara University
