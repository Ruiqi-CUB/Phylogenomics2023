
# Phylogenomics Module
Ruiqi Li | Oct 27th 2025

liruiqi@usc.edu

------
## Content

## 1. Simplified Workflow

####  1.1 Genome/Transcriptome Assembly

####  1.2 OrthoFinder (Ortholog Identification, Alignment, Gene Tree and species Tree Inference)

## 2. Manual Workflow

####  2.1 Genome/Transcriptome/Target Capture Assembly

####  2.2 Alignment with *mafft*

####  2.3 Trimming with *trimal*

####  2.4 Concatenation with *catfasta2phyml*

####  2.5 Phylogeny Inference with *raxml*

## 3. Practice

## 4. Software Setup

  ------


## 1. Simplified Workflow

Software Overview

|software| Tutorial |
| :---: | :---: |
| Trinity | [RNA-Seq De novo Assembly Using Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) |
| TransDecoder | [Find Coding Regions Within Transcripts](https://github.com/TransDecoder/TransDecoder/wiki) |
| OrthoFinder | [Step-by-Step OrthoFinder Tutorials](https://davidemms.github.io/menu/tutorials.html) |

### 1.1 Genome/Transcriptome Assembly.

We won't cover details about Genome/Transcriptome Assembly and Annotation. Let's assume that we have Transcirptome data from 10 samples.

Assemble RNA-Seq data: ` Trinity --seqType fq --left reads_1.fq --right reads_2.fq --CPU 6 --max_memory 20G `

output: `trinity_out_dir/Trinity.fasta`
Next We will need to annotate the transcriptomes with Transdecoder etc., then we can get 10 annotated transcirptomes (protein sequences): `Sample1.fasta`, `Sample2.fasta` ... `Sample10.fasta`



### 1.2 OrthoFinder (Ortholog Identification, Alignment, Gene Tree and species Tree Inference)

OrthoFinder is primarily used for identifying orthologs, but there are a lot more to explore in the Ortholog outputs. We can get a [greedy consensus tree](https://www.plants.ox.ac.uk/publication/896690/europe-pubmed-central) from gene trees from each orthogroup, or a tree inferred from the multiple sequnece alignment concatenated from alignments of single-copy orthogroups.

First, we will need to put all the transcirptome data in one directory `ExampleData`. If you use `ls ExampleData` to check which files are in this directory, your will see `Sample1.fasta`, `Sample2.fasta` ... `Sample10.fasta`.


**The following steps are time-consuming.** You don't need to practice, but your can have a look at the results in your linux server folder.

Please `cd /home/ruiqi/ruiqi_data/PhylogenomicsWorkshop/ExampleData` to view the results.

To get a greedy consensus tree: `orthofinder -f ExampleData/`. The tree will be in the directory `ExampleData/OrthoFinder/Results/Results_Jul19/Species_Tree/SpeciesTree_rooted.txt`


To get a multiple sequnece alignment tree:  `orthofinder -f ExampleData/ -M msa`. The tree will be in the directory `ExampleData/OrthoFinder/Results/Results_Jul19_1/Species_Tree/SpeciesTree_rooted.txt`

To view the tree without downloading the files: cat `SpeciesTree_rooted.txt` then view it on [etetoolkit.org](http://etetoolkit.org/treeview/)



----

## 2. Manual Workflow

Software Overview

|software| Tutorial |
| :---: | :---: |
| mafft | [Multiple Sequence Alignment](https://github.com/mmatschiner/tutorials/blob/master/multiple_sequence_alignment/README.md) |
| trimal | [trimAl Main Page](https://vicfero.github.io/trimal/) |
| catfasta2phyml | [Concatenate FASTA alignments to PHYML, PHYLIP, or FASTA format](https://github.com/nylander/catfasta2phyml) |
| raxml | [RAxML hands-on session](https://cme.h-its.org/exelixis/web/software/raxml/hands_on.html) |


### 2.1 Genome/Transcriptome/Target Capture Assembly and Annotation

Genome/Transcriptome assmebly and annotation are the same as in the previous section. Basically we are manually repeating the steps as in the OrthoFinder. Because we don't need to identify orthogroups from Target Capture sequencing data, this manual method is particularly useful for Target Capture sequencing.

Assemble Target Capture Data:

[HybPiper](https://github.com/mossmatters/HybPiper) was designed for targeted sequence capture, in which DNA sequencing libraries are enriched for gene regions of interest, especially for phylogenetics. HybPiper is a suite of Python scripts/modules that wrap and connect bioinformatics tools in order to extract target sequences from high-throughput DNA sequencing reads. We won't cover details in targeted  capture sequence asssmbly today.



### 2.2 Alignment with *mafft*

This step arranges protein (or DNA) sequences to identify regions of similarity that may be a consequence of evolutionary relationships between the sequences. The input data for alignments could be either single copy orthologs from the OrthoFinder outputs `Single_Copy_Orthologue_Sequences/OGXXXXXXX.fa`, or assembled targeted capture sequence:

Alignment: `mafft Orthogroup1.fasta > Orthogroup1.output`

Output: `Orthogroup1.output`

### 2.3 Trimming with *trimal*

trimAl is a tool for the automated removal of spurious sequences or poorly aligned regions from a multiple sequence alignment.

Trimming: `trimal -in Orthogroup1.output -out Orthogroup1_trimmed.output -automated1`

Output: `Orthogroup1_trimmed.output`

### 2.4 Concatenation with *catfasta2phyml*

catfasta2phyml.pl will concatenate FASTA alignments to one file (interleaved PHYML or FASTA format) after checking that all sequences are aligned (of same length). If there are sequence labels that are not present in all files, a warning will be issued. Sequenced can, however, still be concatenated (and missing sequences be filled with missing data (gaps)) if the argument --concatenate is used.

Concatenation: `./catfasta2phyml/catfasta2phyml.pl --concatenate --verbose --fasta *_trimmed.output > MultipleSequenceAlignment.fasta`

Output: `MultipleSequenceAlignment.fasta`

### 2.5 Phylogeny Inference with *raxml*

RAxML (Randomized Axelerated Maximum Likelihood) is a popular program for phylogenetic analysis of large datasets under maximum likelihood. Its major strength is a fast maximum likelihood tree search algorithm that returns trees with good likelihood scores.

This command will generate 20 ML trees on distinct starting trees and also print the tree with the best likelihood to a file called `RAxML_bestTree.run1`:

`raxmlHPC -p 12345 -N 20 -s MultipleSequenceAlignment.fasta -m GTRCAT -n run1`

We need to tell RAxML that we want to do bootstrapping by providing a bootstrap random number seed via -b 12345 and the number of bootstrap replicates we want to compute via `-N 100`.:

`raxmlHPC -p 12345 -b 12345 -s MultipleSequenceAlignment.fasta -N 100 -m GTRCAT -n run2`.

Having computed the bootstrap replicate trees that will be printed to a file called `RAxML_bootstrap.run2`

we can now use them to draw bipartitions on the best ML tree as follows: `raxmlHPC -f b -t RAxML_bestTree.run1  -z RAxML_bootstrap.run2 -m GTRCAT -n Final`

This call will produce to output files that can be visualized: `RAxML_bipartitions.Final`

Then you can visualize it [online](http://etetoolkit.org/treeview/) or with any local software you choose (e.g. MEGA, FigTree)

----


## 3. Practice

### 3.1 Data

Due to computing power limitations, we will use 2 genes (in practice, there will be hundreds or thousands) to demonstrate the pipeline. Assuming those 2 genes are from OrthoFinder or Target Capture sequencing assembly, we will align, trim both genes respectively, then concatenate the alignments, infer the phylogeny with raxml. In the real world, we will write bash scripts with loops to automate the process, but in this practice, we will do them one by one.


We will use the sequence data from [Matschiner et al. (2017)](https://academic.oup.com/sysbio/article/66/1/3/2418030?login=true#87816111). The dataset used here includes sequences for two genes; the mitochondrial 16S gene coding for [16S ribosomal RNA](https://en.wikipedia.org/wiki/16S_ribosomal_RNA) and the [nuclear RAG1](https://en.wikipedia.org/wiki/RAG1) gene coding for recombination activating protein 1. The sequences represent the 9 species listed in the table below. You will find the sequence file `rag1.fasta` and `16s.fasta` on the workshop Github page.

|ID|Species|Group|
|---|---|---|
|Acanthapomotis|	*Acantharchus pomotis* |	Non-cichlid|
|Acropomjaponic|	*Acropoma japonicum* |	Non-cichlid |
|Etroplumaculat|	*Etroplus maculatus* |	Indian cichlids|
|Maylandzebraxx|	*Maylandia zebra* |	African cichlids|
|Neolampbrichar|	*Neolamprologus brichardi* |	African cichlids|
|Oreochrnilotic|	*Oreochromis niloticus* |	African cichlids|
|Pundaminyerere|	*Pundamilia nyererei* |	African cichlids|
|Tylochrpolylep|	*Tylochromis polylepis* |	African cichlids|
|Ectodusdescamp|	*Ectodus descampsii* |	African cichlids |


Please set up your laptop following instructions in *4. Software Installation* before the following steps. Use `conda activate phylogeny` to activate the environment.


### 3.2 Alignment


Copy data yo your current directory using `cp -r /home/ruiqi/ruiqi_data/PhylogenomicsWorkshop/Data ./`

Use [`cd`](https://phoenixnap.com/kb/linux-cd-command) to navigate to the directory with the data. Use [`ls`](https://www.freecodecamp.org/news/the-linux-ls-command-how-to-list-files-in-a-directory-with-options/) to list files and directories in the current directory.


Alignment:

```
mafft rag1.fasta > rag1.output
mafft 16s.fasta > 16s.output
```

Output: `rag1.output` and `16s.output`

### 3.3 Trimming with *trimal*

Trimming:

```
trimal -in rag1.output -out rag1_trimmed.output -automated1
trimal -in 16s.output -out 16s_trimmed.output -automated1
```
Output: `rag1_trimmed.output` and `16s_trimmed.output`

### 3.4 Concatenation with *catfasta2phyml*
If the `catfasta2phyml.pl` script is in the current directory, you should use `./catfasta2phyml.pl` instead.

Concatenation: `./catfasta2phyml/catfasta2phyml.pl --concatenate --verbose --fasta *_trimmed.output > MultipleSequenceAlignment.fasta`

Output: `MultipleSequenceAlignment.fasta`

### 3.5 Phylogeny Inference with *raxml*

This command will generate 20 ML trees on distinct starting trees and also print the tree with the best likelihood to a file called `RAxML_bestTree.run1`:

`raxmlHPC -p 12345 -N 20 -s MultipleSequenceAlignment.fasta -m GTRCAT -n run1`

We need to tell RAxML that we want to do bootstrapping by providing a bootstrap random number seed via -b 12345 and the number of bootstrap replicates we want to compute via `-N 100`.:

`raxmlHPC -p 12345 -b 12345 -s MultipleSequenceAlignment.fasta -N 100 -m GTRCAT -n run2`.

Having computed the bootstrap replicate trees that will be printed to a file called `RAxML_bootstrap.run2`

we can now use them to draw bipartitions on the best ML tree as follows: `raxmlHPC -f b -t RAxML_bestTree.run1  -z RAxML_bootstrap.run2 -m GTRCAT -n Final`

This call will produce to output files that can be visualized: `RAxML_bipartitions.Final`



Then you can use `cat RAxML_bipartitions.Final` to print the results in the terminal, then visualize [online](http://etetoolkit.org/treeview/) or with any local software you choose (e.g. MEGA, FigTree)


Alternatively, you could use IQtree2 (you will need to install it first in session 4). `-m MFP` is automatic model selection; `-B 100` means 100 bootstraps; `-T 4` means using 4 threads/CPUs.

`iqtree2 -s MultipleSequenceAlignment.fasta -m MFP -B 1000 -T 4`

view the consensus tree in file `MultipleSequenceAlignment.fasta.contree`

----

## 4. Software Installation

### 4.1 conda setup

Follow the [instructions](https://docs.conda.io/en/latest/miniconda.html#installing) here to install miniconda3.


```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
```

Reopen your terminal and you will see *(base)* on the left to the prompt in the terminal if installed correctly.

Then you will need to set up Bioconda, see detailed [instructions here](https://bioconda.github.io/).

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict
```


### 4.2 install softwares

Create environment: `conda create --name phylogeny`

Activate environment: `conda activate phylogeny`

install software used in the workshop: `conda install -c bioconda orthofinder trimal raxml mafft`

If you want to use iqtree2 `conda install bioconda::iqtree`

Install software with Github:`git clone https://github.com/nylander/catfasta2phyml.git` or go to the [Github page](https://github.com/nylander/catfasta2phyml/blob/master/catfasta2phyml.pl) and click Download from the dropdown menu of **...** in the top right corner.
