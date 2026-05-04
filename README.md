# Trio-Based Variant Analysis Pipeline (Chromosome 20)

## Overview

This pipeline implements a complete workflow for variant discovery and filtering in trio-based sequencing data (child, father, mother), focusing on chromosome 20 (GRCh38.p14 assembly).

The workflow includes:

* Input acquisition
* Alignment and quality control
* Variant calling
* Inheritance-based filtering
* Coverage track generation

The pipeline is written in Bash and supports processing of multiple trios iteratively.

---

## Pipeline Structure

The pipeline consists of the following steps:

1. Input Acquisition
2. Inheritance Model Selection
3. Alignment & Quality Control
4. Variant Calling
5. Variant Filtering
6. Coverage Track Generation

---

## Requirements

Make sure the following tools are installed and available in your $PATH:

* Bowtie2
* Samtools
* Qualimap
* FastQC
* MultiQC
* FreeBayes
* BCFtools
* Bedtools
* bgzip (HTSlib)

---

## Input Data

### Required Reference Files

* chr20* (Bowtie2 index)
* chr20.fa (reference genome)
* chr20_bed (target regions)
* samples.txt (sample list)

### Raw Sequencing Data

Paired-end FASTQ files:

*_R1.fq.gz
*_R2.fq.gz

### Directory Structure

Raw data must be located in:

/home/BCG2026_exam//<case_name>/

---

## Execution

Run the pipeline:

bash pipeline.sh

### Interactive Inputs

You will be prompted to enter:

* Case names (multiple trios separated by spaces)
* Sample IDs (ordered as: child father mother)
* Username (used to locate input data)

---

## Step 1 – Input Acquisition

The pipeline:

* Copies chromosome 20 reference index files
* Copies samples.txt
* Collects user input

---

## Step 2 – Inheritance Model Selection

For each trio, select one of the following models:

AR  → Autosomal Recessive
AD  → Autosomal Dominant
ADN → Autosomal De Novo

### Additional Input (AD only)

Specify affected parent:

1 → Father
2 → Mother

### Genotype Filtering Logic

AR:
child = AA, parents = RA

AD:
child = RA, one parent = RA, other = RR

ADN:
child = RA, parents = RR

---

## Step 3 – Alignment & Quality Control

For each trio:

* A dedicated directory is created
* FASTQ files are copied locally

### Alignment

* Tool: Bowtie2
* Paired-end mode (-1, -2)
* Reference: chromosome 20

### BAM Processing

samtools view -Sb
samtools sort
samtools index

### Quality Control

* Qualimap → BAM quality & coverage (BED-restricted)
* FastQC → Read-level QC
* MultiQC → Aggregated reports

---

## Step 4 – Variant Calling

Performed using FreeBayes:

freebayes -f ../chr20.fa -m 20 -C 5 -Q 10 --min-coverage 10 child.bam father.bam mother.bam > case.vcf

### Parameters

* -m 20 → minimum mapping quality
* -C 5 → minimum alternate allele count
* -Q 10 → minimum base quality
* --min-coverage 10 → minimum depth

### Post-processing

bgzip case.vcf
bcftools index case.vcf.gz

---

## Step 5 – Variant Filtering

Filtering pipeline:

bcftools view -R chr20_bed case.vcf.gz |
bcftools view -S samples.txt |
bcftools view -i "" |
bcftools filter -i 'QUAL>20' -Ov -o case.cand.vcf

### Filtering Steps

1. Region filtering (BED file)
2. Sample selection
3. Genotype pattern filtering
4. Quality filtering (QUAL > 20)

### Output

<case_name>.cand.vcf

---

## Step 6 – Coverage Track Generation

Coverage tracks are generated using:

bedtools genomecov -ibam sample.bam -bg -trackline -trackopts "name=sample" -max 100 > sampleCov.bg

### Generated for:

* child
* father
* mother

These files can be visualized in genome browsers (e.g. UCSC Genome Browser).

---

## Output Files

For each case:

* child.bam, father.bam, mother.bam
* BAM index files (.bai)
* QC reports (Qualimap, FastQC, MultiQC)
* Raw VCF (.vcf.gz)
* Filtered VCF (.cand.vcf)
* Coverage tracks (*.bg)

---

## Notes

* The pipeline is restricted to chromosome 20
* Sample order must be: child father mother
* The same sample IDs are reused across cases
* Ensure correct file permissions and paths

---

## Possible Improvements

* Add logging system
* Parallelize execution
* Extend to multiple chromosomes
* Improve error handling
* Automate reporting

---

## Pipeline Completion

At the end of execution, the script prints:

Pipeline execution ended

---

## Author

Vincenzo Ricciardi, Sara Likramaj

