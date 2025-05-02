# 🧠 Alcohol Use Classification from MIMIC-SBDH Clinical Notes

This project reproduces and extends the classification task of alcohol use based on discharge summaries from the [MIMIC-SBDH dataset](https://github.com/hibaahsan/MIMIC-SBDH), which adds Social and Behavioral Determinants of Health (SBDH) labels to the well-known [MIMIC-III](https://physionet.org/content/mimiciii/1.4/) clinical dataset. We test traditional machine learning models and modern transformer-based approaches such as Bio_ClinicalBERT for multiclass classification of alcohol consumption levels.

---

## 📰 Citation

**Paper:**  
Ahsan, H., et al. (2021). [MIMIC-SBDH: A Dataset for Social and Behavioral Determinants of Health](https://proceedings.mlr.press/v149/ahsan21a/ahsan21a.pdf)  
**GitHub Repo:** https://github.com/hibaahsan/MIMIC-SBDH

---

## 📂 Dataset Access

This project requires access to:

### 1. MIMIC-III Clinical Database  
- Apply for access via [PhysioNet Credentialing](https://physionet.org/account/apply/)
- Download from: https://physionet.org/content/mimiciii/1.4/
- Required files:
  - `NOTEEVENTS.csv.gz`
  - `ADMISSIONS.csv.gz`

### 2. MIMIC-SBDH Labels  
- Download CSVs from: https://github.com/hibaahsan/MIMIC-SBDH  
- Required file: `MIMIC-SBDH.csv`

---

## ⚙️ Steps to Reproduce

### ✅ Step 1: Preprocessing

- Use MedSpaCy’s Sectionizer to extract only the “Social History” portion from discharge notes
- Merge `NOTEEVENTS` with `MIMIC-SBDH.csv` by `subject_id` and `hadm_id`
- Keep only rows with valid alcohol labels (0–4)
- Save as `alcohol_use_social_only.csv`

---

### ✅ Step 2: Model Training

- **Random Forest** and **XGBoost** using TF-IDF features from `sklearn`
- **Bio_ClinicalBERT** using HuggingFace’s transformers, trained on tokenized notes
- Evaluate on held-out test set of 1,405 examples

---

### ✅ Step 3: Behavioral Testing with CheckList

We followed [CheckList (Ribeiro et al., 2020)](https://aclanthology.org/2020.acl-main.442.pdf) methodology for NLP behavioral evaluation:

**Capabilities tested**:
- **Negation**: e.g., “Patient does not drink alcohol.”
- **Attribution**: e.g., “Patient drinks. Father does not.”
- **Historical Phrases**: e.g., “Used to drink but quit 3 years ago.”

**Capabilities not tested**:
- **Misspellings**: e.g., “Pt d0es n0t dr1nk ETOH.” We were unable to resolve issues within the Checklist library for the results against misspellings.

**Tools used**:
- `editor.template()` to generate MFT test cases
- `Perturb.add_typos()` to test robustness
- Wrapped model predictions with `PredictorWrapper`
- Ran `.run()`, `.summary()` and `.visual_summary_table()` for evaluation

---

## 📊 Results Summary


Model Accuracy Summary

| Model            | Accuracy | Macro F1 | Weighted F1 |
|------------------|----------|----------|--------------|
| Random Forest     | 29%      | 0.23     | 0.27         |
| XGBoost           | 34%      | 0.20     | 0.29         |
| Bio_ClinicalBERT  | 39%      | 0.27     | 0.37         |


Behavioral Test Results Summary

| Capability          | Test Cases | Random Forest        | XGBoost             | Bio_ClinicalBERT     |
|---------------------|------------|----------------------|---------------------|----------------------|
| Negation            | 8          | ❌ 8 fails (100%)     | ❌ 8 fails (100%)     | ❌ 8 fails (100%)     |
| Historical          | 12         | ✅ 0 fails (0%)       | ✅ 0 fails (0%)       | ✅ 0 fails (0%)       |
| Attribution         | 12         | ⚠️ 6 fails (50%)      | ❌ 12 fails (100%)    | ⚠️ 2 fails (16.7%)    |

---

## 🧠 Combined Discussion Section

### 🎯 Model Accuracy Discussion

Our evaluation across three models—Random Forest, XGBoost, and Bio_ClinicalBERT—reveals clear performance disparities:

- **Bio_ClinicalBERT** outperforms the other models across all metrics: accuracy (39%), macro F1 (0.27), and weighted F1 (0.37). Its ability to capture contextual language enables better generalization to nuanced clinical statements.
- **Random Forest**, while intuitive and easier to train, achieves only 29% accuracy and underperforms on minority classes (reflected in the macro F1 of 0.23), indicating its reliance on majority class heuristics.
- **XGBoost** marginally surpasses Random Forest in accuracy (34%) and weighted F1 (0.29), but falls behind on macro F1 (0.20), suggesting a high variance in per-class performance.
- The **low macro F1** values across all models highlight poor handling of underrepresented classes like “Current Some-Day” and “Unknown,” warranting further exploration of class imbalance mitigation techniques.

Together, these metrics indicate that while transformer-based models provide better holistic performance, all models are susceptible to skewed class distributions and require calibration or augmentation for deployment in sensitive clinical decision tasks.

### 🧪 Behavioral Testing Discussion

Behavioral evaluation with CheckList provided deeper insights into model robustness and reasoning limitations:

- **Negation** remains a critical failure point: all models (100%) failed to correctly handle statements like “Patient does not drink alcohol.” This reveals a shared inability to capture syntactic negation cues that invert meaning.
- **Historical phrases** (e.g., “used to drink but quit”) were correctly handled by all three models (0% failure), likely due to the more formulaic structure and presence of strong temporal indicators.
- **Attribution** (e.g., differentiating patient vs. relative behavior) exposed major weaknesses:
  - **XGBoost** failed 100% of attribution cases, confirming its difficulty with multi-sentence reasoning.
  - **Random Forest** achieved 50% success, better but still inconsistent.
  - **Bio_ClinicalBERT** performed best with only 16.7% failures, likely due to its pretraining on clinical corpora and ability to track entities across clauses.

These behavioral findings affirm that surface-level accuracy metrics mask critical gaps in semantic understanding, and highlight the importance of CheckList for stress-testing clinical NLP models before real-world deployment.

---

## 🚀 Extensions & Next Steps

1. **Fine-tune Bio_ClinicalBERT with Class Weights or Focal Loss**  
   Address severe class imbalance for low-frequency alcohol use categories

2. **Confidence Calibration**  
   Add reliability diagrams and Expected Calibration Error (ECE) to assess model trustworthiness

3. **Expand to Other SBDH Domains**  
   Extend same framework to detect tobacco or drug use using same MIMIC-SBDH labeling schema

4. **Domain Adaptation**  
   Try PubMedBERT or Longformer for capturing longer discharge summaries

---

## 📦 Dependencies

pip install checklist transformers torch scikit-learn pandas medspacy spacy

Versions used:

- transformers==4.30.2
- torch==2.0.1
- scikit-learn==1.2.2
- pandas==1.5.3
- checklist==0.0.11
- medspacy==0.1.1
- spacy==3.5.4

---

## 📚 References

- Ahsan, H., et al. (2021). MIMIC-SBDH: A Dataset for Social and Behavioral Determinants of Health. Proceedings of the 1st Workshop on Trustworthy NLP, PMLR.
https://proceedings.mlr.press/v149/ahsan21a/ahsan21a.pdf 
- Ribeiro, M. T., Wu, T., Guestrin, C., & Singh, S. (2020). Beyond Accuracy: Behavioral Testing of NLP Models with CheckList. Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics (ACL).
https://aclanthology.org/2020.acl-main.442.pdf

