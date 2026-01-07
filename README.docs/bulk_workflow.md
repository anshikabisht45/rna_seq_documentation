# bulk rna seq ka workflow
-
# RNA-seq End-to-End Workflow-

| Step                             | What happens                                                | Typical tools                                | Why this step matters (industry lens)                                         |
| -------------------------------- | ----------------------------------------------------------- | -------------------------------------------- | ----------------------------------------------------------------------------- |
| **Quality Control (QC)**         | Inspect raw FASTQ files for sequencing artifacts            | FASTQC, NGSQC, RNA-SeQC                      | Early detection of technical failure prevents garbage-in–garbage-out analyses |
| **Pre-processing**               | Remove adapters, trim low-quality bases, filter short reads | Trimmomatic, PRINSEQ, Soapnuke               | Improves alignment accuracy and quantification reliability                    |
| **Read Alignment**               | Map reads to a reference genome or transcriptome            | HISAT2, STAR, Bowtie2                        | Ensures biological interpretability and coordinate consistency                |
| **Transcript Assembly**          | Reconstruct transcripts from aligned reads (optional)       | StringTie, Cufflinks, Trinity                | Useful for isoform-level or novel transcript discovery                        |
| **Expression Quantification**    | Count reads/fragments per gene or transcript                | HTSeq-count, featureCounts, Salmon, Kallisto | Produces the numerical matrix used in all downstream analysis                 |
| **Differential Expression (DE)** | Identify genes with statistically significant changes       | DESeq2, edgeR                                | Converts expression matrices into biological conclusions                      |

---

## Table 2: Deep explanations of critical steps (the "why", not just "how")

| Concept                              | Explanation                                                            | Common pitfalls                          | Best-practice mindset                       |
| ------------------------------------ | ---------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------------- |
| **Raw read QC**                      | Evaluates sequencing quality, GC bias, adapter contamination           | Ignoring per-base quality drops          | QC before *any* trimming or alignment       |
| **Adapter trimming**                 | Removes sequencing adapters falsely interpreted as biological sequence | Over-trimming removes true signal        | Trim conservatively, then re-QC             |
| **Reference genome choice**          | Genome build defines coordinates and annotations                       | Mixing hg19 & hg38 across steps          | One project = one genome + matching GTF     |
| **Alignment vs pseudo-alignment**    | Alignment maps reads to genome; pseudo-aligners map to transcripts     | Mixing count types downstream            | Match quantification method to DE tool      |
| **Counting strategy**                | Assign reads to genes or transcripts                                   | Multi-mapping ambiguity                  | Decide gene-level vs transcript-level early |
| **Library size effects**             | Samples have different sequencing depths                               | Comparing raw counts directly            | Always normalize before comparison          |
| **Negative binomial modeling**       | RNA-seq counts are overdispersed                                       | Using t-tests on counts                  | Use count-based statistical models          |
| **Biological vs technical variance** | Variance comes from biology *and* experiments                          | Misinterpreting batch effects as biology | PCA before DE is mandatory                  |

---

## ASCII Flowchart: RNA-seq pipeline (GitHub-friendly)

```
RAW FASTQ FILES
      |
      v
[ Quality Control ]  ----->  FASTQC / RNA-SeQC
      |
      v
[ Pre-processing ]  ----->  Adapter trimming + quality filtering
      |
      v
[ Read Alignment ]  ----->  Reference genome (HISAT2 / STAR)
      |
      v
[ Expression Matrix ] ---->  Gene x Sample count table
      |
      v
[ Normalization ]   ----->  Size-factor / depth correction
      |
      v
[ Differential Expression ] ---->  log2FC + adjusted p-values
      |
      v
BIOLOGICAL INTERPRETATION & DATA DELIVERY
```



