# Bulk RNAseq public dataset analysis
Download R1 and R2 files and decompress using gunzip

trimming: trim_galore

alignment: hisat2

featureCounts -p -a 

```
trim_galore --paired R1.fastq R2.fastq
```
```
# Download GRCm38 mm10 reference genome
wget ftp://ftp.ensembl.org/pub/release-102/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna.primary_assembly.fa.gz
gunzip Mus_musculus.GRCm38.dna.primary_assembly.fa.gz

# Download GRCm38 mm10 Annotation GTF
wget ftp://ftp.ensembl.org/pub/release-102/gtf/mus_musculus/Mus_musculus.GRCm38.102.gtf.gz
gunzip Mus_musculus.GRCm38.102.gtf.gz
```

```
# Rename the file to mm10.fa and mm10.gtf
mv Mus_musculus.GRCm38.dna.primary_assembly.fa mm10.fa
mv Mus_musculus.GRCm38.102.gtf mm10.gtf
```

```
# Build index file for alignment
# This takes a lot of time and generates several mm10_*index files
hisat2-build mm10.fa mm10_index
```

```
# Align reads to the genome 
hisat2 -x mm10_index \ -1 R1_val_1.fq \ -2 R2_val_2.fq \ -S S1.sam \ --threads 4
```

```
# Convert to bam file
samtools view -bS S1.sam | samtools sort -o S1.sorted.bam
```

```
# Convert to sorted bam file
samtools index S1.sorted.bam
```
```
featureCounts -p -a mm10.gtf -o counts1.txt S1.sorted.bam
```
