# rna seq read to transcripts how convert ???

To turn RNA-Seq reads into transcripts, you align short reads to a reference genome/transcriptome, then assemble these alignments, often using specialized software like STAR and Cufflinks, to reconstruct full-length 
transcripts and quantify their expression, handling tricky parts like alternative splicing and multi-mapping reads to build a picture of gene activity. 

Here's a breakdown of the typical process:
1. Pre-processing & Alignment
Quality Control: Clean raw reads (trim adapters, remove low-quality bases).
Alignment: Map reads to a reference genome or transcriptome using an aligner like STAR or HISAT2, which identifies where each short read originates from. 
2. Transcript Assembly & Quantification
Gene-Level vs. Transcript-Level:
Gene-Level: Count reads mapping to each gene for simple expression estimates (good for dominant isoforms).
Transcript-Level (Assembly): Use tools like Cufflinks or Trinity (for de novo) to piece together reads, especially those spanning exon junctions (junction reads), to reconstruct full mRNA transcripts, including novel ones from alternative splicing.
Quantification: Estimate expression levels (e.g., FPKM, TPM, counts) by counting reads assigned to each assembled transcript or gene. 
3. Handling Complexities
Alternative Splicing: Assembly algorithms use reads that span exon-exon junctions to identify different transcript variants (isoforms) from the same gene.
Multi-Mapping Reads: Reads matching multiple locations are challenging; tools use statistical methods (like Expectation-Maximization) or cluster them to assign them proportionally


----------------------------


## GTF Annotation File — What It Is and Why It Matters in RNA-seq

## TL;DR
A **GTF file tells your RNA-seq pipeline what each part of the genome *means***  
(exon, gene, transcript, strand, boundaries).

Without a correct GTF:
- reads may align
- but **gene counts will be wrong or missing**

---

## What is a GTF file?

**GTF (Gene Transfer Format)** is a structured text file that annotates a reference genome.

It defines:
- where genes are located
- which regions are exons vs introns
- how transcripts are built from exons
- which strand a gene belongs to

Think of it as:
- **FASTA** = genome letters  
- **GTF** = biological labels on those letters

---

## Why GTF is critical in RNA-seq

### 1. During alignment (STAR / HISAT2 / Subread)
The GTF helps the aligner:
- recognize exon–intron boundaries
- correctly align reads across splice junctions
- improve accuracy for spliced transcripts

Without a GTF:
- alignments are less biologically informed
- splice-aware mapping is weaker

---

### 2. During read counting (featureCounts / HTSeq)
The GTF is **mandatory** for counting.

It tells the counting tool:
- which gene a read belongs to
- which exons define a gene
- how to handle overlapping features

No GTF = reads mapped but **cannot be assigned to genes**.

---

## What’s inside a GTF file (simplified)

Each row describes **one genomic feature**.

| Column | Meaning |
|---|---|
| seqname | Chromosome (e.g. chr1) |
| source | Annotation source |
| feature | gene / transcript / exon |
| start | Start position |
| end | End position |
| score | Usually `.` |
| strand | `+` or `-` |
| frame | Usually `.` |
| attributes | gene_id, transcript_id, gene_name |

Example:

`chr1 Ensembl exon 11869 12227 . + . gene_id "ENSG00000223972"; transcript_id "ENST00000456328"`;


Meaning:
> This genomic region is an exon belonging to a specific transcript of a gene.

---

## GTF vs GFF (quick clarification)

- **GTF**  
  - stricter format  
  - commonly used in RNA-seq pipelines  

- **GFF3**  
  - more flexible  
  - contains similar information  

Most RNA-seq tools **expect GTF**, so stick with it unless required otherwise.

---

## 🚨 Critical rule (do NOT violate this)

### Genome FASTA and GTF must match exactly.

Example of a BAD setup:
- genome: hg38  
- GTF: hg19  

This leads to:
- missing gene counts
- incorrect assignments
- silent errors (the worst kind)

**One project = one genome build + matching GTF.**

---

## Trusted sources for GTF files

Use only well-maintained annotations:

- **GENCODE** (industry gold standard for human & mouse)
- **Ensembl**
- **RefSeq** (more conservative)

GENCODE is most commonly preferred in research and industry.

---

## Analyst mindset (why this matters)

A GTF is not “just a file you download”.

It defines:
- biological interpretation
- counting logic
- downstream statistical validity

Mistakes here propagate into:
- differential expression
- pathway analysis
- ML features

---

## Interview-ready explanation

> *“The GTF defines gene and exon boundaries and is essential for splice-aware alignment and accurate read counting. I always ensure the GTF matches the reference genome build to avoid annotation mismatches.”*

---

## Common failure symptoms caused by GTF issues

- aligned BAM files but zero counts
- many genes missing unexpectedly
- poor reproducibility
- inconsistent results across tools

First thing to check: **GTF–genome compatibility**.

` gtf end `
----------------

