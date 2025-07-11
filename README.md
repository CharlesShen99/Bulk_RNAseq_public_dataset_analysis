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
Download GENCODE vM17 GTF
```
rsync -avz rsync://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M17/gencode.vM17.annotation.gtf.gz ./
gunzip gencode.vM17.annotation.gtf.gz
```
Extract splice sites
```
hisat2_extract_splice_sites.py gencode.vM17.annotation.gtf > mm10_splicesites.txt
```
```
# sample_L001_R1.fastq.gz
# sample_L001_R2.fastq.gz
# ...
# Combine Reads
cat sample_L00{1,2,3,4}_R1.fastq.gz > sample_combined_R1.fastq.gz
cat sample_L00{1,2,3,4}_R2.fastq.gz > sample_combined_R2.fastq.gz
```
```
# Step 1: Filter for mapped, MAPQ ≥ 20
samtools view -h -F 4 -q 20 -b sample.sam > sample.q20.bam

# Step 2: Remove non-unique reads using NH tag
samtools view -h sample.q20.bam | grep -v 'NH:i:[2-9]' | samtools view -bS - > sample.unique.bam

# Step 3: Sort and index
samtools sort -@ 8 -o sample.unique.sorted.bam sample.unique.bam
samtools index sample.unique.sorted.bam
```


