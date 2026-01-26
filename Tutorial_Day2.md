<!-- -- -- -- -- -- -- -- -- -- -- 
Modder: Hieu

Modifications:  

+ Clean up, correct, and rephrase explanations for clarity.  
+ Fix inaccurate or confusing sample code.  
+ Re-organize code explanations.
+ Add table of contents.

-- -- -- -- -- -- -- -- -- -- -- -->


<!-- Table of Contents GFM -->

* [From raw reads to MAGs](#from-raw-reads-to-mags)
    * [1. Aim](#1-aim)
    * [2. Tools used in tutorials Day2, Day3 and Day4](#2-tools-used-in-tutorials-day2-day3-and-day4)
    * [3. Bash script and the working system](#3-bash-script-and-the-working-system)
        * [3.1. Performing batch calculations](#31-performing-batch-calculations)
        * [3.2. Batch parameters](#32-batch-parameters)
        * [3.3. Special batch parameters](#33-special-batch-parameters)
    * [4. The dataset](#4-the-dataset)
    * [5. Preparation](#5-preparation)
* [Day 2 Workflow](#day-2-workflow)
    * [1. Quality control of raw reads](#1-quality-control-of-raw-reads)
    * [2. Assembly](#2-assembly)
    * [3. Questions](#3-questions)

<!-- /Table of Contents -->


# From raw reads to MAGs


## 1. Aim

In this tutorial, the aim is to learn how to assemble Metagenome Assembled Genomes (MAG) from raw reads. 
You will start by 
* Pre-processing the raw reads (trimming adapters, removing chimeras, removing phiX174 virus sequences…) 
* Assemble reads into contigs/fasta
* Assess quality of assemblies
* Bin contigs into MAGs
* Asses the completeness, contamination, and strain heterogeneity of your MAGs

**! We are not taking credit for the tools, we are simply explaining how they work for an in-house tutorial !** 


## 2. Tools used in tutorials Day2, Day3 and Day4

| Tool | Version | Repository |
| --- | --- | --- |
| fastqc | 0.12.1 | [FastQC](https://github.com/s-andrews/FastQC ) |
| fastp | 0.23.4 | [fastp](https://github.com/OpenGene/fastp ) |
| megahit | 1.2.9 | [megahit](https://github.com/voutcn/megahit ) |
| samtools | 1.19 | [samtools](https://github.com/samtools/samtools ) |
| QUAST | 5.2.0 | [quast](https://quast.sourceforge.net/quast ) |
| Bowtie2 | 2.4.5 | [bowtie2](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml ) |
| MetaBAT2 | 2.12.1 | [Metabat2](https://bitbucket.org/berkeleylab/metabat/src/master/ ) |
| DASTool | 1.1.5 | [DAS_Tool](https://github.com/cmks/DAS_Tool ) |
| anvi'o | 8 | [anvi'o](https://anvio.org/ ) |
| GUNC | 1.0.5 | [GUNC](https://grp-bork.embl-community.io/gunc/ ) |


## 3. Bash script and the working system

The CAU cluster is a Linux-based cluster that is designed for serial, moderately parallel, and high-memory computation.

### 3.1. Performing batch calculations

To run a **batch calculation**, it is not only important to instruct the batch system which program to execute, but also to specify the required resources, such as number of nodes, number of cores per node, main memory or computation time. These resource requests are written together with the program call into a so-called **batch** or **job script**, which is submitted to the batch system with the following command:  

```bash
sbatch <jobscript.sh>
```

An example script, `template.sh`, is provided in your `$WORK` directory.  

Note that every script starts with the directive `#!/bin/bash` on the first line to indicate which language/interpreter the system should use. The subsequent lines contain the directive `#SBATCH`, followed by a specific resource request or some other job information.  

After the lines with the SBATCH settings, pre-installed modules can be loaded (if required). A list of these available modules can be shown with `module av`. Next is the body of the job script, which calls up any commands and programs the user wants to run. Finally, to monitor the duration and resource usage of the script run, the `jobinfo` command is added at the end.
  
### 3.2. Batch parameters

The following table summarizes the most important job parameters. 

| Parameter | Explanation |
| --- | --- | 
| #SBATCH | Slurm batch script directive | 
| --partition=\<name\> or -p \<name\> | Slurm partition (~batch class) | 
| --job-name=\<name\> or -J \<jobname\> | Job name | 
| --output=\<filename\> or -o \<filename\> | Stdout file | 
| --error=\<filename\> or -e \<filename\> | Stderr file; if not specified, stderr is redirected to stdout file | 
| --nodes=\<nnodes\> or -N \<nnodes\> | Number of nodes | 
| --ntasks-per-node=\<ntasks\> | Number of tasks per node; number of MPI processes per node | 
| --cpus-per-task=\<ncpus\> or -c \<ncpus\> | Number of cores per task or process | 
| --mem=\<size[units]\> | Real memory required per node; the default unit is megabytes (M); use G for gigabytes | 
| --time=\<time\> or -t \<time\> | Walltime in the format "days-hours:minutes:seconds" | 
| --mail-user=\<email-address\> | Set email address for notifications |
| --mail-type=\<type\> | Type of email notification: BEGIN, END, FAIL or ALL |


### 3.3. Special batch parameters
  
An important job parameters specific for our course for dedicated resources.

```
#SBATCH --reservation=biol217
```
**$\color{red}ALWAYS\ USE\ IT\ FOR\ THE\ COURSE!$**
  
**OTHER IMPORTANT NOTES FOR OUR COURSE:** 

- **A BATCH SCRIPT SUBMISSION IS TO BE DONE FOR ALL DAYS, NOT JUST TODAY**

- **A BATCH SCRIPT SUBMISSION IS TO BE DONE WITH EVERY STEP**

    - **EXCEPT VISUALIZATION STEPS**


**FOR EVERY JOB SUBMISSION, YOU SHOULD CHANGE**
  
- **Job name** 
- **Stdout file**
- **Stderr file**

**to reflect the step you are doing and get a log file per command execution.**
  
**This will help with debugging and not overwriting output files.**

AFTER ALL PARAMETERS comes 
  
- **conda/micromamba activation**
- **commands and programs for executing a process in your pipeline (see below)**

After job submission, the batch server will evaluates the job script and searches for free, appropriate compute resources. If there are resources, the server will execute the actual computation. Otherwise, the job will be queued for later.

Successfully submitted jobs are managed by the batch system and can be displayed with the following commands:
  
```bash
squeue -u <username>
squeue --me
```
  
For showing individual jobs' details:
  
```bash
squeue -j <jobid>
scontrol show job <jobid>
```
  
To terminate a running job or to remove a queued job from the queue:

```bash
scancel <jobid>  
``` 

For more details, refer to the [documentation](https://www.rz.uni-kiel.de/en/our-portfolio/hiperf/caucluster?set_language=en) from the RZ of CAU KIEL.

Putting everything together, the batch script should start with something like this:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=fastqc
#SBATCH --output=fastqc.out
#SBATCH --error=fastqc.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load micromamba/1.4.2
cd $WORK
micromamba activate .micromamba/envs/00_anvio/
```


## 4. The dataset
 
The background concerning the dataset and analysis based on 16S amplicon sequence analysis was published by [Fischer et al. (2018)](https://sfamjournals.onlinelibrary.wiley.com/doi/full/10.1111/1751-7915.13313). Metagenomes from the same samples were sequenced.

In summary, the samples originated from one mesophilic agricultural biogas plant located near Cologne, Germany, sampled in more or less monthly intervals over a period of 587 days. As described by the authors, [NH4<sup>+</sup>–N] concentrations during the sampling period sometimes exceeded 3.0 g l<sup>-1</sup>, which was reported as a concentration that is beneficial for acetoclastic methanogenesis. In addition, the composition of the microbial community was heavily influenced by the temperature and pH of the bioreactor environment.

**Here we will focus on 3 samples only.**


## 5. Preparation
  
All packages and programs needed are already installed into one `conda`/`micromamba` environment. Activate this environment every time you open a terminal using the following command:

```bash
module load gcc12-env/12.1.0
module load micromamba/1.3.1
cd $WORK
micromamba activate .micromamba/envs/00_anvio/
``` 

Most programs you will be using don't have a graphical user interface but are run through a terminal.  

Running the command `<tool> --help` will display a detailed description of what the tool is for and what parameters it accepts. For example, if you are using `fastqc -o`  and you want to know what the flag `-o`  means, type `fastqc --help` and look for an explanation of `-o`.

**The most important things to look for in the help info of every tool are:**

- **what is the input?**
- **what is the output?** 
- **required parameters!**
- **optional parameters**
- **number of cpus/threads**

A note for pathfinders: Whenever you need to provide a file or directory as input to a command, you can write either the _relative_ or _absolute_ path to that file/directory. For example, suppose you have a directory structure like this:

```bash
analysis/
├── input/
│   └── reads.fastq.gz
└── output/
```

If you are in the `analysis/` directory and want to take a quick look at the sequencing reads file, either of the following commands will work:  

```bash
# Relative path (starts with . or ..)
head ./input/reads.fastq.gz

# Absolute path (starts with /)
head /work_beegfs/sunamNNN/analysis/input/reads.fastq.gz
```

Remember to always double check you commands and paths, such as with `tab` completion, `echo`-ing the full command before running it, or `readlink -e`.  
MAAAAAAAAAANY errors are a direct result of command **$\color{red}typos$** and wrong **$\color{red}paths\ !!!$**
  

---

**$$\color{red}DAY\ 2$$**


# Day 2 Workflow


## 1. Quality control of raw reads
  
First, we need to evaluate the quality of the sequenced data before proceeding with further analyses. You will use `FastQC` and `fastp` for this. 

[`FastQC`](https://github.com/s-andrews/FastQC ) provides an overview of basic quality control metrics like Phred scores, which gives you an idea of how accurate each base call was.

Before you start, create a folder to store your results.

You will run `fastqc` on several files named `*.fastq.gz`, which are compressed (zipped) sequencing reads files. 

```bash
fastqc ? -o ?
```

**$\color{red}SUBMIT\ ALL\ YOUR\ COMMANDS\ IN\ A\ BATCH\ SCRIPT!\ $**

<details><summary>$\color{yellow}Hint$</summary>  

> `fastqc file.fastq.gz` : run `fastq` on `file.fastq.gz`  
> `-o` : output folder  

</details>
  
<details><summary><b>Complete command</b></summary>

```bash
fastqc file.fastq.gz -o output_folder/ 
```

or in a loop:

```bash
for i in *.fastq.gz; do fastqc $i -o output_folder/; done
```
</details>  
  
  
[`fastp`](https://github.com/OpenGene/fastp ) allows you to process and filter the reads. As we have paired-end reads, we need to specify two different input files: forward (`R1`) and reverse (`R2`).

```bash
fastp -i ? -I ? -o ? -O ? -t 6 -q 20 -h ? -R ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `-i R1.fastq.gz` : input forward reads  
> `-I R2.fastq.gz` : input reverse reads  
> `-o output_folder/R1.fastq.gz` : output processed forward reads  
> `-O output_folder/R2.fastq.gz` : output processed reverse reads  
> `-t 6` : how many bases to trim from the tail of the forward reads  
> `-q 20` : Phred score threshold for filtering bases  
> `-h sample1.html` : name of the HTML-format report file  
> `-R "Sample 1"` : title for the output report file  

</details>

<details><summary><b>Complete command</b></summary>

```bash
fastp -i sample1_R1.fastq.gz -I sample1_R2.fastq.gz -o outdir/sample1_R1_clean.fastq.gz -O outdir/sample1_R2_clean.fastq.gz -t 6 -q 20 -h sample1.html -R "Sample 1 Fastp Report"
```

</details>


## 2. Assembly

Once the reads are filtered and cleaned by `fastp`, you can perform genome assemblies using [`megahit`](https://github.com/voutcn/megahit ), an ultra-fast and memory-efficient NGS assembler. It is optimized for metagenomes coassembly and multiple samples.  

$\color{red}Do\ not\ create\ the\ output\ folder\ beforehand\ !!$

```bash
megahit -1 ? -1 ? -1 ? -2 ? -2 ? -2 ? -o ? --min-contig-len 1000 --presets meta-large -m 0.85 -t 12        
```

<details><summary>$\color{yellow}Hint$</summary>

> `-1` : input forward reads  
> `-2` : input reverse reads  
> `-o` : output directory  

</details>

<details><summary><b>Complete command</b></summary>

```bash
megahit -1 sample1_R1_clean.fastq.gz -1 sample2_R1_clean.fastq.gz -1 sample3_R1_clean.fastq.gz -2 sample1_R2_clean.fastq.gz -2 sample2_R2_clean.fastq.gz -2 sample3_R2_clean.fastq.gz -o /PATH/TO/3_coassembly/ --min-contig-len 1000 --presets meta-large -m 0.85 -t 12   
```

</details>

During the assembly process, `megahit` tried out several k-mer sizes and wrote the best assembly to `final.contigs.fa`. All assemblies from different k-mer sizes are found in `outdir/intermediate_contigs/k{N}.contigs.fa`.  

To visualize the assembled contigs in `Bandage`, you need to convert the plain-text sequence file (`fasta`) into a fasta-like graph (`fastg`). To create a graph of the final assembly with k-mer size 99, use:  
  
```
megahit_toolkit contig2fastg 99 final.contigs.fa > final.contigs.fastg
```

Now, open `final.contigs.fastg` in `Bandage`. Once it is loaded (which might take a moment), click on `Draw graph` to visualize the contigs in the assembly.   
You can label the nodes by adding parameters like Depth or Name. Whenever you change something, you need to click `Draw graph` again.


## 3. Questions
  
* **Attach the figure you generated and explain briefly in your own words what you can see.**
