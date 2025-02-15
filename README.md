# Bulk-RNA_Pipeline

conda create -n pipeline_ rna=3.9
conda activate pipeline_ rna

conda install -c bioconda fastqc trimmomatic hisat2 samtools sra-tools subread

prefetch SRR14136828
fastq-dump --split-files --gzip -O ~/sra_data SRR14136828

fastqc ~/sra_data/SRR14136828_1.fastq.gz ~/sra_data/SRR14136828_2.fastq.gz -o fastqc_reports/

trimmomatic PE -phred33 \
    ~/sra_data/SRR14136828_1.fastq.gz ~/sra_data/SRR14136828_2.fastq.gz \
    ~/sra_data/SRR14136828_1_paired.fastq.gz ~/sra_data/SRR14136828_1_unpaired.fastq.gz \
    ~/sra_data/SRR14136828_2_paired.fastq.gz ~/sra_data/SRR14136828_2_unpaired.fastq.gz \
    ILLUMINACLIP:/path/to/Trimmomatic/adapters/TruSeq3-PE.fa:2:30:10 MINLEN:36

fastqc ~/sra_data/SRR14136828_1_paired.fastq.gz ~/sra_data/SRR14136828_2_paired.fastq.gz -o fastqc_reports/

gunzip Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
hisat2-build Homo_sapiens.GRCh38.dna.primary_assembly.fa reference_genome_index

hisat2 -p 4 --dta -x reference_genome_index \
       -1 ~/sra_data/SRR14136828_1_paired.fastq.gz -2 ~/sra_data/SRR14136828_2_paired.fastq.gz \
       -S output.sam

samtools view -bS output.sam > output.bam
samtools sort output.bam -o output_sorted.bam
samtools index output_sorted.bam

featureCounts -T 4 -a annotation.gtf -o counts.txt output_sorted.bam

# Install MultiQC for summarizing QC reports
conda install -c bioconda multiqc
multiqc fastqc_reports/ -o multiqc_summary/
