<!-- - - - - - - - - - - - - - - - 
Modder: Hieu
Date: 2026.01.29

Modifications:  

. Add bin summary from day 3.  
. Rephrase explanations for clarity.  
. Fix inaccurate or confusing sample code.  
. Re-organize code explanations.  
. Explain command options in more details.  
. Add table of contents.  

To add:  
.

- - - - - - - - - - - - - - - - -->


$${\color{red}DAY 4}$$


<!-- Table of Contents GFM -->

* [Aim](#aim)
* [1. Evaluating MAGs Quality](#1-evaluating-mags-quality)
    * [Estimating genome completeness](#estimating-genome-completeness)
    * [Examining bins manually](#examining-bins-manually)
* [Questions](#questions)
* [2. Refining Archaea bins](#2-refining-archaea-bins)
    * [2.1. Detecting chimeras in MAGs](#21-detecting-chimeras-in-mags)
    * [2.2. Creating interactive plots of the chimeras](#22-creating-interactive-plots-of-the-chimeras)
* [Questions](#questions-1)
* [3. Manual bin refinement](#3-manual-bin-refinement)
* [Questions](#questions-2)
* [4. Visualizing Coverage](#4-visualizing-coverage)
* [Questions](#questions-3)

<!-- /Table of Contents -->


## Aim  


On day 3, you binned assembled contigs into genome bins, and checked the bins to see if they represent bacteria or Archaea. Today, you will examine these bins more closely, and try to improve their quality.  

We will focus on refining only the $\color{red}ARCHAEA\ BINS\ !!$  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 1. Evaluating MAGs Quality  

Once the binning is done, you can look at how good each of the MAG bin is.  

### Estimating genome completeness  

Each bin is now hypothetically a genome of one species (or more precisely, a metagenome-assembled genome - MAG). You can evaluate how complete and redundant each of the bin (MAG) is with [`anvi-estimate-genome-completeness`][est-compl]:  

[est-compl]: https://anvio.org/help/9/programs/anvi-estimate-genome-completeness/  

```
anvi-estimate-genome-completeness -c ./path/to/contigs.db -p ./path/to/merged_profiles/PROFILE.db -C METABAT2
```

To only check what bin collections you have generated (without calculating genome completeness), you can use:  

```bash
anvi-estimate-genome-completeness --list-collections -p ./path/to/merged_profiles/PROFILE.db -c ./path/to/contigs.db
```

This data should also be available already as part of the `.html` report from binning (`./path/to/SUMMARY_METABAT2/index.html`).  


### Examining bins manually  

**Note:** This is an $\color{red}INTERACTIVE$ step. Follow the instructions in the `README.md` file $\color{red}!!!$  

Use the following command to initiate an interactive `anvi'o` session:  

```bash
anvi-interactive -p ./path/to/merged_profiles/PROFILE.db -c ./path/to/contigs.db -C METABAT2
```

`anvi-interactive` manually inspect and customize the bins. Once your browser window is open, you can set all relevant parameters, then click on `Draw` in the bottom left corner to generate a view of the bins.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## Questions

+ **Which binning strategy gives you the best quality for the $\color{red}ARCHAEA$ bins?**  
+ **How many $\color{red}ARCHAEA$ bins do you get that are of _High_ quality?**  
+ **How many $\color{red}BACTERIA$ bins do you get that are of _High_ quality?**  

_**Note:**_ We will use only the bins generated from `MetaBAT2` for downstream steps.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 2. Refining Archaea bins  

When you want to alter some data, it is always a good idea to work on a copy of the data first. Locate the $\color{red}Archaea\ bins$ from `MetaBAT2`. The contigs in each bin are contained together in one `fasta` file, and this is the only file you need.  
**Copy the fasta files** to a new directory for refining (remember to ***keep the bins separate***, each in a different directory).  

$\color{yellow}Tip:$ As an example, here is one way to organize the bins for refining:  

``` bash
day4/
└── refine/
    ├── METABAT__10/
    │   └── METABAT__10-contigs.fa
    ├── METABAT__13/
    │   └── METABAT__13-contigs.fa
    └── METABAT__21/
        └── METABAT__21-contigs.fa
```


### 2.1. Detecting chimeras in MAGs  

For this, we will use [GUNC][gunc doc] to check for chimeras and potential contamination. Chimeric genomes are genomes wrongly assembled out of two or more genomes coming from separate organisms (see more [here][gunc paper].  

[gunc doc]: https://grp-bork.embl-community.io/gunc/
[gunc paper]: https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02393-0

**Note:** `GUNC` is installed in the `00_gunc` environment.  

```bash
gunc run -i ? -r ? --out_dir ? --detailed_output --threads 12
```

The database for `GUNC` is installed at:  

```bash
$WORK/databases/gunc/gunc_db_progenomes2.1.dmnd
```

<details><summary>$\color{yellow}Hint$</summary>

> `-i <fasta>` : Input genome/MAG as `fasta` format.  
> `-r <database>` : The reference database for `GUNC` to identify the taxa in the sample.  
> `--out_dir <dir>`: Path to the output directory where the results will be stored.  
> `--detailed_output` : Write output in details, of course :D  
> `--threads 12` : The number of threads to use for faster computation.  

</details>

<details><summary><b>Complete command</b></summary>

```bash
gunc run -i ./path/to/METABAT_BIN_NNN/metabat_bin_NNN.fasta -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir ./path/to/METABAT_BIN_NNN/gunc_out --detailed_output --threads 12
```

</details>

<details><summary><b>Advanced version</b></summary>

```bash
for mag in ./path/to/METABAT__BIN_*/*.fa; do
    bin_dir=$(dirname $mag)
    gunc run -i $mag -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir ${bin_dir}/gunc_out --detailed_output --threads 12
done
```

</details>

If `GUNC` reports errors related to missing dependencies, try installing them:  

``` bash
micromamba activate 00_gunc
micromamba install bioconda::prodigal
micromamba install bioconda::diamond==2.0.4.
```


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


### 2.2. Creating interactive plots of the chimeras  

After running chimera detection, you can visualize the results in a plot.  

```bash
gunc plot -d ? -g ? --out_dir ?
```

<details><summary>$\color{yellow}Hint$</summary>

> `-d <diamond>` : The database search result file from `diamond`, which is a program that `gunc run` uses in the background.  
> `-g <gene_counts>` : The `gene_counts.json` file from `gunc run`.  
> `--out_dir <dir>` : Output directory where the plotting results will be stored.  

</details>

<details><summary><b>Complete command</b></summary>

``` bash
gunc plot -d ./path/to/METABAT__BIN_NNN/gunc_out/diamond_output/*.diamond.progenomes_2.1.out -g ./path/to/METABAT__BIN_NNN/gunc_out/gene_calls/gene_counts.json --out_dir ./path/to/gunc_out
```

</details>

<details><summary><b>Advanced version</b></summary>

```bash
for mag in ./path/to/METABAT__BIN_*/*.fa; do
    bin_dir=$(dirname "$mag")
    gunc plot -d $bin_dir/gunc_out/diamond_output/*.diamond.progenomes_2.1.out -g $bin_dir/gunc_out/gene_calls/gene_counts.json --out_dir ./path/to/gunc_out
done
```

</details>


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->

## Questions  

- **Do you get $\color{red}ARCHAEA$ bins that are chimeric?**  
    - $\color{yellow}Hint:$ Look at the CSS score and the column "PASS GUNC" in the `gunc` output tables for each bin.  
- **In your own words, briefly explain what a chimeric bin is.**  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 3. Manual bin refinement  

As large metagenome assemblies can result in hundreds of bins, pre-select some of the better ones for manual refinement, e.g., > 70% completeness.  

Again, you should ***create a copy*** of the original bin, and $\color{red}only\ work\ on\ the\ copy\ !!$  

``` bash
cp ./path/to/PROFILE.db ./path/to/PROFILE_refined.db
```

Then, use [`anvi-refine`][anvi-refine] to work on your bins manually. **Important:** `anvi-refine` will overwrite the original bins after you modify them.    
$\color{red}Only\ work\ on\ the\ copy\ !!$ - `PROFILE_refined.db`.  

[anvi-refine]: https://anvio.org/help/main/programs/anvi-refine/

**Note:** This is an $\color{red}INTERACTIVE$ step. Follow the instructions in the `README.md` file $\color{red}!!!$  

``` bash
anvi-refine -c ./path/to/contigs.db -p ./path/to/refine/PROFILE_refine.db --bin-id METABAT__BIN_NN -C METABAT2 
```

$\color{yellow}Tips:$  

- You can sort your bins by GC content, coverage, or both.  
- When refining bins, try clustering based on only differential coverage, and then only based on sequence composition to identify outliers.  
- The interface allows you to categorize contigs into separate bins (selection tool).  
- You can remove contigs. Unhighlighted contigs are removed when the data is saved.  
- You can also evaluate the community's taxonomy and duplicated single-copy core genes.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## Questions

- **How much could you improve the quality of your ${\color{red}ARCHAEA}$?**  
    - Compare the completeness and redundance of the bin *before* and *after* refining.  

**Note:** Attach all relevant figures that you generated.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## 4. Visualizing Coverage  

How abundant are the Archaea MAGs, actually? Take a look at them again.  
 
```bash
anvi-interactive -p ./path/to/merged_profiles/PROFILE.db -c ./path/to/contigs.db -C METABAT2
```

$\color{yellow}Hint:$ Also check the `index.html` report from binning.  

$\color{yellow}Tip:$ You can also try [`anvi-inspect`][anvi-inspect], [`anvi-script-get-coverage-from-bam`][get-cov-bam], or [`anvi-profile-blitz`][prof-blitz]. You can check their `--help` result or look at their documentations for details.  


<!-- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -->


## Questions  
  
- **How abundant (relatively) are the $\color{red}Archaea$ bins in the 3 samples?**  
