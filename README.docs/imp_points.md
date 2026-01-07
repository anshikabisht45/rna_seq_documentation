# short read ,long read

-For short-read length RNA-seq technologies, bias and imperfections are primarily generated in sequencing library preparation and short read assembly.
It is difficult for these methods to correctly identify multiple isoforms from a certain gene.
To overcome the disadvantage of short read length, improved read coverage and sequencing depth is required.


-Long-read length RNA-seq technologies avoid shortcomings in template amplification, reduce the false positive rate in splice junction detection and enable the identification of unannotated longer transcripts,
overcoming the common limitations of short-read sequencing [176, 177]. However, this method suffers from the drawback of reduced throughput, higher cost and higher sequencing error rate, especially insertion-deletion
errors. 
-To reduce random errors, PacBio circular consensus-sequencing (CCS) was developed to increase sequencing depth by rereading molecules several times
However, it also reduces the identification rate of unique isoforms. In addition, the sensitivity of long-read sequencing for identification of differentially expressed genes is lower compared to short-read
sequencing 

# to trim or not trim

https://pmc.ncbi.nlm.nih.gov/articles/PMC7671312/

some points from above ppr

- In this study, we used a benchmark RNA-seq dataset and simulation data to assess the impact of read trimming on mapping and quantification of RNA-seq reads
- `adapter defination` -An adapter sequence in genomics is a short, synthetic DNA segment ligated to the ends of fragmented DNA (inserts) during library preparation for Next-Generation Sequencing (NGS),
                     providing crucial components like primer binding sites (P5/P7), unique sample barcodes (indexes), and flow cell attachment sequences to enable clonal amplification and sequencing on the machine.
-  modern read aligners are known to be able to ‘soft-clip’ read bases that cannot be mapped along with the majority of bases in a read

# soft clipping vs read trimming

On the other hand, modern read aligners are known to be able to ‘soft-clip’ read bases that cannot be mapped along with the majority of bases in a read (15–17).
Soft-clipped bases are still included in the mapping results of the reads but are marked as ‘soft-clipped’. Both soft-clipping and read trimming remove bases from the ends of the reads, 
but `soft-clipping is performed within the read mapping procedure` whereas `read trimming is performed prior to mapping as a standalone procedure`

When performing `read trimming, a lot of parameters need to be specified such as adapter sequences and threshold for quality filtering.` 
In contrast, `soft-clipping is performed solely based on the matching of read bases with reference sequences and it does not require users to provide any parameters.`

Step-by-step explanation
# 1️⃣ How much trimming happened?

2.3–4.6% of all read bases were trimmed
This means trimming removed only a few percent of total bases
Sounds small, but in RNA-seq that’s millions of bases
Also:
Trimmomatic removed ~2× more bases than TrimGalore
So Trimmomatic = more aggressive trimming

# 2️⃣ What happened after trimming + mapping?

Successfully mapped bases dropped by 1.3–4.0%
This is the key red flag 🚨
Trimming → fewer bases map
That means trimming didn’t help mapping — it hurt it
So trimming ≠ automatically better alignment.

# 3️⃣ What did Subread (the aligner) do instead?

Subread soft-clipped 18–29% of bases that trimmers removed
Soft-clipping = smart trimming during alignment
Instead of deleting bases upfront:
Subread keeps the read
Clips the bad part only if needed
Uses the good part for mapping
S a big chunk of “trimmed-away” bases were actually rescued during alignment.

# 4️⃣ Were those bases really adapters?

Only 10–27% were adapters; rest were low-quality bases
Important nuance 👇
Most trimmed bases were not adapters
They were just low-quality ends
Low-quality ≠ unusable (aligners can often tolerate them)

# 5️⃣ Adapter handling: Subread vs trimmers

Subread soft-clipped:
94% of adapters removed by Trimmomatic ✅
Meaning: Subread already handles adapters really well
So trimming adapters before alignment may be redundant.

# 6️⃣ TrimGalore issue (this is subtle but important)

TrimGalore reported ~6× more adapters than Trimmomatic
But
Many of those adapters were very short
Likely false positives
Evidence:
Only ~30% of TrimGalore-called adapters were soft-clipped by Subread

Translation:
👉 TrimGalore may be overcalling adapters, especially short ones.

# Final conclusion (why this matters)
What the authors are really saying:
Modern aligners (like Subread) are very good at handling messy read ends

Aggressive trimming:

removes usable bases
reduces mapped signal
can hurt downstream quantification
So the takeaway is:
Let the aligner do some of the cleaning. Over-trimming is often worse than under-trimming.

# Industry / data-analyst framing 
This paragraph supports a very important mindset:
Trimming is a trade-off, not a default
You should:
inspect QC
understand aligner behavior
justify trimming thresholds
Not blindly trim “because tutorials say so”
This is exactly the kind of reasoning companies expect:
“Why did you trim? Why that much? What did it cost you?”


<img width="729" height="848" alt="image" src="https://github.com/user-attachments/assets/8121f7ec-fdb1-4228-85cb-71272ab0abbd" />

# Step 1: What trimming does

Trimmers (Trimmomatic / TrimGalore):
DELETE bases from read ends
Those bases never reach the aligner
Aligner never sees them again
So:
❌ Subread cannot correct already-trimmed bases
❌ Subread cannot “rescue” deleted bases

# Step 2: What Subread does instead

Subread (aligner) does soft-clipping:

It keeps the entire read
During alignment, it:
ignores bad ends
clips adapters only if needed
still uses the good part
This is dynamic, smarter trimming.

<img width="413" height="581" alt="{B1CD601A-73D3-4E6C-8A46-C92B42ACD21F}" src="https://github.com/user-attachments/assets/f3bc7d68-2c2c-4e2e-9116-908c6c20da86" />
<img width="624" height="443" alt="{D802D618-892E-4FC8-8063-33D7323C1C78}" src="https://github.com/user-attachments/assets/186aac3f-fed5-4970-afca-470bd154acd9" />
<img width="451" height="402" alt="{EA85B582-EA74-4476-97B0-7E839D553B9E}" src="https://github.com/user-attachments/assets/3d432722-1a45-4192-b00f-48434a533810" />
<img width="629" height="617" alt="{7C271066-3B2C-414B-AEF9-9B4FD7EC1AD8}" src="https://github.com/user-attachments/assets/aad5b20c-b6d6-48b7-ac74-13a48e2ac589" />
<img width="790" height="537" alt="{246A5522-898B-4D1B-B87F-4094AA4D8F5C}" src="https://github.com/user-attachments/assets/67542a58-82f1-4fa8-bce9-b3f833374b63" />









- These adapters make the library compatible with the sequencer, allowing thousands of samples to be pooled and identified, making them essential for modern high-throughput sequencing. 
