# Bioinformatic Container Index 🧬

## Overview
Containerized bioinformatics tools for RNA-seq and genomic analysis workflows.

---

## All Available Tools

| Tool | Description | Primary Use Case |
|------|-------------|------------------|
| **fastp** | Ultra-fast all-in-one QC & trimming | Quality control, adapter removal, quality filtering |
| **trimmomatic** | Flexible read trimming tool | Alternative trimming strategy with complex parameters |
| **star** | Spliced Alignment Tool | Fast RNA-seq alignment (splice-aware) |
| **featurecounts** | Feature counting from alignments | Gene-level quantification |
| **seqtk** | FASTA/FASTQ manipulation utility | Sequence subsetting, filtering, conversion |
| **gffread** | GFF/GTF manipulation & visualization | Convert GTF to BED, extract transcripts |
| **multiqc** | Aggregate analysis results | QC report aggregation from multiple tools |
| **samtools** | SAM/BAM/CRAM manipulation suite | Sort, index, view, manipulate alignments (essential!) |
| **bedtools** | Genomic arithmetic & interval operations | Peak overlap, windowing, coverage, distance calculations |
| **RSeQC** | RNA-seq quality control | Transcript assembly QC, junction detection, GC bias |
| **stringtie** | Transcript assembly from RNA-seq data | Transcript quantification and differential expression |
| **sambamba** | High-performance SAM/BAM/CRAM toolkit | Faster alternative to samtools (sort, index, merge) |

---

## Quick Start Guide

```bash
# Start all containers at once
cd bioworkbench
docker-compose up -d

# Access individual containers
docker exec -it <container_name> bash

# Stop all containers
docker-compose down
```

---

## Typical Workflow Example

### 1. Raw Data QC & Trimming
```bash
docker exec -it fastp bash
fastp -i input.fastq.gz -o output.fastq.gz -h report.html -j report.json
exit
```

### 2. Alignment with STAR
```bash
docker exec -it star bash
STAR --runThreadN 8 \
     --genomeDir /path/to/genome \
     --readFilesIn input.fastq.gz \
     --readFilesZ false \
     --outFileNamePrefix sample. \
     --outSAMtype BAM SortedByCoordinate
exit
```

### 3. Post-alignment QC with samtools + RSeQC
```bash
docker exec -it samtools bash
samtools index output.bam
samtools flagstat output.bam > stats.txt

exit

docker exec -it RSeQC bash
gffread annotations.gtf -E -o /dev/null
junction_saturation.py -r 100.bed -i junctions.bed -o saturation.txt
exit
```

### 4. Gene Quantification
```bash
docker exec -it featurecounts bash
featureCounts -T 8 -a annotations.gtf -o counts.txt output.bam
exit
```

### 5. Transcript Assembly (Optional)
```bash
docker exec -it stringtie bash
stringtie output.bam -G reference.gtf -o sample.gt -p 8
exit
```

### 6. Interval Operations with bedtools
```bash
docker exec -it bedtools bash
bedtools intersect -a peaks.bed -b genes.bed -u > peak_genes.bed
bedtools coverage -a reads.bam -b regions.bed -d
exit
```

### 7. QC Report Aggregation
```bash
docker exec -it multiqc bash
multiqc . -o ../reports/
exit
```

---

## Tool-Specific Usage Patterns

### 📊 **samtools** - BAM/SAM Manipulation
```bash
# Index BAM files
bam index output.bam

# Sort and merge
bam sort -o merged.bam input1.bam input2.bam
bam merge -o merged.bam file1.bam file2.bam

# View/extract regions
bam view chr1:1000-2000 > region.sam

# Stats and metrics
flagstat output.bam
depth -r chr1:1000-5000 output.bam
```

### 🔧 **bedtools** - Genomic Operations
```bash
# Find overlaps
intersect -a regions.bed -b reads.bed -u

# Calculate coverage
coverage -a reads.bam -b genome.bed

# Get closest feature
closest -a reads.bed -b annotations.gtf

# Window operations
makewindows -g genome.chrom.sizes -w 1000 -s 500 > windows.bed
```

