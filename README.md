# 🧬 Genotype QC & Imputation Pipeline

---

# 📦 STEP 1 — Pre-imputation QC

## 📁 Folder structure (DO NOT MODIFY)

```bash
step1_preimput_QC/
├── A_input_geno_rawdata/
│   ├── {BATCH_NAME}_geno_rawdata/
│   │   ├── {BATCH_NAME}_geno_rawdata.map
│   │   ├── {BATCH_NAME}_geno_rawdata.ped
│   │   └── {BATCH_NAME}_pheno_sex.txt (optional)
│
├── scripts/
│   ├── QC0_Target_data_phenotype.R
│   ├── QC1_Hist_miss.R
│   ├── QC2_Gender_check.R
│   ├── QC3_MAF_check.R
│   ├── QC4_HWE.R
│   ├── QC5_Check_heterozygosity_rate.R
│   └── QC6_Relatedness.R
│
├── info_preimput_QC/
├── output_preimput_QC/
└── STEP1_QC_PREIMPUT.sh
```

---

## 📥 Input Preparation

### 📂 Create a new batch

```bash
step1_preimput_QC/A_input_geno_rawdata/{BATCH_NAME}_geno_rawdata/
```

The pipeline automatically detects batches using this naming convention.

---

### 🧬 Genotype files

```bash
{BATCH_NAME}_geno_rawdata.ped
{BATCH_NAME}_geno_rawdata.map
```

⚠️ File names **must match the folder name**.

---

### 🧾 Phenotype information (`.fam`)

Required format:

```
FID | IID | PHENO
```

| Value | Meaning |
| ----- | ------- |
| 1     | Control |
| 2     | Case    |
| -9    | Missing |

---

### ❗ If phenotype format is incorrect

1. Provide external file:

```
Target_data_pheno_*.txt
```

2. Modify:

```
QC0_Target_data_phenotype.R
```

---

### 🚻 Sex information (optional)

If missing in `.fam`:

```
{BATCH_NAME}_pheno_sex.txt
```

Format (no header):

```
FID | IID | SEX
```

---

## ▶️ Run STEP1

```bash
bash STEP1_QC_PREIMPUT.sh
```

---

## 📤 Output

Directory:

```
output_preimput_QC/{BATCH_NAME}_output_preimput_QC/
```

Files:

```
{BATCH_NAME}_preimput_QC.bed
{BATCH_NAME}_preimput_QC.bim
{BATCH_NAME}_preimput_QC.fam
```

---

## 📄 Logs & documentation

```
info_preimput_QC/{BATCH_NAME}_info_preimput_QC/
```

Includes:

* QC reports
* `.log` files
* pipeline documentation

---

## ✅ Notes

* Related individuals → **identified but not removed**
* Heterozygosity outliers → **removed**

---

# 🧬 STEP 2 — Pre-imputation Data Processing (TopMED)

## 📁 Folder structure (DO NOT MODIFY)

```bash
step2_preimput_dataprocessing/
├── A_input_preimput_processing/
│   ├── {BATCH_NAME}_preimput_processing/
│   │   ├── {BATCH_NAME}_preimput_QC.bed
│   │   ├── {BATCH_NAME}_preimput_QC.bim
│   │   └── {BATCH_NAME}_preimput_QC.fam
│
├── scripts/
│   ├── PRE0_FID_IID.R
│   └── PRE0D_Duplicates.R
│
├── info_preimput_processing/
├── output_preimput_processing/
└── STEP2_PREIMPUT_PROCESSING.sh
```

---

## 📥 Input

Copy STEP1 output into:

```
A_input_preimput_processing/{BATCH_NAME}_preimput_processing/
```

⚠️ Only the folder name must follow the convention.

---

## ⚙️ Configurable parameters

| Variable         | Description          |
| ---------------- | -------------------- |
| runplink         | Path to PLINK        |
| s0liftoverTarget | LiftOver target data |
| mywindowsize     | LD pruning window    |
| mypairwiser2     | LD threshold         |

