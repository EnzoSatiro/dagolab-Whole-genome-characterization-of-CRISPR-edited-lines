# dagolab-Whole-genome-characterization-of-CRISPR-edited-lines
The new EU framework for NGTs focuses on the final plant's molecular traits rather than the editing process. Developers must prove the presence of intended edits and the absence of foreign DNA. We present an integrated long-read whole-genome sequencing workflow for the comprehensive characterization of CRISPR/Cas9-edited lines.

## Bioinformatics Analysis

Command lines utilized within the bioinformatics workflow.

| Function | Software | Command |

| **Quality control** | *LongQc* | `python3 longQC.py sampleqc -o /path/to/output_QC -x ont-ligation /path/to/input_reads.fastq.gz` |

| **Trimming** | *Dorado* | `dorado trim /path/to/input_reads.fastq.gz --sequencing-kit SQK-LSK114 --emit-fastq > /path/to/trimmed_reads.fastq` |

| **Read to vector alignment** | *Minimap2* | `minimap2 -ax map-ont -t 8 /path/to/ccd7_vector.fa /path/to/input_reads.fastq.gz -o /path/to/output/aln_vector.sam` |

| **Kmer matching** | *Minimap2* | `minimap2 -a -x sr -k 15 -w 5 -t 4 --sam-hit-only /path/to/kmer.fasta /path/to/trimmed_reads.fastq -o /path/to/output/aln_kmer.sam` |

| **Mapping vs Reference** | *Minimap2* | `minimap2 -ax map-ont -t 8 /path/to/reference/S_lycopersicum.4.00.fa.gz /path/to/trimmed_reads.fastq -o /path/to/output/aln_reference.sam` |

| **Off-target read selection** | *Bedtools* | `bedtools intersect -abam aln_reference.bam -b <(echo -e "chr_id\tstart\tend") > off_target_reads.bam` |

| **On-target read selection** | *Bedtools* | `bedtools intersect -abam aln_reference.bam -b <(echo -e "chr_id\tstart\tend") > on_target_reads.bam` |

| **Insertion detection** | *Sniffles* | `sniffles --input /path/to/aln_reference.bam --vcf output_sniffles.vcf --minsvlen 35 --threads 8` |

| **Insertion filtering** | *Bcftools* | `bcftools filter -i 'INFO/AF>0.2 && INFO/SUPPORT>=10 && INFO/COVERAGE>=10 && INFO/COVERAGE<=60' input_INS.vcf.gz -o filtered_INS.vcf` |

| **Insertion validation** | *MMseqs* | `mmseqs easy-search /path/to/INS_sequences.fasta /path/to/database_nt /path/to/output_aln_DB tmp --search-type 3` |
