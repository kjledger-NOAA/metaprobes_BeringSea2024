# metaprobes_BeringSea2024

1. removed primers from demultiplexed amplicon reads using cutadapt 
- mkdir trimmed
- conda activate cutadptenv
- DATA=/Users/kimberly.ledger/Documents/metaprobes_BeringSea2024/fastq
- NAMELIST=$(ls ${DATA} | sed 's/e*_L001.*//' | uniq)
- echo "${NAMELIST}"
- for i in ${NAMELIST}; do
   cutadapt --discard-untrimmed -g GCCGGTAAAACTCGTGCCAGC -G CATAGTGGGGTATCTAATCCCAGTTTG -o trimmed/${i}_R1.fastq.gz -p trimmed/${i}_R2.fastq.gz "$DATA/${i}_L001_R1_001.fastq.gz" "$DATA/${i}_L001_R2_001.fastq.gz";
done
- pigz -d trimmed/*.gz
- cd trimmed 
- mkdir filtered
- mkdir filtered/outputs

2. process reads using '1_sequence_filtering.Rmd'

3. perform taxonomic assignment using MIDORI2 database 
makeblastdb -in MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST.fasta \
            -dbtype nucl \
            -out MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db

blastn -query fastq/trimmed/filtered/outputs/myasvs.fasta \
       -db MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db \
       -out fastq/trimmed/filtered/outputs/blastnresults \
       -perc_identity 96 -qcov_hsp_perc 100 \
       -outfmt "6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore sscinames staxids" \
       -num_threads 10
       
4. filter blastn results to finalize taxonomic assignments using 
       