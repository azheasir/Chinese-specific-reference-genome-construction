# Chinese specific reference genome construction
The following content presents the experimental data, process, and results of the article "Chinese specific reference gene construction based on large scale population genetic variations"

Please let us know, if you find mistakes or want your tool added.
#Get tools
Information how to install 'conda' and add the 'bioconda' channel is available on https://www.baidu.com/.
'''Bash
conda create --name sv python=3.6
source activate sv
conda install simuG== bwa==  samtools==  qualimap==  
'''
#Get data
1.Download variant fragments(Taking downloading SNV samples as an example)
'''Bash
wget -c http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/HGSVC2/release/v1.0/integrated_callset/freeze3.snv.alt.vcf.gz
'''
2.Extract Asian samples from the downloaded variant fragments file and add AC AN columns
'''Bash
bcftools view -s HG00512,HG00513,NA18939,HG00864,NA18534,HG01596 -o eastAisa.snv.alt.vcf freeze3.snv.alt.vcf
'''
