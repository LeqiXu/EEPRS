# EEPRS
EEPRS is a phenotype embedding-based PRS interpretation and integration framework designed to effectively combine high-dimensional EHR data with traditional PRS.

The EEPRS pipeline involves five main steps (Figure 1):

* **Step 1: Generate EHR-based phenotype embeddings**  
  We construct low-dimensional embeddings from longitudinal EHR data using embedding models, effectively capturing clinical co-occurrence patterns and latent associations.

* **Step 2: Perform EHR embedding-based GWAS**  
  We conduct GWAS on each embedding dimension as a quantitative trait, capturing subtle genetic associations beyond traditional GWAS approaches by leveraging rich multidimensional phenotypic variations.

* **Step 3: Derive EHR embedding-based PRS**  
  We build embedding-based PRS using embedding GWAS summary statistics, generating individual-level genetic predisposition profiles reflective of embedding dimensions.

* **Step 4: Interpret EHR embedding-based PRS in a PRS-based PheWAS framework**  
  We systematically evaluate associations between embedding-derived PRS and ICD-10 phenotypes to elucidate the clinical significance and biological relevance of genetic risks captured by embeddings.

* **Step 5: Integrate EHR embedding-informed PRS via EEPRS-Integrator in the EEPRS framework**  
  We combine embedding-informed PRS and conventional trait-specific PRS via EEPRS-Integrator, using optimized weights to enhance prediction accuracy without the need for individual-level tuning data.

<p align="center">
  <img src="https://github.com/user-attachments/files/20981292/Figure1.pdf" alt="EEPRS Workflow"/>
</p>

For full details of codes, see the [EEPRS_analysis repository](https://github.com/LeqiXu/EEPRS_analysis).

## Getting Started
In this section, we provide detailed, step-by-step instructions for implementing EEPRS. Please replace all placeholders with the appropriate paths and filenames specific to your computing environment.

### Step 1: Generate EHR-based phenotype embeddings
This step involves constructing phenotype embeddings from EHR data using ICD-10 codes. The procedure includes:

* **Data preprocessing:** Extract individual longitudinal ICD-10 code sequences from EHR data, ordered by hospital admission dates. Treat these sequences as "sentences," with ICD-10 codes as tokens.

* **ICD-10 code embedding:** Generate dense vector representations for ICD-10 codes using two approaches:

  * **Word2Vec-based:** Train embeddings from clinical descriptions using the CBOW Word2Vec model.
  * **GPT-based:** Use GPT-generated plain-language descriptions, embedding them with GPT models.

* **Individual embedding:** Compute individual-level embeddings by averaging ICD-10 code embeddings across each person's clinical record.

* **Dimension reduction:** Apply PCA or ICA dimensionality reduction techniques to individual embeddings, retaining components explaining at least 80% of the variance, creating compact representations (Word2Vec\_PCA, Word2Vec\_ICA, GPT\_PCA, GPT\_ICA).

Detailed code implementation is available in [Phenotype embedding] and [Dimension reduction](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/1.2%20Embedding_GWAS).

### Step 2: Perform EHR embedding-based GWAS
This step involves applying quantile normalization separately to each embedding dimension to approximate normality. Each normalized embedding dimension is then analyzed individually as a quantitative phenotype in marginal linear regression models across all HapMap3 SNPs (S = 1,297,431). Covariates such as age, sex, and the top 20 genetic PCs are included to control for confounding effects. GWAS analyses are performed using [PLINK2](https://www.cog-genomics.org/plink/2.0/).

Detailed code implementation is available in [Embedding_GWAS](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/1.2%20Embedding_GWAS).

Calculated GWAS using UK Biobank training data (N = 207,734) is available in [We will publicly release our data upon publication].

### Step 3: Derive EHR embedding-based PRS
This step computes PRS using EHR embedding-based GWAS summary statistics generated from Step 2. We recommend using PRS methods that require only GWAS summary statistics and LD reference panels for convenience. Detailed implementations of these component methods are available in their respective repositories:

* [PRS-CS-auto](https://github.com/getian107/PRScs)
* [SDPR](https://github.com/eldronzhou/SDPR)
* [SBayesRC](https://github.com/zhilizheng/SBayesRC)

### Step 4: Interpret EHR embedding-based PRS in a PRS-based PheWAS framework
This step interprets the EHR embedding-based PRS using a PRS-based PheWAS framework, assessing associations between each embedding-based PRS (predictor) and ICD-10 code-derived phenotypes (outcomes) using the [PheWAS R package](https://github.com/PheWAS/PheWAS). For each PRS–phenotype pair, we perform logistic regression adjusted for age, sex, and the top 20 genetic PCs to account for population stratification and potential confounders. To correct for multiple testing, we apply the BH procedure to control the FDR.

Detailed code implementation is available in [PheWAS_analyze](https://github.com/LeqiXu/EEPRS_analysis/tree/main/5.%20PheWAS_analyze).

Instructions and phecode data can be found at [Phecode resource](https://wei-lab.app.vumc.org/phecode)

### Step 5: Integrate EHR embedding-informed PRS via EEPRS-Integrator in the EEPRS framework
This step integrates EHR embedding-based PRS with trait-specific PRS via EEPRS-Integrator that requires only GWAS summary statistics. The EEPRS-Integrator pipeline involves four main steps (Figure 2):

* **Step 1: EHR embedding selection**
  We identify embedding GWAS results that are genetically correlated with the target trait, as assessed using [LDSC](https://github.com/bulik/ldsc). Only embeddings showing statistically significant genetic correlation are selected for subsequent integration.

* **Step 2: Target trait GWAS subsampling**
  We generate statistically independent training and tuning GWAS using the Step1 GWAS subsampling in [MIXPRS](https://github.com/LeqiXu/MIXPRS) based on the target trait GWAS summary statistics.

* **Step 3: Estimating PRS combination weights**
  This step consists of two sub-steps:

  * **Step 3.1: PRS coefficient estimation:**
    PRS-CS-auto is applied independently to the subsampled training GWAS summary statistics for the target trait and each selected embedding GWAS (after additional LD pruning), generating LD-pruned PRS beta coefficients.

  * **Step 3.2: Combination weight determination:**
    In contrast to the non-negative least squares approach used in MIXPRS, we employ linear regression to estimate optimal combination weights. This allows for negative weights, accommodating embeddings that may be negatively associated with the target trait. Only embeddings selected in Step 1 are included to ensure robustness. Final weights are estimated using the subsampled tuning GWAS summary statistics and the calculated LD-pruned PRS beta coefficients.

* **Step 4: Derivation of EEPRS**
  Complete GWAS summary statistics for the target trait and selected embeddings are re-analyzed using PRS-CS-auto to generate final PRS beta coefficients. These beta coefficients are then integrated using the weights from Step 3 to derive the final EEPRS. This integrated score captures complementary genetic signals, enhancing predictive accuracy across a range of complex traits.

<p align="center">
  <img src="https://github.com/user-attachments/files/20983094/FigureS1.pdf" alt="EEPRS-Integrator"/>
</p>

[MIXPRS](https://github.com/LeqiXu/MIXPRS)

## Support
Please direct any problems or questions to Leqi Xu ([leqi.xu@yale.edu](mailto:leqi.xu@yale.edu)).
