###
$${\color{Green}DAY\ 8}$$
### 
# RNA-Seq analysis (Transcriptomics) - Tutorial

### **`Aim`**
The aim of this tutorial is to train you in:
- Setting up an RNA-Seq data analysis pipeline
- RNA-Seq data pre-processing
- RNA-Seq data analysis
- Differential gene expression analysis
- Data visualization
<br>
<br>

You will:
- Download the required files
- Set up the files in specific locations
- Check read quality
- Align the reads to a reference sequence
- Calculate the coverage
- Perform gene wise quantification
- Calculate differential gene expression

### **`Tools used`**
| Tool        | Version | Repository                                                                        |
|-------------|---------|-----------------------------------------------------------------------------------|
| READemption |  2.0.4  | [link](https://reademption.readthedocs.io/en/latest/) |
| DESeq2      |   1.50.2   | [link](http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html) |
| edgeR       |   3.40  | [link](https://bioconductor.org/packages/release/bioc/vignettes/edgeR/inst/doc/edgeRUsersGuide.pdf) |
| limma       |   3.54  | [link](https://bioconductor.org/packages/release/bioc/vignettes/limma/inst/doc/usersguide.pdf) |
| R           |   4.5.2 | [link](https://cran.r-project.org/) |

---

### 
$${\color{Green}Example\ 1}$$
### 

---

## Aim

In this first example you are given the complete script to run an RNA-Seq analysis using `READemption`. We will go through the script step by step to understand what each command does and then move on to example 2 where you will run an analysis on your own.

## The dataset
The dataset for this example comes from a publication by [*Kröger et al. 2013*](https://doi.org/10.1016/j.chom.2013.11.010).

The aim of the study was to *"`present a simplified approach for global promoter identification in bacteria using RNA-seq-based transcriptomic analyses of 22 distinct infection-relevant environmental conditions. Individual RNA samples were combined to identify most of the 3,838 Salmonella enterica serovar Typhimurium promoters`"*.

Here you will use a subset of the original dataset including two replicates from two conditions:

**`InSPI2`** describes an acidic phosphate-limiting minimal medium that induces *Salmonella* pathogenicity island (SPI) 2 transcription.
This is suspected to create an environmental shock that could induce the upregulation of
specific gene sets.

**`LSP`** (late stationary phase). This is reached by growth in Lennox Broth medium.

**This is the complete script for an example analysis.**

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=32G
#SBATCH --time=2:00:00
#SBATCH --job-name=reademption
#SBATCH --output=reademption.out
#SBATCH --error=reademption.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
micromamba activate $WORK/.micromamba/envs/08_reademption

# set proxy environment to download the data and use the internet in the backend
export http_proxy=http://relay:3128
export https_proxy=http://relay:3128
export ftp_proxy=http://relay:3128

# create folders
reademption create --project_path READemption_analysis_1 --species salmonella="Salmonella Typhimurium"

# download reference genome sequence files (genome and 3 plasmids)
FTP_SOURCE=ftp://ftp.ncbi.nih.gov/genomes/archive/old_refseq/Bacteria/Salmonella_enterica_serovar_Typhimurium_SL1344_uid86645/
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_016810.fa $FTP_SOURCE/NC_016810.fna
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017718.fa $FTP_SOURCE/NC_017718.fna
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017719.fa $FTP_SOURCE/NC_017719.fna
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017720.fa $FTP_SOURCE/NC_017720.fna

# rename the files similar to the genome naming
sed -i "s/>/>NC_016810.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_016810.fa
sed -i "s/>/>NC_017718.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_017718.fa
sed -i "s/>/>NC_017719.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_017719.fa
sed -i "s/>/>NC_017720.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_017720.fa

# download gene annotations and unzip it
wget -O READemption_analysis_1/input/salmonella_annotations/GCF_000210855.2_ASM21085v2_genomic.gff.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/210/855/GCF_000210855.2_ASM21085v2/GCF_000210855.2_ASM21085v2_genomic.gff.gz
gunzip READemption_analysis_1/input/salmonella_annotations/GCF_000210855.2_ASM21085v2_genomic.gff.gz

# download RNA-seq reads
wget -O READemption_analysis_1/input/reads/InSPI2_R1.fa.bz2 http://reademptiondata.imib-zinf.net/InSPI2_R1.fa.bz2
wget -O READemption_analysis_1/input/reads/InSPI2_R2.fa.bz2 http://reademptiondata.imib-zinf.net/InSPI2_R2.fa.bz2
wget -O READemption_analysis_1/input/reads/LSP_R1.fa.bz2 http://reademptiondata.imib-zinf.net/LSP_R1.fa.bz2
wget -O READemption_analysis_1/input/reads/LSP_R2.fa.bz2 http://reademptiondata.imib-zinf.net/LSP_R2.fa.bz2

# align reads to reference
reademption align -p 4 --poly_a_clipping --project_path READemption_analysis_1

# calculate read coverage
reademption coverage -p 4 --project_path READemption_analysis_1

# quantify gene expression
reademption gene_quanti -p 4 --features CDS,tRNA,rRNA --project_path READemption_analysis_1

# calculate differential expression using DESeq2
reademption deseq -l InSPI2_R1.fa.bz2,InSPI2_R2.fa.bz2,LSP_R1.fa.bz2,LSP_R2.fa.bz2 -c InSPI2,InSPI2,LSP,LSP -r 1,2,1,2 --libs_by_species salmonella=InSPI2_R1,InSPI2_R2,LSP_R1,LSP_R2 --project_path READemption_analysis_1

# visualization
reademption viz_align --project_path READemption_analysis_1
reademption viz_gene_quanti --project_path READemption_analysis_1
reademption viz_deseq --project_path READemption_analysis_1

# environment cleanup
micromamba deactivate
module purge
jobinfo
```
---
$${\color{Green}Example\ 2}$$
---

In this example you will run the analysis yourself.

The dataset you will use comes from a publication by [*Prasse et al. 2017*](https://doi.org/10.1080/15476286.2017.1306170).

## 1. Download the sequence data you want to analyze

> 1. Open the paper and find the NCBI accession number for the sequence data (method section).
> 2. Go to the NCBI website and search for the accession number.
> 3. Find the `SRR numbers` of the data you want to download and run the following commands:

For downloading the data we will use `grabseqs`, a tool that allows downloading of sequence data from various repositories including SRA.

```bash
#use micromamba to activate grabseq
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
micromamba activate $WORK/.micromamba/envs/10_grabseqs
```

Create a new directory called 'fastq_raw' and navigate to it. Download the data using the SRR numbers you found (add them in place of the ***):

```bash
grabseqs sra -t 4 -m ./metadata.csv SRR***
```
<details><summary><b>Click here to see the code</b></summary>

Create and navigate to new folder:
```bash
mkdir fastq_raw
cd fastq_raw
```
Download the data specifying the SRR numbers:

```bash
grabseqs sra -t 4 -m ./metadata.csv SRR4018514
grabseqs sra -t 4 -m ./metadata.csv SRR4018515
grabseqs sra -t 4 -m ./metadata.csv SRR4018516
grabseqs sra -t 4 -m ./metadata.csv SRR4018517
```
</details>

### > **Note:** **Rename each SRR** file according to the sample name. For example, SRR4018514 to `wt_R1.fastq.gz`, SRR4018515 to `wt_R2.fastq.gz`, SRR4018516 to `mut_R1.fastq.gz`, and SRR4018517 to `mut_R2.fastq.gz`.

## 2. Create the reademption folder structure

Activate the reademption environment and create the folder structure for the analysis.

<details><summary><b>Click here to see the code</b></summary>

```bash
# Activate the environment:
module load gcc12-env/12.1.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
micromamba activate $WORK/.micromamba/envs/08_reademption

# go to the directory you want to work in
cd $WORK/rnaseq

# create folders
reademption create --project_path READemption_analysis_2 --species methanosarcina="Methanosarcina mazei Gö1"
```

</details>

## 3. Download the reference genome and annotation files and move them to the input folder

Go to https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000007065/ and download both the genome sequence (fna.gz file) and the genome annotation (gff.gz file) using FTP. Remember to download them into the correct input folders and to unzip them.

<details><summary><b>Click here to see the code</b></summary>

```bash
# download reference genome
wget -O READemption_analysis_2/input/methanosarcina_reference_sequences/GCF_000007065.1_ASM706v1_genomic.fna.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/065/GCF_000007065.1_ASM706v1/GCF_000007065.1_ASM706v1_genomic.fna.gz

# download annotation
wget -O READemption_analysis_2/input/methanosarcina_annotations/GCF_000007065.1_ASM706v1_genomic.gff.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/065/GCF_000007065.1_ASM706v1/GCF_000007065.1_ASM706v1_genomic.gff.gz

# unzip them
gunzip READemption_analysis_2/input/methanosarcina_reference_sequences/GCF_000007065.1_ASM706v1_genomic.fna.gz
gunzip READemption_analysis_2/input/methanosarcina_annotations/GCF_000007065.1_ASM706v1_genomic.gff.gz
```
</details>

## 4. Copy the raw_reads to the READemption_analysis/input/reads folder

The reads you downloaded earlier into the fastq_raw folder can now be moved into the correct input folder. They don't need to be unzipped here.

## 5. Run READemption

Use the code from example 1 as guidance to write a bash-script and run the analysis. Bear in mind that in this example the reads are stored as `FASTQ` files, which is important for the `reademption align` command. Check the [docs](https://reademption.readthedocs.io/en/latest/subcommands.html) for help.

<details><summary><b>Click here to see the code</b></summary>

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=64G
#SBATCH --time=0-04:00:00
#SBATCH --job-name=rna_seq_methanosarcina
#SBATCH --output=rna_seq_methanosarcina.out
#SBATCH --error=rna_seq_methanosarcina.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
micromamba activate $WORK/.micromamba/envs/08_reademption

# align reads to reference
reademption align -p 4 --poly_a_clipping --project_path READemption_analysis_2 --fastq

# calculate read coverage
reademption coverage -p 4 --project_path READemption_analysis_2

# quantify gene expression
reademption gene_quanti -p 4 --features CDS,tRNA,rRNA --project_path READemption_analysis_2

# calculate differential expression using DESeq2
reademption deseq -l mut_R1.fastq.gz,mut_R2.fastq.gz,wt_R1.fastq.gz,wt_R2.fastq.gz -c mut,mut,wt,wt -r 1,2,1,2 --libs_by_species methanosarcina=mut_R1,mut_R2,wt_R1,wt_R2 --project_path READemption_analysis_2

# visualization
reademption viz_align --project_path READemption_analysis_2
reademption viz_gene_quanti --project_path READemption_analysis_2
reademption viz_deseq --project_path READemption_analysis_2

# environment cleanup
micromamba deactivate
module purge
jobinfo
```
</details>

## 6. Analyze your results

The script produced a lot of output files and folders will be created in the `READemption_analysis_2` output folder. Look at the visualizations results and interpret them yourself.

>`all_species_viz_align`:
<details><summary><b>Show interpretation</b></summary>
The <b>stacked species box plot</b> shows how many reads mapped to which species (or didn't map at all). In our case, all mapped reads mapped to our single reference genome of <i>Methanosarcina mazei</i> Gö1.
</details>
<br>

>`methanosarcina_viz_align`: 
<details><summary><b>Show interpretation</b></summary>
The <b>aligned reads box plot</b> shows the number of aligned reads and how they aligned. Uniquely aligned reads were mapped to only one location in the genome, while multiple aligned reads mapped to several locations. Here we can see that most reads aligned uniquely. Split aligned reads are reads that span across multiple separate regions of the genome, which can happen in eukaryotic genomes with introns, but is less common in prokaryotes. Cross aligned reads align to multiple references genomes, which is not relevant here since we only have one.
</details>
<br>

>`methanosarcina_viz_deseq`:
<details><summary><b>Show interpretation</b></summary>
The <b>MA plots</b> visualize the differential expression analysis. Each point represents a gene, with the x-axis showing the average expression level and the y-axis showing the log2 fold change between conditions. Genes that are <i>significantly</i> differentially expressed are shown as red dots, <i>non-significant</i> as black dots. A log2 fold-change of above 0 means that the gene is upregulated in the first condition compared to the second and vice-versa. In general, we expect a mostly symmetric distribution around log2 fold-change = 0, with some genes showing significant up- or down-regulation. 
<br>
<br>
The <b>volcano plots</b> also visualize the differential expression analysis. There are plots for both the raw p-values and the adjusted p-values. The adjusted p-value is more conservative than the raw p-value and therefore shows less false positives. In both cases the plot shows the log2 fold-changes on the x-axis, positive values indicate upregulation in the first condition compared to the second and vice-versa. The y-axis shows the -log10 of the p-value, meaning that the higher up a point is, the lower is its p-value. The green dotted lines indicate the thresholds for significance (p-value < 0.05) and log2 fold-change (> 1 or < -1).
</details>
<br>

>`methanosarciana_viz_gene_quanti`:
<details><summary><b>Show interpretation</b></summary>
The <b>expression scatter plots</b> compare the expression of each gene in different conditions. Generally, we expect most genes to express similarly in all conditions. In that case most points should fall along the diagonal line and the R-value should be close to 1.
<br>
<br>
The <b>RNA class box plots</b> show the number of read counts for different RNA classes (CDS, tRNA, rRNA) in each sample. This can help identify any biases in the data, for example if one RNA class is overrepresented in a sample.
</details>
<br>

>`read_lengths_viz_align`:
<details><summary><b>Show interpretation</b></summary>
The <b>read length distribution plots</b> compare the differences in read lengths before and after trimming and between samples. The read length is determined by the sequencing technology used.
</details>

## 7. What now?
The most important files for further analysis lie in the `methanosarcina_deseq` folder. Here you can find the tables that show the results of the differential expression analysis (log2 FC, p-values etc.), for both the raw data (deseq_raw) and with annotations (deseq_with_annotations). The annotations are crucial, because without them we don't know which genes are actually differentially expressed. 

Think about what you could do with these results.

---



