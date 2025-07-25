# EEPRS
EEPRS (Electronic Health Record Embedding Enhanced Polygenic Risk Score) is a phenotype embedding-based PRS interpretation and integration framework designed to effectively combine high-dimensional EHR data with traditional PRS.

This framework extends the multi-population PRS integration method MIXPRS:
- **Xu, L., Dong, Y., Zeng, X., Bian, Z., Zhou, G., Guan, L., & Zhao, H. (2025). Almost Free Enhancement of Multi-Population PRS: From Data-Fission to Pseudo-GWAS Subsampling. bioRxiv, 2025-06.**
  
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
  * **GPT-based:** Use GPT-o1-generated plain-language descriptions, embedding them using the text-embedding-3-large model.

* **Individual embedding:** Compute individual-level embeddings by averaging ICD-10 code embeddings across each person's clinical record.

* **Dimension reduction:** Apply PCA or ICA dimensionality reduction techniques to individual embeddings, retaining components explaining at least 80% of the variance, creating compact representations (Word2Vec\_PCA, Word2Vec\_ICA, GPT\_PCA, GPT\_ICA).

Detailed implementation code is available in [Phenotype embedding](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/1.1%20Embedding_derive) and [Dimension reduction](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/1.2%20Embedding_GWAS).

### Step 2: Perform EHR embedding-based GWAS
This step involves applying quantile normalization separately to each embedding dimension to approximate normality. Each normalized embedding dimension is then analyzed individually as a quantitative phenotype in marginal linear regression models across all HapMap3 SNPs (S = 1,297,431). Covariates such as age, sex, and the top 20 genetic PCs are included to control for confounding effects. GWAS analyses are performed using [PLINK2](https://www.cog-genomics.org/plink/2.0/).

Detailed implementation code is available in [Embedding_GWAS](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/1.2%20Embedding_GWAS).

Calculated GWAS using UK Biobank training data (N = 207,734) is available in [We will publicly release our data upon publication].

### Step 3: Derive EHR embedding-based PRS
This step computes PRS using EHR embedding-based GWAS summary statistics generated from Step 2. We recommend using PRS methods that require only GWAS summary statistics and LD reference panels for convenience. Detailed implementations of these methods are available in their respective repositories:

* [PRS-CS-auto](https://github.com/getian107/PRScs).
* [SDPR](https://github.com/eldronzhou/SDPR).
* [SBayesRC](https://github.com/zhilizheng/SBayesRC).

### Step 4: Interpret EHR embedding-based PRS in a PRS-based PheWAS framework
This step interprets the EHR embedding-based PRS using a PRS-based PheWAS framework, assessing associations between each embedding-based PRS (predictor) and ICD-10 code-derived phenotypes (outcomes) using the [PheWAS R package](https://github.com/PheWAS/PheWAS). For each PRS–phenotype pair, we perform logistic regression adjusted for age, sex, and the top 20 genetic PCs to account for population stratification and potential confounders. To correct for multiple testing, we apply the BH procedure to control the FDR.

Detailed implementation code is available in [PheWAS_analyze](https://github.com/LeqiXu/EEPRS_analysis/tree/main/5.%20PheWAS_analyze).

Instructions and phecode data can be found at [Phecode resource](https://wei-lab.app.vumc.org/phecode).

### Step 5: Integrate EHR embedding-informed PRS via EEPRS-Integrator in the EEPRS framework
This step integrates EHR embedding-based PRS with trait-specific PRS via EEPRS-Integrator that requires only GWAS summary statistics. The EEPRS-Integrator pipeline involves four main steps (Figure 2):

* **Step 1: EHR embedding selection**  
  We select embedding GWASs that are genetically correlated with the target trait, as assessed using [LDSC](https://github.com/bulik/ldsc). Only embeddings showing statistically significant genetic correlation are selected for subsequent integration.

  Detailed implementation code is available in [LDSC corr](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/1.2%20Embedding_GWAS).

* **Step 2: Target trait GWAS subsampling**  
  We generate statistically independent training and tuning GWAS using the Step1 GWAS subsampling in [MIXPRS](https://github.com/LeqiXu/MIXPRS) based on the target trait GWAS summary statistics.

  Detailed implementation code is available in [Trait GWAS subsample](https://github.com/LeqiXu/EEPRS_analysis/tree/main/1.%20Data_prepare/2.2%20Trait_GWAS_subsample).

* **Step 3: Estimating PRS combination weights**  
  This step consists of two sub-steps:

  * **Step 3.1: PRS coefficient estimation:**  
    We calculate the PRS for subsampled training GWAS summary statistics for the target trait and each selected embedding GWAS (after additional LD pruning), generating LD-pruned PRS beta coefficients.

    * **Calculate PRS with subsampled training GWAS for target trait:**
      [Trait subsample PRS](https://github.com/LeqiXu/EEPRS_analysis/blob/main/2.%20Method_calculate/2.%20TraitPRS_calculate/1.2%20Trait.subsample.PRS.beta.sh).
    * **Perform LD pruning for embedding GWAS:**  
      ```bash
      library(data.table)

      pop = "EUR"
      train_type = "train"

      for (i in c(1:100)){
        # PRScsx format GWAS for full snplist
        PRScsx_clean = fread(paste0("/gpfs/gibbs/pi/zhao/lx94/EEPRS/data/embedding_data/PRScsx/word2vec100_",train_type,"_EUR_UKB_Embedding",i,"_PRScsx.txt"))

        # Prune snplist
        prune_snplist = fread(paste0("/gpfs/gibbs/pi/zhao/lx94/SWIFT/data/prune_clump/snplist/",pop,"_prune_pval1_r20.5_wc250_1.snplist"), header = FALSE)

        # PRScsx format GWAS for prune snplist
        PRScsx_clean_prune_snplist = PRScsx_clean[which(PRScsx_clean$SNP %in% prune_snplist$V1),]

        write.table(PRScsx_clean_prune_snplist, file=paste0("/gpfs/gibbs/pi/zhao/lx94/EEPRS/data/embedding_data/PRScsx/word2vec100_",train_type,"_EUR_UKB_Embedding",i,"_prune_",pop,"_PRScsx.txt"), 
                row.names=F, col.names=T, quote=F, append=F, sep = "\t")

      }
      ```
      Here,
      * PRScsx format GWAS is the embedding GWAS with the following format:
      ```
      SNP               A1      A2      BETA            P
      rs3934834	  C	  T	  0.0063086	  0.00512
      rs3766192	  T	  C	  0.00761278	  5.14e-06
      rs9442372	  G	  A	  0.00690567	  2.66e-05
      ...
      ```
      * Prune snplist can be downloaded in [MIXPRS snplist](https://github.com/LeqiXu/MIXPRS/tree/main/snplist).
        
    * **Calculate LD-pruned PRS for embeddings:**
      [Word2Vec LD-pruned PRS](https://github.com/LeqiXu/EEPRS_analysis/blob/main/2.%20Method_calculate/1.%20EEPRS_calculate/2.1%20Word2Vec_prune.sh); [Word2Vec_PCA LD-pruned PRS](https://github.com/LeqiXu/EEPRS_analysis/blob/main/2.%20Method_calculate/1.%20EEPRS_calculate/2.2%20Word2Vec_PCA_prune.sh); [Word2Vec_ICA LD-pruned PRS](https://github.com/LeqiXu/EEPRS_analysis/blob/main/2.%20Method_calculate/1.%20EEPRS_calculate/2.3%20Word2Vec_ICA_prune.sh); [GPT_PCA LD-pruned PRS](https://github.com/LeqiXu/EEPRS_analysis/blob/main/2.%20Method_calculate/1.%20EEPRS_calculate/2.5%20GPT_PCA_prune.sh); [GPT_ICA LD-pruned PRS](https://github.com/LeqiXu/EEPRS_analysis/blob/main/2.%20Method_calculate/1.%20EEPRS_calculate/2.6%20GPT_ICA_prune.sh). 

  * **Step 3.2: Combination weight determination:**
    In contrast to the non-negative least squares approach used in MIXPRS, we employ linear regression to estimate optimal combination weights. This allows for negative weights, accommodating embeddings that may be negatively associated with the target trait. Only embeddings selected in Step 1 are included to ensure robustness. Final weights are estimated using the subsampled tuning GWAS summary statistics and the calculated LD-pruned PRS beta coefficients.

    Detailed implementation code is available in [PRS combine](https://github.com/LeqiXu/EEPRS_analysis/tree/main/2.%20Method_calculate/3.%20PRS_combine).

* **Step 4: Derivation of EEPRS**
  Complete GWAS summary statistics for the target trait and selected embeddings are re-analyzed to generate final PRS beta coefficients. These beta coefficients are then integrated using the weights from Step 3 to derive the final EEPRS. This integrated score captures complementary genetic signals, enhancing predictive accuracy across a range of complex traits.

  Detailed implementation code is available in [PRS combine](https://github.com/LeqiXu/EEPRS_analysis/tree/main/2.%20Method_calculate/3.%20PRS_combine).

<p align="center">
  <img src="https://github.com/user-attachments/files/20983094/FigureS1.pdf" alt="EEPRS-Integrator"/>
</p>

## Support
Please direct any problems or questions to Leqi Xu ([leqi.xu@yale.edu](mailto:leqi.xu@yale.edu)).