# detailed workflow by khusbu

<img width="905" height="948" alt="{28A372AC-C26E-4A19-A489-76324494B70E}" src="https://github.com/user-attachments/assets/2f5c9128-a5f8-441e-a754-ecbc2a566e8b" />


--------

# Why RNA-seq Needs Splice-Aware Aligners

## TL;DR
RNA-seq reads come from **spliced mRNA**, not raw genomic DNA.  
A **splice-aware aligner** can align reads that span **exon–exon junctions**.  
A non-splice-aware aligner cannot.

---

## The biological problem (root cause)

In eukaryotes:
- Genes contain **exons** (kept)
- and **introns** (removed)

During transcription:
DNA → pre-mRNA → (splicing) → mature mRNA

sql
Copy code

RNA-seq sequences **mature mRNA**, where introns are already removed.

So RNA-seq reads often look like:
[ exon 1 ][ exon 2 ]

yaml
Copy code

But in the genome, those exons are **far apart**.

---

## What goes wrong with non-splice-aware aligners

Non-splice-aware aligners assume:
- reads come from **continuous DNA**
- no large gaps allowed

Result:
- reads spanning exon junctions
  - fail to align
  - or align incorrectly
  - or get discarded

This causes:
- loss of signal
- biased gene counts
- false negatives in expression

---

## What splice-aware aligners do differently

Splice-aware aligners:
- allow **large gaps** in alignments
- interpret gaps as **introns**
- use known exon–intron structure (from GTF)
- can also discover **novel splice junctions**

They correctly align:
read = exon1 | exon2
genome = exon1 ---- intron ---- exon2

yaml
Copy code

---

## Role of the GTF file here

The GTF helps splice-aware aligners:
- know where exons start/end
- speed up junction detection
- improve alignment accuracy

Important:
- aligners can still find novel junctions
- but GTF makes them **biologically informed**

---

## Common splice-aware RNA-seq aligners

| Aligner | Splice-aware | Notes |
|---|---|---|
| STAR | Yes | Very fast, widely used |
| HISAT2 | Yes | Memory-efficient |
| Subread | Yes | Strong soft-clipping + counting |
| Bowtie2 | ❌ No | Not suitable alone |
| BWA | ❌ No | DNA-seq, not RNA-seq |

---

## Why this matters for gene expression analysis

If junction-spanning reads are lost:
- exon coverage drops
- gene counts are underestimated
- differential expression becomes unreliable

Splice-aware alignment ensures:
- accurate gene-level quantification
- correct isoform detection
- biologically valid results

---

## Analyst / interview explanation (clean & short)

> *“RNA-seq reads originate from spliced mRNA, so many reads span exon–exon junctions that are discontinuous in the genome. Splice-aware aligners handle these gaps correctly, which is essential for accurate alignment and gene quantification.”*

---

## One-line mental model (remember this)

**RNA-seq ≠ DNA-seq**  
If the aligner can’t jump introns, it can’t read RNA.

---

## Common beginner mistake

Using:
- Bowtie2 / BWA alone
- without splice awareness
- for RNA-seq

Result:
> “My reads aligned poorly and counts look wrong.”

Root cause:
> Wrong aligner for spliced data.

-----

<img width="1520" height="943" alt="{19671F05-242E-4CF0-B77B-E52BB045ACBE}" src="https://github.com/user-attachments/assets/c70ef334-3b4b-42a6-a4d5-b94af92af9dc" />

#RUN TIME, MEMORY USAGE AND ALIGNER ACCURACY FROM ALIGNERS...(NATURE`s ppr)

<img width="1180" height="625" alt="{7A5FEA40-1951-45F0-A25A-C569BE431088}" src="https://github.com/user-attachments/assets/97d42462-5f0d-41c5-b2ea-2d3ccfc661b1" />


------

# good practice while executing a .sh script

## time ur script from start to end
<img width="1001" height="696" alt="{354E44DB-5EA6-49E0-92E5-8436B8C91FE9}" src="https://github.com/user-attachments/assets/b9c1ec06-795e-4ff8-846a-c7c3edbc7cd0" />
## to have executables to ur tools in ur path so that u do not have to write entire path to ur executables to each of ur tool
## it is always recommended to look at the help page of each tool before using the tool example:   fastqc -h
## make sure u have given permission to your script (.sh)

