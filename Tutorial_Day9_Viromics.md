**$$\color{red}DAY\ 9$$**

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Viromics](#viromics)
	- [1. Aim](#1-aim)
	- [2. Tools and pipelines](#2-tools-and-pipelines)
	- [3. Commands (Do **not** execute\!)](#3-commands-do-not-execute)
	- [4. Questions](#4-questions)
		- [Note about the *Thinking for yourself* questions](#note-about-the-thinking-for-yourself-questions)
		- [Output structure and how to best create commands](#output-structure-and-how-to-best-create-commands)
		- [Virus identification and quality control](#virus-identification-and-quality-control)
		- [Clustering and Abundance](#clustering-and-abundance)
		- [Annotation](#annotation)
		- [Binning](#binning)
		- [Host prediction](#host-prediction)

<!-- /TOC -->

# Viromics

## 1. Aim
Today, you will learn how a typical virome analysis for a metagenomic dataset looks like. The general steps are:

* Read pre-processing and assembly (either virus-specific assembly or standard metagenomic assembly)
* Viral identification
* Virus-specific quality control
* Viral clustering
* Viral binning
* Abundance estimation
* Virus-specific annotation
* Host prediction

Additionally to the steps we will discuss today, it is possible to add steps to the workflow as needed. Often added steps are:

* Viral taxonomy prediction
* Viral Lifecycle prediction
* Virus-specific metabolic analyses

## 2. Tools and pipelines
It is possible to run all steps manually to have maximum control over all parameters. But often, it is more comfortable to use a pipeline instead. A **pipeline** is a (published) tried and tested workflow for a specific application purpose (e.g. "Viromics"). It combines multiple tools and custom scripts with preset cutoffs for e.g. filtering steps. Usually, that workflow represents how *the developers* prefer to do their analysis and it will work for basic cases without adjustments. But the user also has some degrees of freedom to change cutoffs to their liking, and it is possible to add additional tools afterwards.

Today, we will be using **MVP** (**M**odular **V**iromics **P**ipeline). We chose this pipeline (there are many available), since it follows a robust workflow suited as a baseline for an analysis. Additionally, the outputs are structured really well which makes it easy to use for newcomers.
MVP uses clean reads and assemblies from multiple samples to perform the general viromics analysis mentioned above (identification, QC, clustering, binning, abundance, annotation).

Since MVP only prepares output for host prediction via iPHoP (some tools need special formats to work) but does not run the tool itself, we will be manually running **iPHoP** (**i**ntegrated **P**hage-**Ho**st **P**rediction).

**MVP**
* [Publication](https://journals.asm.org/doi/10.1128/msystems.00888-24)
* [Repository](https://gitlab.com/ccoclet/mvp)

**iPHoP**
* [Publication](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.3002083)
* [Repository](https://bitbucket.org/srouxjgi/iphop/src/main/)



**MVP functions and dependencies**

| Purpose | Tool | Version | Repository |
| :---- | :---- | :---- | :---- |
| Viruses, proviruses, and plasmid identification | geNomad | 1.7.4 | [https://github.com/apcamargo/genomad](https://github.com/apcamargo/genomad) |
| Viral Taxonomy | geNomad | 1.7.4 | [https://github.com/apcamargo/genomad](https://github.com/apcamargo/genomad) |
| Quality assessment and filtering | CheckV | 1.0.3 | [https://bitbucket.org/berkeleylab/checkv/src/master/](https://bitbucket.org/berkeleylab/checkv/src/master/) |
| ANI clustering | CheckV | 1.0.3 | [https://bitbucket.org/berkeleylab/checkv/src/master/](https://bitbucket.org/berkeleylab/checkv/src/master/) |
| Abundance estimation | Bowtie2 | 2.5.4 | [https://bowtie-bio.sourceforge.net/bowtie2/index.shtml](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml) |
|  | Minimap2 | 2.28 | [https://github.com/lh3/minimap2](https://github.com/lh3/minimap2) |
|  | Samtools | 1.21 | [https://www.htslib.org/](https://www.htslib.org/) |
|  | CoverM | 0.7.0 | [https://github.com/wwood/CoverM](https://github.com/wwood/CoverM) |
| Gene function annotation | Prodigal | 2.6.3 | [https://github.com/hyattpd/Prodigal](https://github.com/hyattpd/Prodigal) |
|  | MMseqs2 | 14.7e284 | [https://github.com/soedinglab/MMseqs2](https://github.com/soedinglab/MMseqs2) |
|  | HMMER | 3.4 | [https://github.com/EddyRivasLab/hmmer](https://github.com/EddyRivasLab/hmmer) |
|  | DIAMOND | 2.1.0 | [https://github.com/bbuchfink/diamond](https://github.com/bbuchfink/diamond) |
| Viral Binning | vRhyme | 1.1.0 | [https://github.com/AnantharamanLab/vRhyme](https://github.com/AnantharamanLab/vRhyme) |


**Manually run tools (not part of the pipeline)**
| Purpose | Tool | Version | Repository |
| :---- | :---- | :---- | :---- |
| Viral host prediction | iPhoP | 1.3.3 | [https://bitbucket.org/srouxjgi/iphop/src/main/](https://bitbucket.org/srouxjgi/iphop/src/main/) |

\! We are not taking credit for the tools, we are simply explaining how they work for an in-house tutorial \!

---
## 3. Commands (Do **not** execute\!)

🔴 **You will not run the commands as they take a long time and databases are very large\!** 🔴

Instead, we ran them for you. Here is what we did:

<details><summary><b>MVP</b></summary>

```

module load gcc12-env/12.1.0
module load micromamba/1.3.1
micromamba activate MVP

cd $WORK/MVP_test

mvip MVP_00_set_up_MVP -i ./WORKING_DIRECTORY/ -m input_file_timeseries_final.csv  --genomad_db_path ./WORKING_DIRECTORY/00_DATABASES/genomad_db/ --checkv_db_path ./WORKING_DIRECTORY/00_DATABASES/checkv-db-v1.5/

mvip MVP_00_set_up_MVP -i ./WORKING_DIRECTORY/ -m input_file_timeseries_final.csv  --skip_install_databases

mvip MVP_01_run_genomad_checkv -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_02_filter_genomad_checkv -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_03_do_clustering -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_04_do_read_mapping -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv --delete_files

mvip MVP_05_create_vOTU_table -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_06_do_functional_annotation -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_07_do_binning -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv --force_outputs

mvip MVP_100_summarize_outputs -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

```
</details>

<details><summary><b>iPHoP</b></summary>

```

module load miniconda3/4.12.0
conda activate GTDBTk

export GTDBTK_DATA_PATH=./GTDB_db/GTDB_db


gtdbtk de_novo_wf --genome_dir fa_all/ --bacteria --outgroup_taxon p__Patescibacteria --out_dir output/ --cpus 12 --force --extension fa

gtdbtk de_novo_wf --genome_dir fa_all/ --archaea --outgroup_taxon p__Altarchaeota --out_dir output/ --cpus 12 --force --extension fa


module load micromamba/1.3.1
micromamba activate iphop_env

iphop add_to_db --fna_dir fa_all/ --gtdb_dir ./output/ --out_dir ./MAGs_iPHoP_db --db_dir iPHoP_db/

iphop predict --fa_file ./MVP_07_Filtered_conservative_Prokaryote_Host_Only_best_vBins_Representative_Unbinned_vOTUs_Sequences_iPHoP_Input.fasta --db_dir ./MAGs_iPHoP_db --out_dir ./iphop_output -t 12

```
</details>

---

## 4. Questions

### Note about the *Thinking for yourself* questions

These are questions to help you pause and ask yourself if you understood everything. **Do not skip them**, they are designed to help you understand what you are doing and why. Try to **answer them on your own** first (neighbours may be consulted). But please carefully **read the provided solutions afterwards**. If you don't understand the solution, let us know so we can help!

The questions marked as [*Optional*] can be skipped, if you are tight on time.

### Output structure and how to best create commands
🔴 Since it is only small queries you do **not** need to write scripts! Just write commands directly in the terminal! **No sbatch, no scripts, no interactive sessions, just terminal!** 🔴



In your work directory, you have a folder called **"viromics"**. This is the output we created for you using MVP and iPHoP. Everything you need for today is in there. The folder structure looks like this:
```
├── 01_GENOMAD
├── 02_CHECK_V
├── 03_CLUSTERING
├── 04_READ_MAPPING
├── 05_VOTU_TABLES
├── 06_FUNCTIONAL_ANNOTATION
├── 07_BINNING
└── 08_iPHoP
```
Each of the folders contains the output for one of the submodules. **Please look at this [graphic overview of their workflow](https://gitlab.com/ccoclet/mvp/-/raw/main/images/MVP_Complete_Workflow.png)** to understand where to find the output you need.


**Tip:** Build your commands bit by bit and add the argument responsible for **counting *last***. This is to ensure that you can always double check the actual output (e.g. is there anything that shouldn't be in there?) before counting all lines.

You can always cancel commands with **CTRL C** (copying is CTRL Shift C), if they run suspiciously long (so more than a few seconds)!

Feel free to google if you don't know which command will get you the result you want! There is always more than one solution. You are not expected to use regular expressions.

It's dangerous to go alone, take **[this](https://man7.org/linux/man-pages/man1/grep.1.html)**!


### Virus identification and quality control

1) How many **free viruses** are in the BGR\_140717 sample?

	<details><summary><b>Hint</b></summary>

	Check the folder 01_GENOMAD and look for the *_virus.fna files. Use grep. Write your commands directly into the terminal. You will **not** need to use scripts or sbatch today!
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	grep ">" -c 01_GENOMAD/BGR_140717/BGR_140717_Viruses_Genomad_Output/BGR_140717_modified_summary/BGR_140717_modified_virus.fna
	```
	</details>

2) How many **proviruses** are in the BGR\_140717 sample?

	<details><summary><b>Hint</b></summary>
	Check the folder 01_GENOMAD and look for the *_virus.fna files. Use grep.
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	grep ">" -c 01_GENOMAD/BGR_140717/BGR_140717_Proviruses_Genomad_Output/proviruses_summary/proviruses_virus.fna
	```
	</details>


3) How many ***Caudoviricetes*** viruses are in all samples together? (Use the filtered version)
	<details><summary><b>Hint</b></summary>
	Look at the 02_CHECK_V folder and use the filtered quality summary.
	Tip: It is possible to solve this question in a single command. Use wildcards!
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	grep -c "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv
	```
	</details>

4) How many **Unclassified** viruses are in all samples together? (Use the filtered version)
	<details><summary><b>Hint</b></summary>
	Look at the 02_CHECK_V folder and use the filtered quality summary.
	Tip: It is possible to solve this question in a single command. Use wildcards!
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	grep -c "Unclassified" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv
	```
	</details>

5) What other **taxonomies** are there across all files? Below, paste a table with (at least) taxonomy, Genome type and Host type.
	<details><summary><b>Hint</b></summary>

	Use grep, but with **another argument** instead of -c ([Manual page](https://man7.org/linux/man-pages/man1/grep.1.html)).
	Remember, you need **everything except** Caudoviricetes and Unclassified, and grep is case sensitive for any options or arguments (-c and -C do different things. grep "example" and "Example" get different results).

	Tip: With the **pipe character**, you can chain commands together. Each one will be performed based on the **output of the previous command**. E.g.:

	```
	grep .... | command 2 .... | command 3 .... | command 4 ...
	```
	Will first do grep, then command 2 on the output of grep, then command 3 on the output of command 2, and so on.

	</details>

	<details><summary><b>Finished commands</b></summary>
	Take all lines that don't contain "Caudoviricetes", "Unclassified", or "Sample" anywhere by using -v or --invert-match. We are excluding the keyword "Sample", because that is what the headers start with! You could also use any other word that is <b>exclusive</b> to the headers, like "viral_genes" or "kmer_freq".

	```
	grep -v "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv |grep -v "Unclassified" |grep -v "Sample"
	```
	</details>

6) How many **High-quality** and **Complete** viruses are in all samples together? (Use the filtered version and focus on the CheckV quality)
	<details><summary><b>Hint</b></summary>
	Look at the 02_CHECK_V folder and use the filtered quality summary. For this, we will need to focus on <b>specific columns</b> of the file. Use the "cut" command.
	</details>

	<details><summary><b>Finished commands</b></summary>
	For this command we need to remember, that grep looks for a match *anywhere*. Not just in the column you are thinking about. The column "miuvig_quality" also has the keyword "High-quality". So we need to specify the column we want grep to look at.

	So, we first extract only column 8 that contains the CheckV quality (column counting starts at 1). Then, we take all lines that contain any of the two keywords (-e for each of the keywords) and then count using ```grep -c ""```. The empty quotation marks can be used to match everything, so that each line is counted once. Instead of counting with grep, you could also do ```wc -l```.

	```
	cut -f 8 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -e "High-quality" -e "Complete" |grep -c ""

	```
	Bonus: If you do ```cut -f 1,8``` (and skip the counting), you can see which samples the viruses came from (since you are cutting **two columns**, Sample and checkv_quality).
	</details>

7) Create a table based on the CheckV quality with the columns **Sample, Low-quality, Medium-quality, High-quality, Complete**. Fill it with the amount of viruses for each of the categories.
Tip: If you are tight on time, just save the output and make it pretty later!
	<details><summary><b>Hint</b></summary>
	Remember to focus on the CheckV quality only. Even though it is possible to fully generate the table in a single query (using the paste command), we suggest that you mix a for loop, cut, grep, and some manual table creation to get to the result.

	Build up your commands based on the code below. **Once you are sure that everything you need to be done per iteration (per sample) is written down, remove the ```;break```**. Break will exit a for loop after the first iteration (this is useful to construct queries in peace without having to scroll through all the output).

	You can add new commands in between semicolons. E.g. ```...; echo "Number of low quality viruses"; ...```.
	```
	for x in BGR_130305  BGR_130527  BGR_130708  BGR_130829  BGR_130925  BGR_131021  BGR_131118  BGR_140106  BGR_140121  BGR_140221  BGR_140320  BGR_140423  BGR_140605  BGR_140717  BGR_140821  BGR_140919  BGR_141022  BGR_150108; do echo $x; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Low-quality" ;break; done
	```
	(Just test around 😊. You should be able to learn everything you need to know from the code above.)
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	for x in BGR_130305  BGR_130527  BGR_130708  BGR_130829  BGR_130925  BGR_131021  BGR_131118  BGR_140106  BGR_140121  BGR_140221  BGR_140320  BGR_140423  BGR_140605  BGR_140717  BGR_140821  BGR_140919  BGR_141022  BGR_150108; do echo $x; echo  "Low-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Low-quality" ; echo  "Medium-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Medium-quality"; echo  "High-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "High-quality"; echo  "Complete"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Complete"; done

	```
	Low effort edit, so that it can be converted to markdown more quickly:
	```
	for x in BGR_130305  BGR_130527  BGR_130708  BGR_130829  BGR_130925  BGR_131021  BGR_131118  BGR_140106  BGR_140121  BGR_140221  BGR_140320  BGR_140423  BGR_140605  BGR_140717  BGR_140821  BGR_140919  BGR_141022  BGR_150108; do echo "| $x | "; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Low-quality" ; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Medium-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "High-quality";  cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Complete"; done
	```
	</details>

8) For the **Complete** viruses from all samples, extract all the lines (from the same output file you just used to answer previous questions) so we can take a closer look. Also add a header so we know what each column contains.

	<details><summary><b>Hint</b></summary>
	We now need all columns again, for all samples. You don't need to grab the header in the same query (you are allowed to extract it manually).
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	grep "Complete" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv
	```
	</details>

9) Answer the following questions based on the data from 8):
	<details><summary><b>Hint</b></summary>
	The output table is based on merged output from CheckV and geNomad. Check the output documentation linked below, if there is any columns you don't understand.

	[CheckV](https://bitbucket.org/berkeleylab/CheckV/src/master/#markdown-header-output-files),
    [geNomad](https://portal.nersc.gov/genomad/quickstart.html#understanding-the-outputs)
	</details>

	* In what samples were the complete viruses found?
	* Are they integrated proviruses?
	* How long are they?
	* How many **viral hallmark genes** do they have?
	* What percentage of the **viral genes** are **viral hallmark genes**? You may round to full numbers.
	* *Thinking for yourself:* Why are there more genes (gene_count) than viral genes and host genes combined? Tip: What does the column gene_count tell us?

	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>
	[For the last question]: Gene_count are all genes encoded in this sequence. We know for sure that some match known host and virus genes (viral_genes, host_genes). For the other genes, we simply don't know. They are just normal (viral) genes without any matches to the reference databases of geNomad and CheckV.
	</details>

### Clustering and Abundance
10) *Understanding the output:* Find the **clustering output**. What is the difference between the three tsv files?
Tip: Just look at the file names.

11) How many **cluster representatives** are there?
	<details><summary><b>Hint</b></summary>
	When counting, make sure to exclude the header (you don't need to exclude it in the query, just do -1).
	</details>

	<details><summary><b>Finished commands</b></summary>
	Don't forget to subtract one from the number you get!

	```
	grep -c "" 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv
	```
	Bonus: If you are unsure, if there is really only one line per cluster representative, you can do this instead:
	```
	cut -f 1 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv |sort|uniq |wc -l
	```
	Here, we extract the first column (that has the identifiers of all cluster representatives). Then, we sort it and use only the unique lines (sorting is needed to ensure that uniq works properly). Counting is done using the alternative method mentioned above, but you could also do ```grep -c ""```.
	</details>

12) How many of the cluster representatives are **proviruses**?
	<details><summary><b>Hint</b></summary>
	We need to look at a specific column for this again. Remember to build your command bit by bit and do the counting step <b>last</b>, so you can immediately spot mistakes.
	</details>

	<details><summary><b>Finished commands</b></summary>

	```
	cut -f 5 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv | grep -c "Yes"
	```
	</details>

13) What clusters do the **complete** viruses from 8) belong to? How large are the clusters?
	<details><summary><b>Hint</b></summary>

	Use the column **virus_id** from the output you saved in 8). That ID is unique to a sequence and will stay the same throughout all files. So it can be used as identifyer everywhere.
	To get information about the clusters, look at the first two columns of the file you also used in 12).
	</details>

	<details><summary><b>Finished commands</b></summary>
	To get the lines containing the information about the relevant clusters, do this:

	```
	grep -e "BGR_131021_NODE_96_length_46113_cov_32.412567" -e "BGR_140121_NODE_54_length_34619_cov_66.823718" -e "BGR_140717_NODE_168_length_31258_cov_37.020094" 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv
	```
	You could leave it here and manually count the cluster members. If you want to be a bit more fancy, you could do this:

	To get the number of cluster members per cluster, we first iterate through the three clusters we are interested in and print their name. Then, we extract column 2 (which has all cluster members) from the file with all the clusters and limit the selection to only that one cluster we are interested in. Afterwards, we grep comma, since the cluster members are comma separated. By using -o with grep, we have it print **only** the matches, one per line. Counting needs to be done in a seperate command. ```grep "," -o -c``` returns 1 (since it is only one line). So we need to count after piping the output.
	Now we know the amount of commas. To get the amount of cluster members, we need to add one, since the last member doesn't have a comma written behind it.
	```
	for x in BGR_131021_NODE_96_length_46113_cov_32.412567 BGR_140121_NODE_54_length_34619_cov_66.823718 BGR_140717_NODE_168_length_31258_cov_37.020094; do echo $x; cut -f 2 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv | grep "$x" | grep "," -o |grep "" -c; done
	```
	</details>

14) *Thinking for yourself:* Now we want to look at **abundances**. The files can be found in 04_READ_MAPPING, split per sample. Open any of the CoverM files and **ignore everything except the first two columns**.

	What does this file tell you, conceptionally? Meaning: What do the lines in this file represent (i.e. what kind of data is described in line 2, line 3, .....), who do the IDs listed in the second column belong to, what kind of data was combined to generate this file, and what can we learn from it?
		<details><summary><b>Hint</b></summary>
		Check how many lines this file has or find the documentation for this module.
		Try to answer this question on your own first!
		</details>
		<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>
		For this file, the entire reads of one sample were mapped against **all** 5374 viral cluster representatives. Column 2 contains the ID of the currently described cluster representative. From this file, we can learn how abundant each of the recovered viruses (vOTUs) is in each of the samples (each sample has its own file).
		</details>

15) *Thinking for yourself:* If you scroll through the CoverM file you selected, you will come across lines where all of the metrics (except length) are **zero**. How could this have happened? Is it a bug or a feature?!  
	<details><summary><b>Hint</b></summary>
	Think about what you just learned in 14).
	</details>
	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>

	As you just learned, this file contains mapping information about **all** viruses but for only **one** sample. If you look back at your table from 7), not every sample contains 5000+ viruses. So the lines with only zeros are cluster representatives for viruses that a) weren't assembled from this particular sample and b) didn't even have any matching reads (if you remember from Alex' lectures: Not all reads become contigs, not all contigs become vMAGs). Those can be e.g. viruses that weren't present at this timepoint (e.g. died out), in this sample (they swam away when the technician came to collect), or that were in the sample but didn't get sequenced or the reads got filtered out in the read cleaning step.
	</details>

16) [*Optional*] Right now, the output for this module is spread across multiple files, which is inconvenient and happens a lot in bioinformatics. Luckily, each file has a column where the sample information is listed anyways, so we can merge them. **Task**: Merge all **CoverM** files for all the samples (order doesn't matter) using e.g. the ```cat``` command (not cut). Remember to exclude the headers and keep only one. Your finished file should have ```96733 lines total```.
	<details><summary><b>Hint</b></summary>

	Cat con**cat**enates multiple files and prints the result. It is compatible with wildcards. You can **redirect** output that is being printed to the terminal to a file by using the ```>``` character (you did this in the metagenomic parts when converting sam to bam!) This is how you use it:
	```
	cat file1 file2 file... > new_merged_file.txt
	```

	Tip: You can also redirect output from other commands like ```grep``` and can use temporary/intermediate files....

	Tip 2: Check your merged file using ```	less```. If you press ```SHIFT G```, it will take you to the bottom of the file (Is it a different Sample ID?). Exit with ```Q```.

	[Documentation](https://man7.org/linux/man-pages/man1/cat.1.html)

	</details>

	<details><summary><b>Finished commands</b></summary>

	First, we get everything except the header (```-v "Sample"```) and redirect the output to a temporary file. Since grep will also print the file name when using wildcards (or multiple files), we tell to just don't.

	```
	grep --no-filename -v "Sample" 04_READ_MAPPING/BGR_*/*_CoverM.tsv >04_READ_MAPPING/tmp_CoverM_output.tsv
	```
	Next, we extract the header of one of the original files and store it in another temporary file. You could also copy and paste the header manually. But since it is a **t**ab **s**eparated **v**alues file, and tab characters behave weird when being copied (to put it simple), it is better to do it like this.

	```
	grep "Sample" 04_READ_MAPPING/BGR_150108/BGR_150108_CoverM.tsv >04_READ_MAPPING/tmp.tsv

	```

	Now, all that is left is to concatenate both files (here, the file order matters since we want the header to be on top!) and remove all temporary files (be very careful when removing files using wildcards!!!).

	```
	cat 04_READ_MAPPING/tmp.tsv 04_READ_MAPPING/tmp_CoverM_output.tsv >04_READ_MAPPING/merged_CoverM_output.tsv
	rm 04_READ_MAPPING/tmp*

	```
	</details>

17) Using the file you just created, how **abundant** are the complete viruses in different samples (**RPKM**)? Create a table.
	<details><summary><b>Hint</b></summary>
	Remember what you learned before: All the sequences have unique identifiers. How were your complete viruses called (Task 13)?

	Remember what you learned before: How could you look for multiple patterns within a file?
	</details>

	<details><summary><b>Finished commands</b></summary>

	Here, we are looking for multiple patterns using grep. It will result in a file that is ordered by sample, not by virus.

	```
	grep -e "BGR_131021_NODE_96_length_46113_cov_32.412567" -e "BGR_140121_NODE_54_length_34619_cov_66.823718" -e "BGR_140717_NODE_168_length_31258_cov_37.020094" 04_READ_MAPPING/merged_CoverM_output.tsv
	```

	To have it nicer for parsing it into a table, we can run it in a for loop to force it to sort by virus. In addition, we here extract only the columns that are interesting to us (Sample, Contig, RPKM).

	```
	for x in "BGR_131021_NODE_96_length_46113_cov_32.412567" "BGR_140121_NODE_54_length_34619_cov_66.823718"  "BGR_140717_NODE_168_length_31258_cov_37.020094" ; do grep "$x" 04_READ_MAPPING/merged_CoverM_output.tsv | cut -f1,2,11 ; done
	```
	You can redirect (```>```) the output to a file as well (won't have a header), if you add this at the end:
	```
	..... ; done > 04_READ_MAPPING/merged_viruses_RPKM.tsv
	```

	</details>

### Annotation
18) *Understanding the output:* Find the annotation output from **module 06**. Based on the file naming alone, what is each of the files for and what is different between files with the same extension (file ending, like ```.tsv```)?

	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>

	The two tsv files contain gene annotation information for only the 5000+ viral cluster representatives. (```GENOMAD```, ```PHROGS``` and ```PFAM``` are the databases that were used for annotation.)

	The difference between them is, that one is based on something ```conservative``` and the other on something ```relaxed```. Those are filtering parameters. If you count the amount of lines in each file, you can see that the conservative file (```7749```) is way shorter than the relaxed file (```24648```).

	The ```faa``` files contain the actual protein predictions; one for the cluster representatives and one for all viruses.

	</details>

19) We will focus on the file with the **conservative** results. Find the lines regarding the **complete viruses**.

	For this, we will be using **LibreOfficeCalc** (Working with Excel substitutes is another cornerstone of Bioinformatics). In order for it to work, you need to **download the file to your PC** (it doesn't work directly from the cluster!). Open the file. Then find the **quick filter function** (or press ```SHIFT CTRL L```). Column 3 contains the sequence identifiers we have been using the whole time, while column 2 has gene IDs. **Filter your results** by clicking on the arrow at column 3 and pasting the first of your complete virus identifiers into the search bar there (repeat for all complete viruses later).

	**Important**: As soon as you open the **filtering window**, you can't use your keyboard anywhere else (e.g. to copy things). You need to close the popup first, then you can copy things again.
	Also, make sure to scroll to the sides. There are **more columns** in the table than you can currently see!

	Answer these questions for **all** of your complete viruses:
	* **How many genes** does the complete virus have?
		<details><summary><b>Hint</b></summary>
		After filtering, select all result rows and look at the bottom of the window.
		</details>

	* What kind of genes does this virus have? Look at the **PHROGS_Category**. (This can also be solved by using commands)

		<details><summary><b>Finished commands</b></summary>

		```
		for x in "BGR_131021_NODE_96_length_46113_cov_32.412567" "BGR_140121_NODE_54_length_34619_cov_66.823718"  "BGR_140717_NODE_168_length_31258_cov_37.020094" ; do echo $x; grep "$x" 06_FUNCTIONAL_ANNOTATION/MVP_06_All_Sample_Filtered_Conservative_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Gene_Annotation_GENOMAD_PHROGS_PFAM_Filtered.tsv | cut  -f23 |sort | uniq; done
		```
		</details>

20) Reset the table so you can see **all** viruses again. Filter **PHROGS_Category** to ```moron, auxiliary metabolic gene and host takeover```. If you look at the column **PHROGS_GO_Annotation**, you can see more general information related to the pathways.

	What kind of **metabolism** are the viruses involved in?

21) [*Optional*] While still looking at the results filtered for ```moron, auxiliary metabolic gene and host takeover```:  Are there any **toxin genes**? Briefly look up the function of them (What do they do, where do they occur, what does it mean for this virus and its host?)
	<details><summary><b>Hint</b></summary>

	Column **PHROGS_Annotation** or **PFAM_Annotation**
	</details>

### Binning

22) *Understanding the output:* Find this table and open it: ```07_BINNING/07C_vBINS_READ_MAPPING/MVP_07_Merged_vRhyme_Outputs_Unfiltered_best_vBins_Memberships_geNomad_CheckV_Summary_read_mapping_information_RPKM_Table.tsv```

	What does each **line** represent (focus on column 1,2,5)? **What is the purpose of this table?**

	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>

	This table holds information for the different **viral bins**. Each line is one bin member, so one virus. By design of this pipeline, binning was performed on the vOTUs. So those viruses are the **cluster representatives** from module 03.

	This table aggregates most output from previous modules (like quality, taxonomy, abundance etc.). For each time point, we have the **covered fraction** (how much of the contig has reads mapped to it) and the **RPKM** ("Reads Per Kilobase Million", coverage of the contig, normalized by sequencing depth ([explanation in words](https://www.rna-seqblog.com/rpkm-fpkm-and-tpm-clearly-explained/), [explanation as formula](https://www.metagenomics.wiki/pdf/qc/RPKM))). So to put it simple: We can see the abundance of each cluster representative at different time points.
	</details>

23) How many **High-quality** "viruses" are there after binning?
	<details><summary><b>Hint</b></summary>

	Apply the filter based on the CheckV quality.
	Tip: Remember what the **purpose of binning** is!
	</details>

	<details><summary><b>"Solution"</b></summary>
	If your number has two digits, it is wrong! Why do we do binning again?
	Tip: Select all members of one vBin (=vMAG), then look at the RPKM values.
	</details>

24) *Thinking for yourself*: Filter the results so you can only see ```vBin_16``` (or just highlight the corresponding 5 lines). The metrics for all bin members are the same. But ```membership_provirus``` has different values. What does this tell you about the vBin/**vMAG** 16?
	<details><summary><b>Hint</b></summary>

	What did you learn about the **life cycles** of viruses earlier today?
	</details>

	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>

	We do binning to generate **vMAGs**, viral metagenome assembled genomes. This means, all members of a vMAG or vBin should belong to the same virus. Since the virus 16 was detected both as an **integrated provirus** and a **free virus**, it is likely a virus that can enter **both lytic and lysogenic cycle** (assuming the binning is correct).
	</details>

25) *Thinking for yourself*: Are your **complete** viruses part of a bin? **Why does this result make sense?**
	Tip: Use the search function or ```grep```!

	<details><summary><b>Hint</b></summary>
	Think about the purpose of binning.
	</details>

	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>

	The complete viruses are **not** part of a bin. The purpose of binning is to restore a **fragmented** genome by grouping together the individual fragments based on coverage information. Since our **complete** viruses are, well, complete, we don't need to find their missing pieces.

	</details>

### Host prediction

26) *Understanding the output*: Find this table and open it: ```08_iPHoP/Host_prediction_to_genome_m90.csv```. The input file to generate this table is ```08_iPHoP/MVP_07_Filtered_conservative_Prokaryote_Host_Only_best_vBins_Representative_Unbinned_vOTUs_Sequences_iPHoP_Input_clean.fna```. Based on the **first column** of the table and the **name** of the input file: What does each **line** represent? (Meaning: What kind of viruses are in this file?)

	Tip: This is a **csv** file. So you might need to change the delimiter to **comma** when you open the file.

	<details><summary><b>Solution (only read once you tried thinking on your own!)</b></summary>

	The input file contains the **best vBins** (written as vBin_number in the table, with vRhyme_number__sequenceID being individual sequences of that bin), and all **unbinned vOTU representatives**. Each line is one virus (unbinned vOTU representative, vMAG, or vBin member=binned vOTU representative), where a **host** could be predicted. Viruses **without** a prediction are **not** in the file. Hosts that start with BGR_* are **MAGs** from the dataset, while all other host identifiers are from reference databases.
	</details>

27) What hosts were predicted for the **complete viruses**? From what habitat did the hosts come from?
	<details><summary><b>Hint</b></summary>

	Tip: You do **not** need to google for this. Just look at the file and think about the dataset.
	</details>

	<details><summary><b>Finished commands</b></summary>

	You can also just use the **search function**. But if you don't trust the results and suspect you **missed something**, use grep:

	```
	for x in "BGR_131021_NODE_96_length_46113_cov_32.412567" "BGR_140121_NODE_54_length_34619_cov_66.823718"  "BGR_140717_NODE_168_length_31258_cov_37.020094" ; do echo "Searching for: $x"; grep "$x" 08_iPHoP/Host_prediction_to_genome_m90.csv ;done
	```
	</details>

28) Find an example for a virus with **more than one host prediction**
	* A) With **closely** related hosts
	* B) With **distantly** related hosts

	Be sure to look at the **VirusID** in column 1 carefully to make sure it is not just different viruses!
	Tip: You won't  have to scroll very far...

	[For both A) and B)]:
	What might be the reason for this result? Discuss based on:
	* 1) Biological reasoning
	* 2) Prediction method

		<details><summary><b>Hint</b></summary>

		Look at the **last 3 columns** and think about what you learned about host prediction methods earlier today.  Explanation of ```Main method``` ([documentation](https://bitbucket.org/srouxjgi/iphop/src/main/)):

		```blast```: Score obtained only from blastn hits to host genomes

		```iPHoP-RF```: Score obtained based on all host-based tools (blastn to host genomes, CRISPR hits, k-mer tools: WIsH, VHM, PHP)

		```CRISPR```: Score  obtained only from CRISPR hits
		</details>
	* 3) Potential contamination of the host MAG

	Tip: This might be a good addition for your protocol...
