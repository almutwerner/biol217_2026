<!-- - - - - - - - - - - - - - - - 
Modder: Hieu
Date: 2026.01.28

Modifications:  

- Rearrange the sections into a more logical order.  
- Remove redundant content (assembly from day 2).    
- Move bin quality evaluation to day 4.  
- Rephrase explanations for clarity.  
- Fix inaccurate or confusing sample code.  
- Re-organize code explanations.  
- Explain command options in more details.  
- Add table of contents.  

To add:  
- Note on command run time & verifying output integrity before going for break.  
- Clarification on bowtie2 prefix naming scheme.  
- Talk about potential "Warning" from anvio profile db

- - - - - - - - - - - - - - - - -->


$${\color{red}DAY 3}$$


<!-- Table of Contents GFM -->

* [Aim](#aim)
* [1. Evaluating Assembly Quality](#1-evaluating-assembly-quality)
* [Questions](#questions)
* [2. Mapping Sequencing Reads to Assembled Contigs](#2-mapping-sequencing-reads-to-assembled-contigs)
    * [2.1. Re-formatting the contigs](#21-re-formatting-the-contigs)
    * [2.2. Indexing the contigs](#22-indexing-the-contigs)
    * [2.3. Mapping reads onto contigs](#23-mapping-reads-onto-contigs)
    * [2.4. Sorting mapped reads](#24-sorting-mapped-reads)
* [3. Binning reads](#3-binning-reads)
    * [3.1. Generating contigs database](#31-generating-contigs-database)
    * [3.2. Annotating CDSs](#32-annotating-cdss)
    * [3.3. Visualizing the contigs database](#33-visualizing-the-contigs-database)
    * [3.4. Creating an `anvi'o` profile](#34-creating-an-anvio-profile)
    * [3.5. Merging `anvi'o` profiles from all samples](#35-merging-anvio-profiles-from-all-samples)
    * [3.6. Binning contigs into genomes](#36-binning-contigs-into-genomes)
        * [Using `MetaBAT2`](#using-metabat2)
        * [Using `MaxBin2`](#using-maxbin2)
* [Questions](#questions-1)

<!-- /Table of Contents -->


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## Aim

On day 2, you started with raw metagenomic sequencing reads. You checked the quality of the reads, cleaned them, and assembled them into contigs. You then looked at a graph of all the assembled contigs and wondered, why are these contigs so small and fragmented? Do any of these contigs belong to the same species? How many species are there in the samples? And just what kinds of organisms are they?  

You might have also started thinking about when it is better to pool the reads from all samples into one co-assembly, versus assembling each sample separately.  

Today, you will get to check in more details how well the assembly went, and begin to figure out who are present in the samples. You will achieve this through _**binning**_.  

In metagenomic binning, we group together assembled contigs that likely belong to the same genome to get a more complete and informative genome. The grouping is based on various metrics like tetranucleotide frequency, differential coverage, and genome completion.  


Most of the tools in this tutorial come from [`anvi'o`](https://anvio.org/), an **AN**alysis and **VI**sualization platform for microbial **'O**mics. It is VERY extensive, contains a huge collection of tools, is highly integrated, super easy to use, and interactive. You can check out its documentations ([1](https://anvio.org/help/main/)), tutorials ([2](https://anvio.org/learn/), [3](https://merenlab.org/2016/06/22/anvio-tutorial-v2/)) and other [resources](https://anvio.org/blog/).    

The workflow will be as follows:  

1. Check assembly quality  
2. Map sequencing reads to contigs  
3. Bin contigs into genomes (MAGs) based on read mapping  
4. Estimate the quality of binned MAGs  

Again, $\color{red}RUN\ EVERYTHING\ IN\ A\ BATCH\ SCRIPT$, unless otherwise noted.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 1. Evaluating Assembly Quality  

[`quast`](https://quast.sourceforge.net/quast) is a very informative **QU**ality **AS**sessment **T**ool to evaluate genome assemblies. See its [manual](https://quast.sourceforge.net/docs/manual.html) for more details on command-line options.  

```bash
metaquast ? -o ? -t 6 -m 1000
```

<details><summary>$\color{yellow}Hint$</summary>

> `metaquast <file>` : Run quality check on assembly file `contigs.fa`.  
> `-o <dir>` : Path to output directory.  
> `-t 6` : Maximum number of CPU threads to use for calculations.  
> `-m 1000` : Minimum contig length to include in calculations. Any contigs shorter than this will be ignored.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
metaquast /path/to/final.contigs.fa -o /path/to/metaquast_out/ -t 6 -m 1000
```
</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## Questions

* **What is your N50 value? Why is this value relevant?**
* **How many contigs are assembled?**
* **What is the total length of the contigs?**


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 2. Mapping Sequencing Reads to Assembled Contigs  

Read mapping results allow us to check for contamination, classify taxa, identify incorrect binning, and more.  

If you simply use the `final.contigs.fa` file for read mapping, you will run into issues such as:  

+ The contig names (sequence IDs) are too complicated and may contain special characters.  
+ Many contigs are too small for the mapping to be meaningful.  
+ The contigs and nucleotide positions are random and not organized in a systematic manner.  
+ The fasta format usually cannot store many types of relevant information, such as:  
    + Where the genes are  
    + Tetra-nucleotide frequencies  
    + Which species a contig may belong to  
    + Detailed information on individual nucleotides  

Therefore, you need to clean up the assembly and convert it into a more efficient format before you can map the reads onto it.  


### 2.1. Re-formatting the contigs  

To simplify sequence IDs and filter out short contigs, we will use [`anvi-script-reformat-fasta`](https://anvio.org/help/9/programs/anvi-script-reformat-fasta/).  

```bash
anvi-script-reformat-fasta ? -o ? --min-len 1000 --simplify-names --report-file ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `anvi-script-reformat-fasta <fasta>` : Run the program on the file `<fasta>`.  
> `-o <fasta>` : Write output (re-formatted fasta) to file `<fasta>`.  
> `--min-len 1000` : Remove contigs that are shorter than 1000 nucleotides.  
> `--simplify-names` : Make sequence IDs simpler and easier to work with.  
> `--report-file <table>` : Write the old and new sequence IDs to a table called `<table>`.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-script-reformat-fasta /path/to/final.contigs.fa -o /path/to/contigs.anvio.fa --min-len 1000 --simplify-names --report-file /path/to/name_conversion.txt
```

</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 2.2. Indexing the contigs  

If your contigs are organized and indexed, the mapping will run much faster. To index the re-formatted reads, we will use a program from the [`bowtie2`](https://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) suite.  

```
bowtie2-build ? ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `bowtie2-build <contigs> <index>`  
> `<contigs>` : Input re-formatted contigs fasta file to be indexed.  
> `<index>` : Output index file.  

</details>


<details><summary><b>Complete command</b></summary>

```
bowtie2-build /path/to/contigs.anvio.fa /path/to/contigs.anvio.fa.index
```

</details>

**Note:** `bowtie2-build` will split the index into multiple files like `*.1.bt2`, `*.2.bt2`, `*.3.bt2`, and so on. **Do not touch them.**  
$\color{red}Only\ work\ with$ `contigs.anvio.fa.index`.   


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 2.3. Mapping reads onto contigs  

We now use the main [`bowtie2`](https://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) program for the actual mapping. The output of read mapping is called a **S**equence **A**lignment **M**ap file, and ends with `*.sam`.  

$\color{yellow}Tip:$  

> Should you use raw reads or cleaned reads for mapping?

You should run one `bowtie2` command for each sample. For each pair of reads, the command is:  

```bash
bowtie2 -1 ? -2 ? -S ? -x ? --very-fast 
```

<details><summary>$\color{yellow}Hint$</summary>

> `-1 <forward>` : Input clean forward reads.  
> `-2 <reverse>` : Input clean reverse reads.  
> `-x <index>` : Contig index file.  
> `-S <sam>` : Output `.sam` file.  
> `--very-fast` : Run mode. This mode is, obviously, _very fast_, but also less accurate.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
bowtie2 -1 /path/to/sample1_R1_clean.fastq.gz -2 /path/to/sample1_R2_clean.fastq.gz -x /path/to/contigs.anvio.fa.index -S /path/to/sample1.sam --very-fast
```

</details>

If you are familiar with [string manipulation](https://tldp.org/LDP/abs/html/string-manipulation.html) in `bash`, you can try running a `for` loop.  

<details><summary><b>Advanced version</b></summary>

```bash
for forward in /path/to/*_R1_clean.fastq.gz; do
    reverse="${forward/R1/R2}"
    sample="${forward%%_R1_clean*}"
    bowtie2 -1 $forward -2 $reverse -x /path/to/contigs.anvio.fa.index -S /path/to/${sample}.sam --very-fast 
done
```

</details>

In the end, there should be 3 `.sam` files (one for each sample).  

If you look at the `*.sam` files (e.g., `head`, `less`, `cat`), you will see that you can read them, which means a computer will have to translate them to machine language every time it reads the file. We don't really need to read these read mapping files directly, so it is better to convert them into machine language (binary). This will make data processing go much faster.  

To conver **S**equence **A**lignment **M**ap (`.sam`) to **B**inary **A**lignment **M**ap (`.bam`), we can use [`samtools`](https://github.com/samtools/samtools). Detailed command-line parameters are found in its [documentation](https://www.htslib.org/doc/samtools.html).   

```bash
samtools view -Sb ? > ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `<sam> > <bam>`  
> `-S` : Select `.sam` as the input format (only for old versions of `samtools`).  
> `-b` : Select `.bam` as the output format.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
samtools view -Sb /path/to/sample1.sam > /path/to/sample2.bam
```

</details>

<details><summary><b>Advanced version</b></summary>

```bash
for sam_file in /path/to/*.sam; do samtools view -Sb $sam_file > "${sam_file/.sam/.bam}"; done
```
</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 2.4. Sorting mapped reads  

Sorting mapped reads not only speeds up data processing but also allows you to do downstream analyses such as visualization and variant calling (more info [here](https://samtools.org/which-samtools-command-is-used-to-sort-a-bam-file/)).  

Here, we will use [`anvi-init-bam`](https://anvio.org/help/main/programs/anvi-init-bam/), which not only _**sorts**_ but also _**indexes**_ the `.bam` files (in just one command).  

```bash
anvi-init-bam ? -o ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `<unsorted> -o <sorted>`  
> `<unsorted>` : Input unsorted `.bam` file.  
> `<sorted>` : Output sorted `.bam` file.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-init-bam /path/to/sample1.bam -o /path/to/sample1_sorted.bam
```

</details>

<details><summary><b>Advanced version</b></summary>

```
for unsorted in /path/to/*sample.bam; do anvi-init-bam /path/to/$unsorted -o /path/to/"${unsorted/.bam/_sorted.bam}"; done
```

**Important:** A common pitfall is to provide `*.bam` in the `for` loop. Remember that `*.bam` includes `*_sorted.bam`.  

</details>

**Note:** The command will automatically generate `*.bam.bai` files. They are the index files. **Do not touch them.**  
$\color{red}Only\ work\ with$ the $\color{red}sorted$ `.bam`.   


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 3. Binning reads  

With the results from read mapping, we are ready to bin our contigs into individual genomes (MAGs) and start to figure out which microbes are present in the samples.  

### 3.1. Generating contigs database  

As mentioned above, the `fasta` format of the contigs is limited and cannot store more complex types of information. To convert the `fasta` format into a more powerful type, we will use [`anvi-gen-contigs-database`](https://anvio.org/help/main/programs/anvi-gen-contigs-database/). With this, the simple `fasta` becomes a database that can store many types of information that are of interest to us, such the taxonomy of the genomes in the bins.  

`anvi-gen-contigs-database` will also:  

+ Compute k-mer frequencies for each contig.  
+ Soft-split contigs longer than 20,000 nucleotides into smaller ones.  
+ Identify open reading frames (ORFs) using [`Prodigal`](https://github.com/hyattpd/Prodigal) (a program for predicting genes in bacteria and Archaea).  

```bash
anvi-gen-contigs-database -f ? -o ? -n ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `-f <fasta>` : input _re-formatted_ fasta file.  
> `-o <contigs>` : output contigs database file.  
> `-n <name>` : name of the work project.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-gen-contigs-database -f /path/to/contigs.anvio.fa -o /path/to/contigs.db -n biol217
```

</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 3.2. Annotating CDSs  

It is also a good idea to search for any potential biological functions that the predicted ORFs may have. This information can come in handy when you want to study the metabolism of the species in the experiment (for example). We will do that with [`anvi-run-hmms`](https://anvio.org/help/main/programs/anvi-run-hmms/J). It will perform an HMM search against several collections of genes to see if the ORFs predicted by `anvi-gen-contigs-database` are similar to any known genes.  

```
anvi-run-hmms -c ? --num-threads 4
```

<details><summary>$\color{yellow}Hint$</summary>

> `-c <contigs>` : input contigs database.  
> `--num-threads 4` : the number of CPU threads to use for computing.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-run-hmms -c /path/to/contigs.db --num-threads 4
```

</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 3.3. Visualizing the contigs database  

Once your contigs database is built and annotated, you can examine it to see what types of information it is carrying.  

**Note:** This is an $\color{red}INTERACTIVE$ step. Follow the instructions in the `README.md` file $\color{red}!!!$  

```bash
anvi-display-contigs-stats /path/to/contigs.db

```


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 3.4. Creating an `anvi'o` profile  

An `anvi'o` profile is like an upgraded `anvi'o` database that can also store read mapping results and detailed per-nucleotide information.  
For each sample, run the command as follows:  

```bash
anvi-profile -i ? -c ? --output-dir ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `-i <bam>` : Input sorted and indexed `.bam` file.  
> `-c <contigs>` : Contigs database file.  
> `-o <dir>` : Output directory to store the profile and log files.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-profile -i /path/to/sample1_sorted.bam -c /path/to/contigs.db -o /path/to/sample1_profile/
```

</details>

<details><summary><b>Advanced version</b></summary>

```bash
for bam in /path/to/*_sorted.bam; do
    sample="${bam%%_sorted.bam}"
    anvi-profile -i $bam -c /path/to/contigs.db -o /path/to/${sample}/
done

```

</details>

`anvi-profile` will:  

+ Process each contig that is longer than **2,500 nucleotides** by default.  
    + Remember that the minimum contig length should be long enough for tetra-nucleotide frequencies to have enough meaningful signal. Never go below 1,000 nts.  
+ Calculate mean coverage, standard deviation of coverage, and the average coverage for the inner quartiles (Q1 and Q3) for each contig.  
+ Characterize single-nucleotide variants (SNVs) for every nucleotide position.  
    + By default, the profiler will not pay attention to any nucleotide position with less than 10X coverage.  
+ **Not** cluster contigs by default  
    + because single profiles are rarely used for genome binning or visualization  
    + and because the clustering step increases the profiling runtime for no good reason.  

In the output folder that you typed into the `anvi-profile` command, you will find the following files:  

> `RUNLOG.txt` : detailed log for the profiling run.  
> `PROFILE.db` : The desired `anvi'o` profile database that contains key information about the mapping of short reads from multiple samples onto the assembled contigs.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 3.5. Merging `anvi'o` profiles from all samples  

To analyze and compare all samples together, you will need to merge the profiles coming from your different samples into one profile:  

```bash
anvi-merge ? ? ? -o ? -c ? --enforce-hierarchical-clustering
```

<details><summary>$\color{yellow}Hint$</summary>

> `anvi-merge <profile1> <profile2> <profile3>`  
> `-o <dir>` : Output directory to store the merged profile.  
> `-c <contigs>` : Contigs database file.  
> `--enforce-hierarchical-clustering` : Construct a phylogenetic tree that shows the relationships between the contigs.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-merge /path/to/sample1/PROFILE.db /path/to/sample2/PROFILE.db /path/to/sample3/PROFILE.db -o /path/to/merged_profiles/ -c /path/to/contigs.db --enforce-hierarchical-clustering
```
</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 3.6. Binning contigs into genomes  

Now, everything is ready for genome _**binning**_. You will try and compare two programs: [`MetaBAT2`](https://bitbucket.org/berkeleylab/metabat/src/master/ ) and [`MaxBin2`](https://sourceforge.net/projects/maxbin2/).  

After binning, look at the `.html` report in the output folder (`/path/to/SUMMARY_METABAT2/index.html`).  

#### Using `MetaBAT2`  

The binning is done as follows:  

```bash
anvi-cluster-contigs -p ? -c ? -C METABAT2 --driver metabat2 --log-file ? --just-do-it
anvi-summarize -p ? -c ? -o ? -C METABAT2
```

_**Note**_: The bins will be stored _**within your merged**_ profile and not as a new file. (Remember how an `anvi'o` profile can store many types of complex data.)  

<details><summary>$\color{yellow}Hint$</summary>

> `anvi-cluster-contigs`  
> `-p <profile>` : The merged `PROFILE.db`.  
> `-c <contigs>` : The contigs database file.  
> `-C <collection>` : Give a name for the output collection of bins.  
> `--driver <binner>` : Select the binning program.  
> `--log-file <file>` : Output log file to record everything that happens during this command.  
> `--just-do-it` : You need to enable this option because `anvi-cluster-contigs` is an experimental workflow that is still under development and may produce unexpected errors.  

> `anvi-summarize`  
> `-o <dir>` : Output directory to store the summary.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
anvi-cluster-contigs -p /path/to/merged_profiles/PROFILE.db -c /path/to/contigs.db -C METABAT2 --driver metabat2 --just-do-it --log-file /path/to/metabat2.log
anvi-summarize -p /path/to/merged_profiles/PROFILE.db -c /path/to/contigs.db -o /path/to/SUMMARY_METABAT2 -C METABAT2
```

</details>


#### Using `MaxBin2`  

The commands are the exact same as before. You just need to write `MAXBIN2` instead.  

```bash
anvi-cluster-contigs -p ? -c ? -C MAXBIN2 --driver maxbin2 --log-file ? --just-do-it
anvi-summarize -p ? -c ? -o ? -C MAXBIN2
```

<details><summary><b>Complete command</b></summary>

```bash
anvi-cluster-contigs -p /path/to/merged_profiles/PROFILE.db -c /path/to/contigs.db -C MAXBIN2 --driver maxbin2 --just-do-it --log-file /path/to/maxbin2.log
anvi-summarize -p /path/to/merged_profiles/PROFILE.db -c /path/to/contigs.db -o /path/to/SUMMARY_MAXBIN2 -C MAXBIN2
```

</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## Questions  

+ **How many ${\color{red}ARCHAEA}$ bins did you get from `MetaBAT2`?**  
+ **How many ${\color{red}ARCHAEA}$ bins did you get from `Maxbin2`?**  

_**Note:**_ We will use the bins generated from `MetaBAT2` for downstream steps.  

