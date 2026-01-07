# 📊 Table 1: Key differences between Bulk RNA-seq and Single-cell RNA-seq

| Aspect                                  | Bulk RNA-seq                                                     | Single-cell RNA-seq (scRNA-seq)                  |
| --------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------ |
| **Goal**                                | Measure **average gene expression** across all cells in a sample | Measure **gene expression of individual cells**  |
|                                         | Compare conditions (e.g. tumor vs normal)                        | Identify **cell types, states, trajectories**    |
| **Biological resolution**               | Population-level signal                                          | Cell-level heterogeneity                         |
| **Protocol**                            | RNA extracted from **all cells together**                        | RNA extracted from **isolated single cells**     |
|                                         | Reverse transcription → cDNA                                     | Cell barcodes + UMIs                             |
|                                         | PCR amplification                                                | High amplification + molecular indexing          |
| **Unique molecular identifiers (UMIs)** | Usually **not used**                                             | **Core component**                               |
| **Spike-ins**                           | Optional (e.g. ERCCs)                                            | Often used to assess technical noise             |
| **Main technical noise**                | Library size, GC bias, batch effects                             | Dropouts, capture efficiency, amplification bias |
| **Quality control metrics**             | GC content, adapters, duplicated reads                           | Genes per cell, reads per cell                   |
|                                         | % reads mapping to genome                                        | % mitochondrial reads                            |
|                                         | Replicate reproducibility                                        | Cell-level QC + bulk-style QC                    |
| **Normalization focus**                 | Between-sample depth and gene length                             | Between-cell depth + dropout correction          |
| **Common normalization methods**        | RPKM, FPKM, TPM                                                  | Library size scaling, log-normalization          |
| **Batch correction**                    | Sample-level batch effects                                       | Cell-level + sample-level batch effects          |
| **Downstream analyses**                 | Differential expression                                          | Dimensionality reduction                         |
|                                         | Alternative splicing                                             | Cell clustering                                  |
|                                         | Gene / transcript quantification                                 | Differential expression between cell types       |
|                                         |                                                                  | Trajectory / pseudotime inference                |


🧠 Table 2: Explanation of “hard” concepts (with trusted references)

| Concept                                 | What it actually means (simple)                      | Why it matters                       | Trusted references                    |
| --------------------------------------- | ---------------------------------------------------- | ------------------------------------ | ------------------------------------- |
| **Average gene expression**             | Bulk RNA-seq collapses all cells into **one signal** | Loses cell-type-specific biology     | Nature Methods (2010), Genome Biology |
| **Cell heterogeneity**                  | Different cells express different genes              | Key in cancer, immunity, development | Cell (2017), Nature Reviews Genetics  |
| **UMIs**                                | Barcodes that tag **original RNA molecules**         | Fix PCR amplification bias           | Nature Methods (2014), Islam et al.   |
| **Dropout events**                      | Gene is expressed but **not detected** in a cell     | Creates false zeros in scRNA-seq     | Nature Biotechnology (2016)           |
| **Capture efficiency**                  | Fraction of RNA captured per cell                    | Drives technical variability         | Nature Methods (2018)                 |
| **Spike-ins**                           | External RNA added at known amounts                  | Estimate technical noise             | Genome Biology (2014)                 |
| **Mitochondrial reads %**               | Fraction of reads mapping to mitochondrial genes     | High % = stressed or dying cells     | Nature Methods (2018)                 |
| **Batch effects**                       | Non-biological variation from experiments            | Can dominate PCA/UMAP                | Bioinformatics (2010), Nature         |
| **Library size effects**                | Different sequencing depths across samples/cells     | Biases expression estimates          | Robinson & Oshlack, Genome Biology    |
| **RPKM / FPKM / TPM**                   | Normalize for gene length + depth                    | Comparable expression within samples | Nature Protocols (2012)               |
| **Dimensionality reduction**            | Compress thousands of genes into few axes            | Enables visualization and clustering | Nature Methods (2008)                 |
| **Clustering**                          | Group cells by expression similarity                 | Identifies cell types/states         | Cell (2015), Nature Methods           |
| **Pseudotime analysis**                 | Order cells along biological progression             | Models differentiation trajectories  | Nature Biotechnology (2014)           |
| **Differential expression (scRNA-seq)** | Compare expression **between cell groups**           | Cell-type-specific DEGs              | Genome Biology (2017)                 |



