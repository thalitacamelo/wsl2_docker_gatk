# Files preparation
## Change to your data directory and decompress the files.
```bash
gzip -d *.gz
```
## Filter chromossome 22 from dbSNP. Output: dbSNP_chr22.vcf
```bash
grep -w '^#\|^#CHROM\|^22' 00-common_all.vcf > dbSNP_chr22.vcf
```
### Correct 22 to chr22. Output: dbSNP_chr22_corrected.vcf
```bash
awk '{if($0 !~ /^#/) print "chr"$0; else print $0}' dbSNP_chr22.vcf > dbSNP_chr22_corrected.vcf
```

# FastQC
## Open a new terminal start a container of the FastQC image, specifying the filesystem location you want to mount.
```bash
docker run -v ~/GATK/DesafioMendelics:/data -it biocontainers/fastqc:v0.11.9_cv8 bash
```
### Run FastQC
```bash
fastqc -t 4 -o amostra-lbb_R1.fq amostra-lbb_R2.fq
```
### Start python webserver to visualize the results on `http://localhost:8000/` and check some quality parameters.
```
python3 -m http.server
```

# Samtools
## Create reference fasta index. Open another terminal and start a samtools container. Output: grch38.chr22.fasta.fai.
```bash
# Open a new termical and start a samtools container.
docker run -v ~/GATK/DesafioMendelics:/data -it biocontainers/samtools:v1.9-4-deb_cv1 bash
```
```bash
# Then Create fasta index
samtools faidx grch38.chr22.fasta
```

# GATK
## Open a new terminal and start a container of the GATK4 image, specifying the filesystem location you want to mount.
docker run -v ~/GATK/DesafioMendelics:/gatk/my_data -it broadinstitute/gatk:latest bash

## Convert the reads files to one BAM file. Output: unaligned_read_pairs.bam
```bash
../gatk FastqToSam -F1 amostra-lbb_R1.fq -F2 amostra-lbb_R2.fq -O unaligned_read_pairs.bam --SAMPLE_NAME AMOSTRA-LBB
```

## Create an index for VCF files. Outputs: dbSNP_chr22_corrected.vcf.idx, pequeno-gabarito.vcf.idx.
### An index allows querying features by a genomic interval. 
### The -java-options -Xmx6G argument specify memory allocation
```bash
../gatk --java-options -Xmx6G IndexFeatureFile -I dbSNP_chr22_corrected.vcf
../gatk --java-options -Xmx6G IndexFeatureFile -I pequeno-gabarito.vcf
```

## Create BWA-MEM index image file. Tools that utilize BWA-MEM (e.g. BwaSpark, PathSeqBwaSpark) require an index image file of the reference sequences. Output: reference.img
```bash
../gatk BwaMemIndexImageCreator -I grch38.chr22.fasta -O grch38.chr22.fasta.img
```

## Create sequence dictionary. Output: reference.dict.
```bash
../gatk CreateSequenceDictionary -R grch38.chr22.fasta
```

## Alignment of reads to reference sequence. Outputs: aligned_reads.bam and aligned_reads.bam.sbi
```bash
../gatk --java-options -Xmx6G BwaSpark -I unaligned_read_pairs.bam -R grch38.chr22.fasta -O aligned_reads.bam
```

## Mark Duplicate reads. Sequencing error propagate in duplicates, flagging them mitigates duplication artifacts. 
### Outputs: marked_dup_metrics.txt, marked_duplicates.bam, marked_duplicates.bam.bai, marked_duplicates.bam.sbi
```bash
../gatk --java-options -Xmx6G MarkDuplicatesSpark -I aligned_reads.bam -O marked_duplicates.bam -M marked_dup_metrics.txt
```

## Many GATK tools require at least one read group tag in BAM files. Output: marked_duplicates_rg.bam.
```bash
../gatk --java-options -Xmx6G AddOrReplaceReadGroups -I marked_duplicates.bam -O marked_duplicates_rg.bam -LB AMOSTRA-LBB -PL ILLUMINA -PU AMOSTRA-LBB -SM AMOSTRA-LBB
```

## Base Quality Score Recalibration (Create table). Output:recal_data.table
```bash
../gatk --java-options -Xmx6G BaseRecalibrator -I marked_duplicates_rg.bam -R grch38.chr22.fasta --known-sites dbSNP_chr22_corrected.vcf -O recal_data.table
```

## Base Quality Score Recalibration (apply). Outputs: analysis_ready_reads.bai, analysis_ready_reads.bam
```bash
../gatk --java-options -Xmx6G ApplyBQSR -I marked_duplicates_rg.bam -R grch38.chr22.fasta --bqsr-recal-file recal_data.table -O analysis_ready_reads.bam
```

## Call variants using Haplotype Caller. Use the dbSNP and capture files to improve quality of the analysis. Outputs: variants.vcf, variants.vcf.idx
```bash
../gatk --java-options -Xmx6G HaplotypeCaller -I analysis_ready_reads.bam -R grch38.chr22.fasta --dbsnp dbSNP_chr22_corrected.vcf -L coverage.bed -O variants.vcf
```

## Apply tranche filtering to VCF based on scores. Outputs: variants_score.vcf, variants_score.vcf.idx, variants_filtered.vcf, variants_filtered.vcf.idx
```bash
../gatk --java-options -Xmx6G CNNScoreVariants -R grch38.chr22.fasta -V variants.vcf -O variants_score.vcf
```
```bash
../gatk --java-options -Xmx6G FilterVariantTranches -V variants_score.vcf --resource dbSNP_chr22_corrected.vcf --info-key CNN_1D -O variants_filtered.vcf
```

## Funcotator (FUNCtional annOTATOR). 
### Data Source Downloader Tool.
```bash
../gatk FuncotatorDataSourceDownloader --germline --validate-integrity --extract-after-download
```
### Change to funcotator_dataSources.v1.7.20200521g directory
```bash
cd funcotator_dataSources.v1.7.20200521g
```
### Extract gnomAD_exome.tar.gz
```bash
tar -zxf gnomAD_exome.tar.gz
```
### Exectue Funcotator annotation. Output: variants_annotation.vcf
```bash
../gatk --java-options -Xmx6G Funcotator -V variants_filtered.vcf -R grch38.chr22.fasta -O variants_annotation --output-file-format VCF --data-sources-path funcotator_dataSources.v1.7.20200521g --ref-version hg38
```

## Collects summary and per-sample metrics about variant calls in a VCF file.
```bash
../gatk --java-options -Xmx6G CollectVariantCallingMetrics -I variants_filtered.vcf --DBSNP dbSNP_chr22_corrected.vcf -O summary_variants
```