1st step -quality control- fastqc tool
<img width="1107" height="838" alt="{6AFC1D1D-A045-4BE5-9223-9EB4CDBEA8D8}" src="https://github.com/user-attachments/assets/df55cedb-d139-4a69-80e3-a58917cad7ba" />
-signal decay /phasing at end of graph ki vjah s graph 28 phred score s neeche aa jaata end m thoda
-20 phred score s agr neeche h toh trimmomatic must h...poor quuality reads h
-no adapter -no trmming
-no bad reads -no trimming
-HISAT2 soft clips the read ---masks read that don`t align

-WHEN WE NEED A LINE WE RUN ON LINUX TO GET AN OUTPUT AND WEE NEED IT AGAIN ,JUST COMMENT IT OUT(LOOK HOW TO COMMENT OUT A LINE AND THEN CONTINUE WITH UT COMMANDS


##GET THE GENOME INDICES

###SEARCH HOSAT2 DOWNLOAD ON GOOGLE

<img width="1920" height="973" alt="{D60B2B12-6F17-460B-BA0A-80DFB33CF2F1}" src="https://github.com/user-attachments/assets/8a705abb-7f1f-42c6-a988-94586e80a9fb" />

COPY LINK AND USE IT IN .SH FILE
<img width="750" height="155" alt="{4CDB2B47-8630-4659-B497-6B74814D1BAA}" src="https://github.com/user-attachments/assets/89f45a6b-61d5-47a5-9943-b6d57bcbc2df" />

-BEFORE writing the command for alignment, it is important to know if our RNA seq reads have been generated using a standard protocol, using strand-specific info in mapping improves resolution of
multi map reads & anti-cells overlap genes

how to do that??
Strandedness

https://link.springer.com/article/10.1186/s12864-015-1876-7
https://chipster.csc.fi/manual/library-type-summary.html
https://rseqc.sourceforge.net/#infer-experiment-py

# why is it imp to know and check strandedness of reads before alignment

First: what is “strandedness” in simple terms?
Strandedness answers ONE question:
** Does the RNA-seq protocol preserve which DNA strand the RNA came from? **

```
🧠 First: what is “strandedness” in simple terms?

Strandedness answers ONE question:
Does the RNA-seq protocol preserve which DNA strand the RNA came from?

Genes can exist on:

+ strand
– strand
Stranded RNA-seq remembers this information.
Unstranded RNA-seq forgets it.

🧬 Why this matters biologically
Case 1️⃣: Two genes overlap on opposite strands (very common)

If reads are stranded:

gene A gets its reads

gene B gets its reads
✔ clean separation

If reads are unstranded:

reads can be assigned to the wrong gene
❌ false expression
This directly affects:
gene counts
DEGs
pathways

🧠 Why checking strandedness BEFORE alignment is critical
🔥 Reason 1: Alignment won’t fail — it will be wrong

This is the scary part.
If you choose wrong strandedness:
HISAT2 / Salmon will still run
HTSeq will still count
DESeq2 will still give results
But:
Your counts will be biologically incorrect.
No error. No warning. Just wrong science.

# 🔥 Reason 2: Counting step depends on strandedness

Tools like HTSeq-count require:

** -s yes | no | reverse **


If this is wrong:
sense genes lose reads
antisense genes gain reads
expression patterns flip
This is unrecoverable later.

🔥 Reason 3: Library prep protocols differ

Different kits produce different strandedness:
TruSeq Stranded mRNA → usually reverse
Unstranded kits → no
Some older protocols → yes
You cannot assume.
Metadata lies sometimes.

🧠 What happens if you ignore strandedness?
Stage	What goes wrong
Alignment	Reads still align
Counting	Reads assigned to wrong genes
PCA	Samples cluster weirdly
DESeq2	DEGs look noisy
Biology	Pathways don’t make sense

This is why people say:

“My RNA-seq results look off but I don’t know why.”

Strandedness is often the reason.

🧪 How to CHECK strandedness properly
Best tool:

RSeQC – infer_experiment.py
Command:
infer_experiment.py -r hg38.bed -i sample.bam


Output tells you:
stranded forward
stranded reverse
unstranded
This is a diagnostic step and must be done early.

🧠 How this affects YOuR pipeline decisions
If using HISAT2 + HTSeq
infer strandedess
set -s accordingly in HTSeq

document it in README

If using Salmon
Salmon auto-detects strandedness
BUT you should still verify it from metadata/QC
Salmon flags:
-l A (auto)
ISR, ISF, etc.

📝 One sentence you can use in Methods

“Library strandedness was assessed prior to read counting to ensure correct assignment of reads to genes.”

That line signals competence.

🧠 One-line intuition (memorize this)

Wrong strandedness doesn’t crash your pipeline — it silently corrupts your biology.

```
-when writing new commands  in .sh  ,comment out earlier commands so that u don`t rerun them again
 like alignment step krre h toh fastq vgerah comment kr liye

 -write each part of script and go on to terminal and run it

 -feature count can tell whether a read is mapped to a gene
 -we need to download a genome annotation file before using feature count

 # how can u get a genome annotation file for feature count

 ensembl website m jaakr human m click krna h

fasta gtf vgerah m s kuch bhi dowload krlo

on .sh terminal use wget to download the link
isme bhi strandedness imp h janna


--after completing .sh, uncomment out everything so u can run the pipeline in 1 go 

### it`s always a good practice to start with 1 sample or few samples to have an rough idea of memory to be allocated and time utilised













