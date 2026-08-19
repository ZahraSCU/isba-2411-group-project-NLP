# Governance and Risk Appendix — MedReview Insight

**Course:** ISBA 2411  
**Team:** Zahra Fahimfar, Varsha Pai, Krystle Jozen Dario  
**System:** MedReview Insight

## 1. Purpose and Scope

MedReview Insight is a research and educational NLP system that analyzes patient-written medication reviews. It predicts the satisfaction expressed in a review, displays supporting language signals, and retrieves semantically similar patient experiences.

### Intended uses

- Demonstrating an end-to-end NLP classification and retrieval workflow
- Studying language patterns in patient-written medication reviews
- Predicting coarse satisfaction labels: low, medium, or high
- Exploring similar patient experiences in the Drugs.com review dataset
- Supporting classroom evaluation, research discussion, and model auditing

### Prohibited and out-of-scope uses

- Diagnosing a medical condition
- Recommending a medication, dosage, or treatment
- Advising a person to begin, stop, or change medication
- Emergency triage or crisis assessment
- Replacing a physician, pharmacist, or other qualified professional
- Making insurance, employment, eligibility, or access-to-care decisions
- Production deployment without additional clinical, legal, security, and fairness review

The application displays the following warning:

> **Research and educational use only.** This system summarizes patient-reported experiences and model predictions. It does not diagnose, prescribe, or recommend treatment.

## 2. Data Governance

### Data source

The project uses the Drugs.com Medication Reviews Dataset. The original source files contain 161,297 training reviews and 53,766 test reviews. The final experiment uses:

- 90,000 reviews for classifier training
- 15,000 reviews for evaluation
- 35,000 reviews for the semantic retrieval library

### Data characteristics

Each record can include a medication name, condition, review text, numerical rating, date, and useful-vote count. Numerical ratings are converted into three satisfaction labels:

- Low: ratings 1–4
- Medium: ratings 5–6
- High: ratings 7–10

The rating is treated as a proxy for patient satisfaction. It is not a clinical outcome or verified assessment of treatment effectiveness.

### Data handling rules

- Do not commit the raw dataset to a public repository unless its license explicitly permits redistribution.
- Do not add names, contact information, medical-record numbers, or other direct identifiers.
- Avoid displaying or republishing full review text when a short excerpt is sufficient.
- Do not combine the review data with external identity or demographic datasets.
- Store secrets and credentials outside the notebook and repository.
- Remove notebook outputs that contain credentials, private links, or unnecessary sensitive text before sharing.

## 3. Bias and Fairness Risks

### Identified bias risks

| Risk | Why it matters | Current mitigation |
|---|---|---|
| Self-selection bias | People with unusually positive or negative experiences may be more likely to post reviews. | Clearly describe the data as anecdotal and avoid population-level medical claims. |
| Class imbalance | High-satisfaction reviews are much more common, while the medium class is relatively small. | Use class-balanced training and report Macro-F1 in addition to accuracy. |
| Uneven medication and condition coverage | Common medications may dominate the learned patterns and retrieval results. | Display medication and condition context and recommend subgroup evaluation. |
| Missing demographic information | The dataset does not support reliable evaluation across age, sex, race, disability, or other protected groups. | Do not claim demographic fairness; document this as an unresolved limitation. |
| Language and writing-style bias | Misspellings, review length, fluency, sarcasm, and nonstandard English can affect predictions. | Combine word- and character-level TF-IDF and audit errors across text length and language quality. |
| Rating-label ambiguity | Ratings 5–6 can represent mixed or unclear experiences. | Maintain a separate medium class and review per-class precision, recall, and F1. |

### Fairness claims

This project does **not** claim demographic fairness. The dataset lacks the demographic attributes and representative sampling needed for a valid demographic fairness analysis.

## 4. Privacy Risks

Although the source reviews are publicly available, review text may contain personal medical details or information that could indirectly identify an individual.

### Privacy risks

- A review may include a name, location, age, date, or unusual medical history.
- Retrieved reviews may expose more patient text than is necessary for the analysis.
- Application or notebook logs could retain user-entered health information.
- Public screenshots or demonstrations could unintentionally display sensitive text.

### Privacy mitigations

- Use fictional or de-identified inputs during presentations and recorded demonstrations.
- Display only the minimum review excerpt needed to explain a result.
- Do not store user inputs or conversation history outside the temporary session.
- Do not enable analytics or external logging that captures patient text.
- Do not commit raw inputs, session logs, or generated artifacts containing private text.
- Review screenshots and notebook outputs before publishing them.
- If deployed beyond coursework, implement access controls, retention limits, encryption, deletion procedures, and a formal privacy review.

## 5. Failure Modes

