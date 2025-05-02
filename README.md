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

## 📁 File Structure

project-root/
├── alcohol_use_social_only.csv       # Extracted Social History + labels
├── untitled6.ipynb                   # Main Colab notebook with all models and evaluations
├── checklist_tests/                  # Templates for Negation, Attribution, etc.
└── README.md                         # This file

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
- **Misspellings**: e.g., “Pt d0es n0t dr1nk ETOH.”

**Tools used**:
- `editor.template()` to generate MFT test cases
- `Perturb.add_typos()` to test robustness
- Wrapped model predictions with `PredictorWrapper`
- Ran `.run()`, `.summary()` and `.visual_summary_table()` for evaluation

---

## 📊 Results Summary

| Model            | Accuracy | Macro F1 | Weighted F1 |
|------------------|----------|----------|--------------|
| Random Forest     | 29%      | 0.23     | 0.27         |
| XGBoost           | 34%      | 0.20     | 0.29         |
| Bio_ClinicalBERT  | 39%      | 0.27     | 0.37         |

- Bio_ClinicalBERT generalizes better to negation and history-based statements
- Random Forest overfits to majority classes
- Behavioral testing reveals all models struggle with attribution and spelling noise

---

## 🧠 Extensions & Next Steps

1. **Fine-tune Bio_ClinicalBERT with Class Weights or Focal Loss**  
   Address severe class imbalance for low-frequency alcohol use categories

2. **Confidence Calibration**  
   Add reliability diagrams and Expected Calibration Error (ECE) to assess model trustworthiness

3. **Expand to Other SBDH Domains**  
   Extend same framework to detect tobacco or drug use using same MIMIC-SBDH labeling schema

4. **Domain Adaptation**  
   Try PubMedBERT or Longformer for capturing longer discharge summaries

---

## 🧪 Dependencies

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

- Ahsan, H., et al. “MIMIC-SBDH: A Dataset for Social and Behavioral Determinants of Health.” arXiv preprint arXiv:2112.05883 (2021).  
- Ribeiro, M. T., et al. “Beyond Accuracy: Behavioral Testing of NLP Models with CheckList.” ACL (2020).  
- Alsentzer, E., et al. “Publicly Available Clinical BERT Embeddings.” arXiv:1904.03323 (2019).  
- Devlin, J., et al. “BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding.” NAACL (2019).  
