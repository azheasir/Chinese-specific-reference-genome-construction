# Chinese specific reference genome construction
The following content presents the experimental data and process of the article "Chinese specific reference gene construction based on large scale population genetic variations"

Please let us know, if you find mistakes or want your tool added.
# Get tools
Information how to install 'conda' and add the 'bioconda' channel is available on https://bioconda.github.io/.
```Bash
conda create --name gene python=3.8
source activate gene
conda install bwa==0.7.17  samtools==1.6  qualimap
git clone https://github.com/yjx1217/simuG.git 
```
# Get data
1.Download variant fragments(Taking downloading SNV samples as an example):
```Bash
wget -c http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/HGSVC2/release/v1.0/integrated_callset/freeze3.snv.alt.vcf.gz
```
2.Extract Asian samples from the downloaded variant fragments file and add AC AN columns:
```Bash
bcftools view -s HG00512,HG00513,NA18939,HG00864,NA18534,HG01596 -o eastAisa.snv.alt.vcf freeze3.snv.alt.vcf
```
3.Download Grch38 reference genome:
```Bash
wget -c https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000001405.26/GCF_000001405.26_GRCh38_genomic.fna.gz
```
4.Download genomic samples from the three ethnic groups of Han in the north, Han in the south, and Dai(NA18525、NA18644、NA18757、NA18747、NA18561；HG00458、HG00717、HG00476、HG00534、HG00716；HG00759、HG01799、HG01028、HG02389、HG01812,Taking downloading NA18525 as an example)):
```Bash
wget -c 	ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR741/SRR741372/SRR741372_1.fastq.gz
wget -c 	ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR741/SRR741372/SRR741372_2.fastq.gz
wget -c   ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR741/SRR741373/SRR741373_1.fastq.gz
wget -c   ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR741/SRR741373/SRR741373_2.fastq.gz
```
# Experiment process
1.Using AC/AN to calculate AF, and then classifying the original East Asian variation fragments using different gradients of AF values（Taking the indel group with AF ≥ 0 as an example）:
```python
import vcf
vcf_reader = vcf.Reader(open('D:\基因\eastAisa.indel.alt.vcf','r'))
vcf_writer = vcf.Writer(open('D:\基因\eastAisa1.indel.alt.vcf','w'),vcf_reader)
e=0
g=0
d=0
r=0.0000
for record in vcf_reader:
    a=record.INFO['AC']
    b=record.INFO['AN']
    c=a[0]
    f=record.POS
    if b!=0:
            d=c/b
            if d>=r:   vcf_writer.write_record(record)
```
2.Using SimuG for Reference Genome Correction and Generation of New Reference Genomes(Using SNV to modify grch38）:
```Bash
perl simuG.pl -refseq GCF_000001405.26_GRCh38_genomic.fna.gz -snp_vcf eastAisa1.snv.alt.vcf -prefix output_prefixsnv1
```
（Using SV/INDEL to modify grch38）:
```Bash
perl simuG.pl -refseq GCF_000001405.26_GRCh38_genomic.fna.gz -indel_vcf eastAisa1.indel.alt.vcf -prefix output_prefixindel1
```
3.Comparison of Chinese sequencing data with the newly constructed reference genome and the original GRCh38 reference genome:
```Bash
bwa index output_prefixsnv1.simseq.genome.fa
bwa mem -t 16 output_prefixsnv1.simseq.genome.fa SRR792936_1.fastq.gz SRR792936_2.fastq.gz -o outputbh1snv11.sam
bwa mem -t 16 output_prefixsnv1.simseq.genome.fa SRR792963_1.fastq.gz SRR792963_2.fastq.gz -o outputbh1snv12.sam
samtools merge -@16 outputbh1snv1.bam outputbh1snv11.sam outputbh1snv12.sam
samtools sort -@16 outputbh1snv1.bam > outputbh1snv1.sort.bam
bwa index outputbh1snv1.sort.bam
qualimap bamqc -nt 16 -bam --java-mem-size=24G outputbh1snv1.sort.bam  -outdir outputbh1snv1
```