---

## ▶️ Run STEP2

```bash
bash STEP2_PREIMPUT_PROCESSING.sh
```

---

## 📤 Output

```
QCtargetdata-updated-chr{1..22}.vcf.gz
QCtargetdata-updated-chr{1..22}.vcf.gz.tbi
```

Ready for:

* TOPMed
* Michigan Imputation Server

---

## 🌐 External resources

Downloaded automatically:

* HRC reference panel
* HRC checking tools (Will Rayner)

---

## ⚠️ LiftOver (optional)

Enable if not GRCh37:

```bash
s0liftoverTarget=YES
```

---

# 📊 STEP 2B — PCA & Ancestry

## 📁 Folder structure (DO NOT MODIFY)

```bash
step2B_batchPCs/
├── A_input_batchPCs/
│   └── 1000G_reference_data/
│
├── scripts/
│   ├── PRE0_FID_IID.R
│   ├── PRE0D_Duplicates.R
│   ├── PRE3_PCA.R
│   └── PRE3_PCA_SAFE.R
│
├── info_batchPCs/
├── output_batchPCs/
└── STEP2B_BATCH_PCS.sh
```

---

## 📥 Input

* STEP1 QCed data
* 1000G reference dataset

---

## ▶️ Run STEP2B

```bash
bash STEP2B_BATCH_PCS.sh
```

---

## 📤 Output

```
4_PCA_results.covariate
4_PCA_results.eigenvec
4_PCA_results.eigenval
```

---

## ✅ Results

* PCA plots (ancestry)
* Covariate file (10 PCs)

⚠️ Outliers are **identified but not removed**

---

# 🧪 STEP 3 — Post-imputation Data Processing

## 📁 Folder structure (DO NOT MODIFY)

```bash
step3_postimput_dataprocessing/
├── A_input_postimput_processing/
│   ├── {BATCH_NAME}_input_postimput_processing/
│   │   ├── chr_1.zip
│   │   ├── ...
│   │   └── chr_22.zip
│
├── scripts/
│   ├── POS4_MAF_check.R
│   └── POS5_Hist_miss.R
│
├── info_postimput_processing/
├── output_postimput_processing/
└── STEP3_POSTIMPUT_PROCESSING.sh
```

---

## 📥 Input options

### Option 1 — Local files

```
chr_1.zip ... chr_22.zip
```

---

### Option 2 — Download from server

* Requires internet
* Uses `curl` + `7z`

---

## 🔐 Decompression

```bash
module load p7zip
```

---

## ▶️ Run STEP3

```bash
bash STEP3_POSTIMPUT_PROCESSING.sh
```

---

## 📤 Output

```
{BATCH_NAME}_TOPMED_POSTimputed.bed
{BATCH_NAME}_TOPMED_POSTimputed.bim
{BATCH_NAME}_TOPMED_POSTimputed.fam
```

---

## 🔎 QC filters

* Imputation quality: `R² > 0.9`
* MAF ≥ 0.001
* Missingness (two-step filtering)
* Removal:

  * Palindromic SNPs
  * Indels
  * Duplicate variants

---

## 📄 Logs

```
info_postimput_processing/{BATCH_NAME}_info_postimput_processing/
```

---

## ✅ Final result

Fully QCed dataset ready for:

* PRS analysis
* GWAS
* downstream analyses

---

## ⚠️ Notes

* Thresholds should be adjusted depending on study design
* TOPMed output → GRCh38 (may require liftover)
* Michigan output → GRCh37 (no liftover needed)

---

# ☁️ Imputation server parameters (recommended)

| Parameter       | Value      |
| --------------- | ---------- |
| Reference panel | HRC r1.1   |
| Build           | GRCh37     |
| rsq filter      | 0.3        |
| Phasing         | Eagle v2.4 |
| Population      | EUR        |
