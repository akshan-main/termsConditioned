# TermsConditioned: Calibrated Clause Triage and Slice Audit

DATA 512 – Human Centered Data Science  
Final Project – A7: Final Project Report  

---

## Abstract

This project investigates how a simple, calibrated clause-family classifier can be used as a triage assistant for contract-style text. Using the LEDGAR subset of the LexGLUE benchmark, a RoBERTa-large encoder is fine-tuned (with LoRA adapters) to predict clause families for individual contract paragraphs. The model’s logits are calibrated with temperature scaling, which reduces expected calibration error and makes output probabilities interpretable as risk-style scores.

A fixed set of “attention-worthy” clause families (for example Waivers, Remedies, Indemnity, Governing Laws, Jurisdictions, Amendments) is defined, and a scalar attention score is constructed by summing calibrated probabilities over this bucket. A simple policy sweep over confidence thresholds selects a triage rule that keeps roughly 99.5 percent of clauses automated, while capping the rate of “attention-worthy but treated as low attention” errors.

Slice tables over clause families, length and density buckets, and regex phrase flags (such as “sole discretion” or “governing law”) reveal where false reassurance is concentrated. Waivers, Remedies, and long, dense paragraphs drive most of the harm score, suggesting that these slices need additional safeguards in any deployment. A lightweight triage API produces paragraph-level review cards and a ranked queue for real Terms of Service text, demonstrating how calibrated scores and slice audits can support human-centered review of contract clauses.

---

## Repository contents

This folder (for the A7 final project) is expected to contain:

- `AkshanKrithick_DATA512_project_termsConditioned.ipynb`  
  Main Jupyter notebook containing the full written report and all analysis code.

- `AkshanKrithick_DATA512_project_termsConditioned.ipynb.pdf`  
  PDF export of the main notebook.

- `data/ledgar/train/`  
  Training split used for demonstration and reproducibility.

- `data/ledgar/validation/`  
  Validation split used for calibration, threshold selection, and slice analysis.

- `data/ledgar/test/`  
  Test split (left unused in the main analysis, provided for completeness).

- `termsconditioned_export/`  
  Output artifacts produced by the notebook, including:
  - `policy_sweep.csv`
  - `family_table.csv`
  - `bucket_table.csv`
  - `flag_table.csv`
  - `termsconditioned_meta.json`
  - ROC, PR, and reliability CSVs
  - PNG plots for ROC, PR, and reliability diagrams

- `audit_pack.md`  
  A standalone validation audit report summarizing global metrics, calibration, the chosen operating point, and the worst slices by harm score.

- `README.md`  
  This file.


---

## Data sources and licenses

### LEDGAR / LexGLUE

- **Dataset:** LEDGAR subset of LexGLUE (legal clause classification benchmark)  
- **Source:** Hugging Face datasets hub  
  - LexGLUE dataset card (including LEDGAR):  
    https://huggingface.co/datasets/coastalcph/lex_glue  
- **Original papers:**
  - Tuggener et al., “LEDGAR: A Large-Scale Multi-label Corpus for Text Classification of Legal Provisions in Contracts”, LREC 2020.
  - Chalkidis et al., “LexGLUE: A Benchmark Dataset for Legal Language Understanding in English”, 2021 (arXiv:2110.00976).

According to the dataset card on Hugging Face, LexGLUE (including LEDGAR) is distributed under the **CC BY 4.0** license. The data consists of contract provisions drawn from public SEC EDGAR filings. The project follows the dataset license, cites the original sources, and does not attempt to deanonymize or link individual paragraphs back to specific parties.

All experiments in the notebook are run on LEDGAR-style contract paragraphs. LEDGAR is used as a stand-in for Terms-and-Conditions-like legal prose; no scraping of live Terms of Service is performed for the quantitative analysis.

### Terms of Service example

A short qualitative demo is run on a single public Terms of Service style document to show how the triage assistant behaves on a realistic policy:

- TickTick Terms of Service:  
  https://ticktick.com/tos?language=en_us  

The TickTick text is used only as a copied, static example in the notebook for demonstration of the triage API. No automated scraping is performed, and no claims are made about performance on the broader space of consumer policies.

---

## External resources and model links

- **Hugging Face model (classifier checkpoint):**  
  RoBERTa-large with LoRA adapters fine-tuned on LEDGAR clause families  
  https://huggingface.co/akshan-main/termsconditioned-roberta-large-ledgar-lora

- **Hugging Face datasets documentation:**  
  https://huggingface.co/docs/datasets  
- **Transformers library documentation:**  
  https://huggingface.co/docs/transformers  
- **PyTorch documentation:**  
  https://pytorch.org/docs/stable/  

Additional related work cited in the notebook:

- Lippi et al., “CLAUDETTE: an Automated Detector of Potentially Unfair Clauses in Online Terms of Service”, 2018 (arXiv:1805.01217).

---

## Reproducing the analysis

High-level steps to reproduce the full analysis:

1. **Environment setup**
   - Use Google Colab or a local environment with a GPU (T4 or better recommended).
   - Open `AkshanKrithick_DATA512_project_termsConditioned.ipynb`.
   - Ensure internet access is available so the notebook can:
     - Install required Python packages (PyTorch, Transformers, PEFT, bitsandbytes, datasets, evaluate, scikit-learn, pandas, pyarrow).
     - Download the LEDGAR split from `coastalcph/lex_glue`.
     - Download the published classifier from Hugging Face.

2. **Authentication**
   - Run the Hugging Face `login()` cell and paste a personal access token with read access (write access is only needed if pushing models back to the hub).

3. **Data**
   - The notebook loads LEDGAR directly via `datasets.load_dataset`.  
   - The `data/` directory in this repository contains train/validation/test sets, it is an alternate way to load the ledgar data.

4. **Execution order**
   - Run the notebook cells from top to bottom:
     - Setup and imports
     - Dataset loading and slice feature construction
     - Classifier loading (from Hugging Face)
     - Calibration and policy sweep
     - Binary risk evaluation and plots
     - Slice table construction and export
     - Triage API and ToS demo
     - Audit pack export and findings sections

5. **Outputs**
   - All analysis outputs (CSV tables, PNG figures, and the markdown audit pack) are written to `termsconditioned_export/`.  
   - These artifacts can be inspected directly from the repository or re-generated by re-running the notebook.

This repository is intended to be self-contained for the purposes of the DATA 512 final project: it includes the main notebook, a PDF export, representative input data, exported analysis artifacts, and links to all external datasets, models, and legal documents used in the study.