| Failure mode | Potential impact | Detection | Mitigation |
|---|---|---|---|
| Incorrect satisfaction classification | The displayed label may misrepresent the review. | Evaluate on held-out data; inspect confusion matrices and per-class metrics. | Show probabilities and evidence; do not treat the prediction as medical truth. |
| Medium reviews predicted as low or high | Mixed experiences may be oversimplified. | Monitor medium-class precision, recall, and F1. | Keep the medium class and flag uncertain predictions for manual interpretation. |
| Spurious evidence terms | Highlighted words may be correlated with a label without explaining the patient's experience. | Manually inspect evidence for correct and incorrect examples. | Label evidence as model signals, not causal explanations. |
| Irrelevant semantic retrieval | MiniLM may retrieve text that is linguistically similar but medically unrelated. | Review similarity scores and sampled retrieval results. | Display medication, condition, rating, and similarity; limit the number of results. |
| Unsupported chatbot response | A response may go beyond the retrieved evidence or imply medical advice. | Test prohibited prompts and compare responses with retrieved evidence. | Use deterministic, evidence-grounded responses and refuse treatment-related requests. |
| Overconfidence and automation bias | Users may trust a high probability as medical certainty. | Conduct user testing and review high-confidence errors. | Display a clear disclaimer and distinguish model confidence from medical certainty. |
| Missing or corrupted artifacts | The application may fail to start or return incomplete results. | Startup checks verify all required artifact files and Streamlit health. | Stop with a clear error and instruct the user to rerun artifact export. |
| Colab disconnection | The temporary application URL stops working. | Health check or connection failure. | Keep an application screenshot for presentation backup and rerun the launch cell after reconnecting. |
| Dataset or concept drift | Performance may decline on newer reviews or different populations. | Compare new data distributions and periodically reevaluate metrics. | Retrain and validate on current, representative data before broader use. |
| Malicious or unexpected input | Extremely long, empty, or adversarial text may cause poor output or resource problems. | Input validation and application logs that exclude sensitive content. | Require nonempty input, limit input length, sanitize text, and fail safely. |

## 6. Medical-Safety Controls

The application is designed to analyze expressed satisfaction, not to provide clinical guidance.

### Refused requests

The assistant rejects requests involving:

- Dosage or dose changes
- Medication recommendations
- Prescribing decisions
- Starting or stopping medication
- Treatment planning
- Diagnosis

### Safe-response behavior

- State that the system cannot provide medical advice.
- Explain that results summarize patient-written reviews.
- Direct users to a qualified healthcare professional for medical decisions.
- Avoid presenting retrieved reviews as proof that a medication is safe or effective.

## 7. Model Evaluation and Transparency

The final classifier is evaluated on a held-out sample of 15,000 reviews.

| Model | Accuracy | Macro-F1 |
|---|---:|---:|
| Majority-class baseline | 65.9% | 0.265 |
| Hybrid word + character TF-IDF | 83.3% | 0.710 |
| MiniLM embeddings + logistic regression | 61.7% | 0.524 |

The hybrid TF-IDF model improves accuracy by 17.4 percentage points over the majority-class baseline. Macro-F1 is reported because accuracy alone can hide weak performance on the smaller medium class.

### Transparency provided in the application

- Predicted satisfaction label
- Class probabilities
- Model confidence
- Evidence terms and experience themes
- Semantically similar reviews
- Medication, condition, rating, and similarity context
- Research-use and medical-safety warnings

These elements improve inspectability but do not make the model clinically validated or fully explainable.

## 8. Security and Secret Management

- Never commit API keys, access tokens, passwords, or ngrok credentials.
- Use Colab Secrets, environment variables, or a local `.env` file excluded by `.gitignore`.
- Rotate any credential that was previously displayed in a notebook or commit.
- Do not place secrets in screenshots, notebook outputs, Streamlit logs, or README examples.
- Pin and periodically review package versions before production use.
- Treat model artifacts as untrusted unless they were created by the approved notebook and stored securely.
- Do not load untrusted `joblib` or pickle files because deserialization can execute malicious code.

## 9. Human Oversight

For coursework, all outputs should be interpreted by the project team. For any use beyond coursework:

- Route low-confidence or conflicting cases to human review.
- Allow reviewers to reject or override the model output.
- Record overrides without retaining unnecessary patient text.
- Regularly examine high-confidence mistakes.
- Suspend the system if safety filters, artifact validation, or monitoring fails.

## 10. Monitoring Recommendations

If the system is maintained or deployed, monitor:

- Accuracy and Macro-F1 over time
- Per-class precision, recall, and F1
- Medium-class error rate
- Performance by medication and condition
- Prediction-confidence distribution
- High-confidence errors
- Retrieval relevance and similarity-score distribution
- Medical-advice refusal success rate
- Application failures and missing-artifact errors
- Changes in input length, language, and data distribution

## 11. Residual Risk

The safeguards reduce risk but do not eliminate it. The system can still produce incorrect predictions, misleading evidence terms, or irrelevant retrieval results. Public reviews cannot represent every patient or clinical situation. Therefore, the system remains an educational research prototype and must not be used for diagnosis, prescribing, treatment selection, or other medical decisions.

## 12. Responsible-Use Checklist

Before sharing or demonstrating the project, confirm that:

- [ ] The dataset is not being redistributed without permission.
- [ ] No credentials or private tokens appear in the repository or notebook outputs.
- [ ] Demonstration inputs are fictional or de-identified.
- [ ] The research-use disclaimer is visible.
- [ ] Medical-advice prompts are refused.
- [ ] Evaluation metrics are labeled with the correct 15,000-review evaluation sample.
- [ ] The 90,000 training sample and 35,000 retrieval library are described accurately.
- [ ] The README setup and usage instructions match the uploaded filenames.
- [ ] The application screenshot contains no private information.
- [ ] Limitations are discussed during the presentation.