### 📈 **RSeQC** - RNA-seq QC
```bash
# Gene body coverage
geneBody_coverage.py -r ref.gtf -i aligned.bam -o coverage.txt

# Inner distance (insert size distribution)
inner_distance.py -1 r1.fastq -2 r2.fastq -o inner_dist.txt

# Junction annotation
junction_annotation.py -i junctions.bed -g genes.gtf
```

### 🧬 **stringtie** - Transcript Assembly
```bash
# Assemble transcripts
stringtie aligned.bam -G reference.gtf -o sample.gt -p 8 --rf

# Merge assemblies
stringtie-merge -p 8 *.gt > merged.gtf

# Quantify from reference
quantReference -G reference.gtf -A quant.txt merged.gtf
```

### ⚡ **sambamba** - Fast BAM Operations
```bash
# Index (faster than samtools)
index output.bam

# Sort (much faster for large files)
sort -o sorted.bam unsorted.bam

# Merge multiple BAMs
merge -o merged.bam file1.bam file2.bam file3.bam

# Extract by regions
view -r chr1:1000-2000 output.bam > region.bam
```

---

## Common Use Cases by Analysis Type

### RNA-seq Analysis Pipeline
```
fastp → star → samtools (index) → featurecounts + RSeQC QC → multiqc
                         ↓
                    stringtie → transcript analysis
```

### ChIP-seq/ATAC-seq Pipeline
```
fastp → trimmomatic → star → samtools → bedtools (peak calling) → RSeQC → multiqc
```

### Variant Calling Prep
```
fastp → samtools (sort/index) → GATK best practices
```

---

## Tips & Best Practices

### Performance Optimization
- Use **sambamba** instead of samtools for faster operations on large BAM files
- Set appropriate `--runThreadN` in STAR based on available RAM (default 8-16 threads)
- Use `-p` parameter in stringtie/sambamba for parallelization

### Storage Management
- Keep input data in `/data` volume mounted from host
- Export outputs to separate directory per analysis run
- Clean intermediate files regularly

### Container Access Patterns
```bash
# Start specific container only
docker-compose up -d samtools bedtools RSeQC

# Stop and remove all containers
docker-compose down --remove-orphans

# View logs
docker-compose logs -f <container_name>

# Copy files from/to container
docker cp <container>:<container_path> ./host_path
```

---

## Version Information

| Tool | BioContainers Version | Docker Image |
|------|----------------------|--------------|
| fastp | Latest biocontainers release | `biocontainers/fastp` |
| trimmomatic | 0.12+ | `biocontainers/trimmomatic` |
| star | 2.7.x | `biocontainers/star` |
| featurecounts | 2.0.x (subread) | `biocontainers/subread` |
| seqtk | Latest biocontainers release | `biocontainers/seqtk` |
| gffread | 0.12.x | `biocontainers/gffread` |
| multiqc | 1.18+ | `biocontainers/multiqc` |
| **samtools** | 1.20 | `biocontainers/samtools` |
| **bedtools** | 2.30 | `biocontainers/bedtools` |
| **RSeQC** | 3.0.2 | `biocontainers/RSeQC` |
| **stringtie** | 2.2.1 | `biocontainers/stringtie` |
| **sambamba** | 0.7.3 | `biocontainers/sambamba` |

---

## Troubleshooting

### Common Issues
1. **"Permission denied" on files** - Ensure correct file permissions in mounted volumes
2. **"Container not found"** - Check if service is started: `docker-compose ps`
3. **"No space left on device"** - Clean container images: `docker system prune`
4. **Tool not found** - Verify BioContainers version compatibility

---

## Additional Tools You Might Want Later

### For Variant Analysis
- **bcftools** - Variant calling and filtering
- **GATK** - Genomics Analysis Toolkit (best practices)
- **freebayes** - Population-scale variant caller

### For Epigenetics/ChIP-seq
- **MACS2** - Peak calling
- **deepTools** - Visualization and analysis of ChIP-seq/ATAC-seq data
- **HOMER** - Motif analysis and peak annotation

### For Metagenomics
- **Kraken2** - Taxonomic classification
- **FastQ Screen** - Contaminant detection
- **CheckM** - Genome quality assessment

---

## Credits

All tools are from [BioContainers](https://biocontainers.pro/) project, providing containerized bioinformatics software that matches the Bioconductor package ecosystem.

*Last updated: 2026-10-27*