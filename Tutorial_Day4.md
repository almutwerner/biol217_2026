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
* [Coverage visualization](#coverage-visualization)
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

```
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
gunc plot -d ./path/to/METABAT__BIN_NNN/gunc_out/diamond_output/*.diamond.progenomes_2.1.out -g ./path/to/gunc_out/gene_calls/gene_counts.json --out_dir ./path/to/gunc_out
```

</details>

<details><summary><b>Advanced version</b></summary>

```bash
for mag in ./path/to/METABAT__BIN_*/*.fa; do
    gunc plot -d ./diamond_output/METABAT__#-contigs.diamond.progenomes_2.1.out -g ./gene_calls/gene_counts.json
    gunc run -i $mag -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir ./path/to/gunc_out --detailed_output --threads 12
done
```

</details>


## Questions


- **Do you get $\color{red}ARCHAEA$ bins that are chimeric?**  
    - $\color{yellow}Hint:$ Look at the CSS score and the column "PASS GUNC" in the `gunc` output tables for each bin.  
- **In your own words, briefly explain what a chimeric bin is.**  


## 3. Manual bin refinement  

As large metagenome assemblies can result in hundreds of bins, pre-select the better ones for manual refinement, e.g. > 70% completeness.

Before you start, make a **copy/backup** of your unrefined bins the ``ARCHAEA_BIN_REFINEMENT``.

You can save your work as refinement overwrites the bins. 

``` 
module load gcc12-env/12.1.0
module load micromamba/1.3.1
micromamba activate 00_anvio
``` 

Use anvi refine to work on your bins manually. *“In the interactive interface, any bins that you create will overwrite the bin that you originally opened. If you don’t provide any names, the new bins’ titles will be prefixed with the name of the original bin, so that the bin will continue to live on in spirit.
Essentially, it is like running anvi-interactive, but disposing of the original bin when you’re done.” https://anvio.org/help/main/artifacts/interactive/*



```diff
-!!!!!!!!!!!!!!!!!!!!!AS MENTIONED BEFORE!!!!!!!!!!!!!!!!!!!!!
- Here you need to access anvi’o interactive -
- REPLACE the command line you want to run in interactive mode -
```

```
module load gcc12-env/12.1.0
module load micromamba/1.3.1
micromamba activate 00_anvio

anvi-refine -c ./path/to/contigs.db -C METABAT2 -p ./path/to/merged_profiles/PROFILE.db --bin-id METABAT__##
```

You can now sort your bins by **GC content**, by **coverage** or both. 

For refinement it is easier to use the clustering based on only differential coverage, and then only based on sequence composition in search for outliers.

The interface allows you to categorize contigs into separate bins (selection tool). Unhighlighted contigs are removed when the data is saved.

You can also evaluate taxonomy and duplicate single copy core genes.


You can also remove contigs. 

Spend some time to experiment in the browser.

For refinement use clustering based on only differential coverage, and then only based on sequence composition in search for outliers.


#### Questions
* Does the quality of your ${\color{red}ARCHAEA}$ improve? 
* hint: look at completeness redundancy in the interface of anvio and submit info of before and after 
* Submit your output Figure

> INSERT\
> YOUR\
> ANSWER\
> HERE


## Coverage visualization

You should manually visualize your **ARCHAEA BINS** coverage.
 
**Do so by using anvio interactive interface.**

## Questions
  
* **how abundant are the archaea bins in the 3 samples? (relative abundance)**
* **you can also use anvi-inspect -p -c, anvi-script-get-coverage-from-bam or, anvi-profile-blitz. Please look up the help page for each of those commands and construct the appropriate command line

!NOTE: check your binning html output, the second table, and look for mean coverage

* https://anvio.org/help/main/artifacts/summary/
* 
 
> INSERT\
> YOUR\
> ANSWER\
> HERE


