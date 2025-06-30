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

## Support
Please direct any problems or questions to Leqi Xu ([leqi.xu@yale.edu](mailto:leqi.xu@yale.edu)).
