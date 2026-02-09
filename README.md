# DKO (Data-driven KnockOut for identifying high impact genes)


This is a Pytorch implementation of DKO, as described in our manuscript:

Yongjian He, Vered Klein, Orr Levy, Xu-Wen Wang, [Predicting cell-specific gene expression profile and knockout impact through deep learning] (arXiv https://arxiv.org/abs/2510.03359). 

<p align="center">
  <img src="DKO.png" alt="demo" width="600" height="470" style="display: block; margin: 0 auto;">
</p>


The code is tested in python environment environment-history.yml

## Contents

- [Overview](#overview)
- [Data type for DKO](#Data-type-for-DKO)
- [How the use the DKO framework](#How-the-use-the-DKO-framework)

# Overview

Gene expression data is essential for understanding how genes are regulated and interact within biological systems, providing insights into disease pathways and potential therapeutic targets. Gene knockout has proven to be a fundamental technique in molecular biology, allowing the investigation of the function of specific genes in an organism, as well as in specific cell types. However, gene expression patterns are quite heterogeneous in single-cell transcriptional data from a uniform environment, representing different cell states, which produce cell-type and cell-specific gene knockout impacts. A computational method that can predict the single-cell resolution knockout impact is still lacking. Here, we present a data-driven framework for learning the mapping between gene expression profiles derived from gene assemblages, enabling the accurate prediction of perturbed expression profiles following knockout (KO) for any cell, without relying on prior perturbed data. We systematically validated our framework using synthetic data generated from gene regulatory dynamics models, two mouse knockout single-cell datasets, and high-throughput in vitro CRISPRi Perturb-seq data. Our results demonstrate that the framework can accurately predict both expression profiles and KO effects at the single-cell level. Our approach provides a generalizable tool for inferring gene function at single-cell resolution, offering new opportunities to study genetic perturbations in contexts where large-scale experimental screens are infeasible.


# Data type for DKO
## 1. Ptrain.csv: matrix of single-cell data of size N*M, where N is the number of gene and M is the cell size (without header).

|           | cell 1 | cell 2 | cell 3 | 
|-----------|----------|----------|----------|
| gene 1 | 0.45     | 0.35     | 0.76     | 
| gene 2 | 0.25     | 0.1      | 0        | 
| gene 3 | 0        | 0.55     | 0.1        | 
| gene 4 | 0.3      | 0        | 0.14     |


## 2. Virtual KO experiment: thought experiemt was realized by removing each present gene in each sample. This will generated two data type.

- **Ptest.csv**: perturbation matrix of size `N × C` (no header), where `N` is the number of genes (rows) and `C` is the number of perturbed cells / KO events (columns). Each column represents a single-gene KO in a specific cell.





|           | cell 1 | cell 2 | cell 3 | cell 4 | cell 5 | cell 6 | cell 7 | cell 8 | cell 9 |
|-----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|
| gene 1 | 0        | 0.45        | 0.45     | 0        | 0.35     | 0.35     |0         |0.76     | 0.76      | 
| gene 2 | 0.25     | 0           | 0.25     | 0.1      | 0        | 0.1      | 0        |0        | 0        | 
| gene 3 | 0        | 0           | 0        | 0.55     | 0.55     | 0        | 0.1      |0        | 0.1      | 
| gene 4 | 0.3      | 0.3         | 0        | 0        | 0        | 0        | 0.14     |0.14     | 0       | 

- **Recorder**: `C × 2` table with columns `gene_id` and `cell_id`. Each row `[gene_id, cell_id]` records that the gene is knocked out in that cell.

|gene_id           | cell_id |
|-----------|----------|
| 1 | 1     | 
| 2 | 1     | 
| 4 | 1     | 
| 1 | 2     | 
| 2 | 2     | 
| 3 | 2     | 
| 1 | 3     | 
| 3 | 3     | 
| 4 | 3     | 



# How to use the DKO framework
## Step 1: Predict gene expression profile using gene assemblage
Open `Learn_Mapping_and_Predicting_gene_profile.ipynb` and load `Ptrain.csv` as input. The pipeline splits the cells in Ptrain.csv into an 80/20 train–test set (80% training, 20% testing), trains the model on the training set, and outputs predicted gene expression profiles for the held-out 20% test cells.

## Step 2: Single-gene KO impact prediction
Run `Predicting_KO_impact.ipynb` with `Ptrain.csv` as input. The pipeline will:
1. Construct **Ptest** (and **Ztest**) and a **Recorder** matrix from `Ptrain.csv`.
2. Train the model on **Ptrain.csv**.  
3. Apply the trained model to **Ztest** to predict the post-KO profiles.
4. Computing gene KO impact by comparing the profile befor and after KO
5. **Outputs:** **KO impact scores** for each `(gene, cell)` pair.

The output file:
|gene_id   | cell_id | impact score |
|-----------|----------|----------|
| 1 | 1     | 0.044832664|
| 2 | 1     | 0.061767839
| 4 | 1     | 0.039787205
| 1 | 2     | 0.032485329
| 2 | 2     | 0.049242764
| 3 | 2     | 0.046929884
| 1 | 3     | 0.036644731
| 3 | 3     | 0.059429204
| 4 | 3     | 0.017946411

Each row reports the predicted KO impact score of gene_id in cell_id.
By default, **Ptest** and **Recorder** files enumerate perturbations across the entire input matrix (i.e., applying KO perturbations to all genes across all cells). You may also use your own single-cell expression matrix as `Ptrain.csv`. Depending on your needs, you can optionally construct **Ptest** and **Recorder** for a selected subset of genes/cells and run KO prediction only on those perturbations.

## Step 3: Double-gene KO impact prediction
Similarity to Step2, Run `Predicting_double_gene_KO_impact.ipynb` with `Ptrain.csv` as input. The workflow is the same as Step 2, except that each perturbation knocks out two genes. The output is a KO impact score for each (gene1_id, gene2_id, cell_id) triple.
For example：
| gene1_id | gene2_id | cell_id | k_pred   |
|---------:|---------:|--------:|---------:|
| 4        | 16       | 425     | 0.076976 |
| 8        | 45       | 182     | 0.045585 |
| 36       | 84       | 321     | 0.055198 |
| 71       | 88       | 352     | 0.027712 |
| 10       | 63       | 429     | 0.065817 |
| …        | …        | …       | …        |

Each row reports the predicted KO impact score of the gene pair (gene1_id, gene2_id) in cell_id.




