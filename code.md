---
title: "COVID-19 Hospitalization in HostSeq"
subtitle: "Code documentation for analysis of n=4873 samples"
author: "Elika Garg"
date: 'May 2023'
output: 
  html_document: 
    keep_md: yes
    theme: paper
    toc: yes
    toc_depth: 3
---



# Regenie

## Input genotypes


```bash
plink2 
--pfile n4873 vzs # final plink file after QC with 4873 samples 
--extract infinium-gsa-24-v3-0-a1-b151-rsids.txt # subset to Illumina GSA variants
--maf 0.01 # filter minor-allele frequency >1%
--mac 100 # filter minor-allele count >100
--not-chr Y,MT # exclude chromosomes Y and MT
--max-alleles 2 # conform to bi-allelic format
--make-pgen psam-cols=fid,sex # create plink file with required columns
--out input_step1 # genotype file for step-1 in regenie

plink2 
--pfile n4873 vzs # final plink file after QC with 4873 samples 
--not-chr Y,MT # exclude chromosomes Y and MT
--max-alleles 2 # conform to bi-allelic format
--make-pgen psam-cols=fid,sex # create plink file with required columns
--out input_step2 # genotype file for step-2 in regenie
```

## Step-1


```bash
regenie 
--step 1 # run step-1 of regenie
--pgen input_step1 # use genotype file created for step-1  
--covarFile pheno # covariate file
--phenoFile pheno # phenotype file 
--phenoCol hospitalization # phenotype variable name 
--covarColList age,agexsex,age2,age2xsex,PC1,PC2,PC3,PC4,PC5,PC6,PC7 # covariate variable names 
--catCovarList sex # categorical covariate
--minCaseCount 10 # filter minimum case counts >10 
--bsize 1000 # block size for regenie 
--bt # specify phenotype is binary  
--test additive # specify model is additive
--par-region hg38 # split-PAR region on chrX using hg38 coordinates
--out output_step1 # results from step-1 in regenie
```

## Step-2

### GWAS


```bash
regenie 
--step 2 # run step-2 of regenie
--pgen input_step2 # use genotype file created for step-2  
--pred output_step1_pred.list # use prediction file created in step-1
--covarFile pheno # covariate file
--phenoFile pheno # phenotype file 
--phenoCol hospitalization # phenotype variable name 
--covarColList age,agexsex,age2,age2xsex,PC1,PC2,PC3,PC4,PC5,PC6,PC7 # covariate variable names 
--catCovarList sex # categorical covariate
--minCaseCount 10 # filter minimum case counts >10 
--minMAC 5 # filter minimum minor-allele count >5 
--bsize 1000 # block size for regenie 
--bt # specify phenotype is binary  
--test additive # specify model is additive
--firth --approx --pThresh 0.05 # use Firth-approx as fallback for imbalance case-control variants
--par-region hg38 # split-PAR region on chrX using hg38 coordinates
--out output_gwas # results from step-2 in regenie
```

### GxSex interaction 2df test


```bash
regenie 
--step 2 # run step-2 of regenie
--pgen input_step2 # use genotype file created for step-2  
--pred output_step1_pred.list # use prediction file created in step-1
--covarFile pheno # covariate file
--phenoFile pheno # phenotype file 
--phenoCol hospitalization # phenotype variable name 
--covarColList age,agexsex,age2,age2xsex,PC1,PC2,PC3,PC4,PC5,PC6,PC7 # covariate variable names 
--catCovarList sex # categorical covariate
--minCaseCount 10 # filter minimum case counts >10 
--minMAC 5 # filter minimum minor-allele count >5 
--bsize 1000 # block size for regenie 
--bt # specify phenotype is binary  
--test additive # specify model is additive
--interaction sex # specify sex is E in GxE
--firth --approx --pThresh 0.05 # use Firth-approx as fallback for imbalance case-control variants
--par-region hg38 # split-PAR region on chrX using hg38 coordinates
--out output_gxsex # results from step-2 in regenie
```

### SKAT-O


```bash
regenie 
--step 2 # run step-2 of regenie
--pgen input_step2 # use genotype file created for step-2  
--pred output_step1_pred.list # use prediction file created in step-1
--covarFile pheno # covariate file
--phenoFile pheno # phenotype file 
--phenoCol hospitalization # phenotype variable name 
--covarColList age,agexsex,age2,age2xsex,PC1,PC2,PC3,PC4,PC5,PC6,PC7 # covariate variable names 
--catCovarList sex # categorical covariate
--bsize 1000 # block size for regenie 
--bt # specify phenotype is binary  
--anno-file impact # variant annotation file
--set-list set # variant set file
--mask-def mask # mask definition file
--vc-tests skato # specify test is skato
--firth --approx --pThresh 0.05 # use Firth-approx as fallback for imbalance case-control variants
--par-region hg38 # split-PAR region on chrX using hg38 coordinates
--out output_skato # results from step-2 in regenie
```

# PRSice

## Input genotypes


```bash
plink2 
--pfile n4873 vzs # final plink file after QC with 4873 samples 
--autosome # only autosomes
--max-alleles 2 # conform to bi-allelic format
--make-bed # create plink bed file
--out input_prsice # genotype file for prsice
```

## PRS


```bash
Rscript PRSice2/PRSice.R
--prsice PRSice2/PRSice_linux # prsice executable file
--target input_prsice # use genotype file created for prsice 
--pheno pheno # phenotype file 
--pheno-col hospitalization # phenotype variable name 
--cov pheno # covariate file
--cov-col age,agexsex,age2,age2xsex,PC1,PC2,PC3,PC4,PC5,PC6,PC7 # covariate variable names 
--cov-factor sex # categorical covariate
--base HostSeq_metasubtracted_COVID19_HGI_B1_ALL_leave_23andme_and_BQC19_20220403.tsv.gz # GWAS file 
--a1 ALT # column name for alt allele in GWAS file
--a2 REF # column name for ref allele in GWAS file
--bp POS # column name for position in GWAS file
--pvalue P # column name for p-value in GWAS file
--stat BETA # column name for effect size in GWAS file
--chr-id c:l-aB # variant ID creation style
--beta # specify effect size is in form of beta
--binary-target T # specify phenotype is binary  
--model add # specify model is additive  
--clump-kb 750 # clumping window size in kb
--clump-p 1 # clumping p-value threshold
--clump-r2 0.1 # clumping r2 threshold
--bar-levels 5e-8,1e-5,1e-4,1e-3,1e-2,5e-2,1e-1,5e-1,1 # list of p-value threholds
--fastscore # only calculate prs at listed p-value thresholds
--all-score # write prs for all listed p-value thresholds
--score std # specify effect size will be standardized
--out output_prs # results from prsice
```

