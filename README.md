# dagolab-Whole-genome-characterization-of-CRISPR-edited-lines
The new EU framework for NGTs focuses on the final plant's molecular traits rather than the editing process. Developers must prove the presence of intended edits and the absence of foreign DNA. We present an integrated long-read whole-genome sequencing workflow for the comprehensive characterization of CRISPR/Cas9-edited lines.

<img width="4247" height="3065" alt="Figure1" src="https://github.com/user-attachments/assets/b6f6b64e-ada3-4882-8190-ab83765a55b8" />

Workflow for comprehensive molecular characterization of genome-edited plants. Each analytical module is designed to generate evidence addressing a specific molecular endpoint relevant to the European NGT regulatory framework. (a1) Quality control of raw Oxford Nanopore sequencing reads. (a2) Read trimming to remove low-quality sequences, adapters, and barcodes, generating high-quality (HQ) reads. (b1) Alignment of HQ reads to the complete vector sequences. (b2) Comparison of HQ reads with vector sequences fragmented into 30-bp k-mers to improve the detection of vector-derived sequences. (c1) Alignment of HQ reads to the reference genome. (c2) Identification of potential off-target mutations. (c3) Verification of the expected on-target mutations. (c4) Detection and validation of potential genomic insertions.

## Bioinformatics Analysis

Command lines utilized within the bioinformatics workflow.

| Function | Software | Command |

| **Quality control** | *LongQc* | `python3 longQC.py sampleqc -o /path/to/output_QC -x ont-ligation /path/to/input_reads.fastq.gz` |

| **Trimming** | *Dorado* | `dorado trim /path/to/input_reads.fastq.gz --sequencing-kit SQK-LSK114 --emit-fastq > /path/to/trimmed_reads.fastq` |

| **Read to vector alignment** | *Minimap2* | `minimap2 -ax map-ont -t 8 /path/to/ccd7_vector.fa /path/to/input_reads.fastq.gz -o /path/to/output/aln_vector.sam` |

| **Kmer matching** | *Minimap2* | `minimap2 -a -x sr -k 15 -w 5 -t 4 --sam-hit-only /path/to/kmer.fasta /path/to/trimmed_reads.fastq -o /path/to/output/aln_kmer.sam` |

| **Mapping vs Reference** | *Minimap2* | `minimap2 -ax map-ont -t 8 /path/to/reference/reference_genome.fa.gz /path/to/trimmed_reads.fastq -o /path/to/output/aln_reference.sam` |

| **Off-target read selection** | *Bedtools* | `bedtools intersect -abam aln_reference.bam -b <(echo -e "chr_id\tstart\tend") > off_target_reads.bam` |

| **On-target read selection** | *Bedtools* | `bedtools intersect -abam aln_reference.bam -b <(echo -e "chr_id\tstart\tend") > on_target_reads.bam` |

| **Insertion detection** | *Sniffles* | `sniffles --input /path/to/aln_reference.bam --vcf output_sniffles.vcf --minsvlen 35 --threads 8` |

| **Insertion filtering** | *Bcftools* | `bcftools filter -i 'INFO/AF>0.2 && INFO/SUPPORT>=10 && INFO/COVERAGE>=10 && INFO/COVERAGE<=60' input_INS.vcf.gz -o filtered_INS.vcf` |

| **Insertion validation** | *MMseqs* | `mmseqs easy-search /path/to/INS_sequences.fasta /path/to/database_nt /path/to/output_aln_DB tmp --search-type 3` |
