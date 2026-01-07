#  Table: Single-cell RNA-seq workflow (Cell Ranger → Analysis)

| Stage                    | Input → Output                       | What actually happens                        | Why it matters (analyst lens)                    |
| ------------------------ | ------------------------------------ | -------------------------------------------- | ------------------------------------------------ |
| **Raw input**            | `fastq.gz`                           | Sequenced reads from droplet-based scRNA-seq | Starting point; errors here propagate everywhere |
| **Demultiplexing**       | I1, R1, R2 FASTQs                    | I1 = cell barcode, R1 = UMI, R2 = cDNA       | Separates *who* (cell) from *what* (gene)        |
| **Cell Ranger `count`**  | FASTQs → gene-barcode matrix         | End-to-end processing by 10x Genomics        | Industry standard for 10x data                   |
| **Barcode extraction**   | Reads → barcodes + UMIs              | Identify cell barcode & UMI per read         | Enables molecule-level counting                  |
| **Barcode correction**   | Error-correct barcodes (Hamming ≤ 1) | Fix sequencing errors in barcodes            | Prevents artificial cell inflation               |
| **Read alignment**       | cDNA reads → genome                  | Align reads using **STAR**                   | Maps reads to biological coordinates             |
| **Gene tagging**         | Reads → gene annotations             | Assign reads to genes/transcripts            | Links reads to features                          |
| **UMI correction**       | Collapse UMIs (Hamming ≤ 1)          | Remove PCR duplicates                        | Makes counts quantitative                        |
| **UMI counting**         | Cell × Gene UMI counts               | Final raw count matrix                       | Core object for all downstream steps             |
| **Cell calling**         | Select real cells                    | Separate cells vs empty droplets             | Removes ambient RNA noise                        |
| **Gene–cell matrix**     | Sparse matrix                        | Raw expression data                          | Input for QC & normalization                     |
| **Cell filtering**       | Remove low-quality cells             | Filter by genes, UMIs, mito %                | Removes dead / stressed cells                    |
| **Gene filtering**       | Remove low-information genes         | Drop rarely expressed genes                  | Improves signal-to-noise                         |
| **Normalization**        | Counts → normalized values           | Depth correction + log transform             | Enables cross-cell comparison                    |
| **Scaling & regression** | Adjust expression values             | Regress mito %, depth, cell cycle            | Reduces technical confounding                    |
| **PCA**                  | High-D → low-D                       | Capture major variance axes                  | Prep for clustering                              |
| **UMAP**                 | PCA → 2D                             | Nonlinear visualization                      | Human-interpretable structure                    |
| **Graph clustering**     | Cells → clusters                     | Community detection on kNN graph             | Identifies cell types/states                     |
| **Final output**         | Clean matrix + clusters              | Analysis-ready dataset                       | Ready for biology or ML                          |

# 🧠 Table 2: “Hard” concepts explained simply

| Concept                    | Meaning in plain language    | Why it’s critical                     |
| -------------------------- | ---------------------------- | ------------------------------------- |
| **Cell barcode**           | ID tag for each cell         | Without it, single-cell ≠ single-cell |
| **UMI**                    | ID tag for each RNA molecule | Fixes PCR amplification bias          |
| **Hamming distance**       | Number of base mismatches    | Used for error correction             |
| **STAR alignment**         | Splice-aware genome mapping  | Required for eukaryotic RNA           |
| **Ambient RNA**            | Free RNA in droplets         | Causes false expression               |
| **Mitochondrial %**        | Fraction of mito genes       | Proxy for cell stress/death           |
| **Normalization**          | Correct for sequencing depth | Raw counts aren’t comparable          |
| **Regression**             | Remove technical effects     | Avoids fake clusters                  |
| **Graph-based clustering** | Network-based grouping       | Scales to large datasets              |
| **Cell × Gene matrix**     | Rows = cells, cols = genes   | Core data product                     |


RAW FASTQ FILES
(I1, R1, R2)
      |
      v
Cell Ranger count
      |
      v
Extract barcodes + UMIs
      |
      v
Correct barcodes (Hamming ≤ 1)
      |
      v
Align reads to genome (STAR)
      |
      v
Tag reads with genes
      |
      v
Correct UMIs (PCR collapse)
      |
      v
Count UMIs per cell & gene
      |
      v
Select real cells (cell calling)
      |
      v
CELL × GENE COUNT MATRIX
      |
      v
Cell & gene QC filtering
      |
      v
Normalization + scaling
      |
      v
PCA → UMAP
      |
      v
Graph-based clustering
      |
      v
CLEAN, ANALYSIS-READY scRNA-seq DATASET

