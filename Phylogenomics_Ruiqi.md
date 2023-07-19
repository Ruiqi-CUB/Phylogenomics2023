
# Phylogenomics Workshop
Ruiqi Li | July 20th 2023

Ruiqi.Li@Colorado.edu

------
## Content

1. Simplified Workflow

  1.1 Genome/Transcriptome Assembly

  1.2 OrthoFinder (Ortholog Identification, Alignment, Gene Tree and species Tree Inference)

2. Manual Workflow

  2.1 Genome/Transcriptome/Target Capture Assembly

  2.2 Alignment with *mafft*

  2.3 Trimming with *trimal*

  2.4 Concatenation with *catfasta2phyml*

  2.5 Phylogeny Inference with *raxml*

3. Practice

4. Conda Setup

  ------


### 1. Simplified Workflow

Software Overview

|software| Tutorial |
| :---: | :---: |
| Trinity | [RNA-Seq De novo Assembly Using Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) |
| TransDecoder | [Find Coding Regions Within Transcripts](https://github.com/TransDecoder/TransDecoder/wiki) |
| OrthoFinder | [Step-by-Step OrthoFinder Tutorials](https://davidemms.github.io/menu/tutorials.html) |

#### 1.1 Genome/Transcriptome Assembly.

We won't cover details about Genome/Transcriptome Assembly and Annotation. Let's assume that we have Transcirptome data from 10 samples.

Assemble RNA-Seq data: ` Trinity --seqType fq --left reads_1.fq --right reads_2.fq --CPU 6 --max_memory 20G `

output: `trinity_out_dir/Trinity.fasta`
Next We will need to annotate the transcriptomes with Transdecoder etc., then we can get 10 annotated transcirptomes (protein sequences): `Sample1.fasta`, `Sample2.fasta` ... `Sample10.fasta`



#### 1.2 OrthoFinder (Ortholog Identification, Alignment, Gene Tree and species Tree Inference)

OrthoFinder is primarily used for identifying orthologs, but there are a lot more to explore in the Ortholog outputs. We can get a [greedy consensus tree](https://www.plants.ox.ac.uk/publication/896690/europe-pubmed-central) from gene trees from each orthogroup, or a tree inferred from the multiple sequnece alignment concatenated from alignments of single-copy orthogroups.

First, we will need to put all the transcirptome data in one directory `MyData`. If you use `ls MyData` to check which files are in this directory, your will see `Sample1.fasta`, `Sample2.fasta` ... `Sample10.fasta`

To get a greedy consensus tree: `orthofinder -f MyData/`. The tree will be in the directory `Results/Species_Tree/SpeciesTree_rooted.txt`



To get a multiple sequnece alignment tree:  `orthofinder -f MyData/ -M msa`

(I need to test if it gives us a species tree directly)




### 2. Manual Workflow

Software Overview

|software| Tutorial |
| :---: | :---: |
| mafft | [Multiple Sequence Alignment](https://github.com/mmatschiner/tutorials/blob/master/multiple_sequence_alignment/README.md) |
| trimal | [trimAl Main Page](https://vicfero.github.io/trimal/) |
| catfasta2phyml | [Concatenate FASTA alignments to PHYML, PHYLIP, or FASTA format](catfasta2phyml) |
| raxml | [RAxML hands-on session](https://cme.h-its.org/exelixis/web/software/raxml/hands_on.html) |


#### 2.1 Genome/Transcriptome/Target Capture Assembly and Annotation

Genome/Transcriptome assmebly and annotation are the same as the previuous section. Basically we are repeating the steps in the OrthoFinder. Because we don't need to identify orthogroups from Target Capture sequencing data, this manual method is particularly useful for Target Capture sequencing.

Assemble Target Capture Data:



#### 2.2 Alignment with *mafft*

From the OrthoFinder Outputs, we can find sequence of each Orthogroup, we will use single copy orthologs for the downstream analyses:

Alignment: `mafft Orthogroup1.fasta > Orthogroup1.output`

Output: `Orthogroup1.output`

#### 2.3 Trimming with *trimal*

Trimming: `trimal -in Orthogroup1.output -out Orthogroup1_trimmed.output -automated1`

Output: `Orthogroup1_trimmed.output`

#### 2.4 Concatenation with *catfasta2phyml*

Concatenation: `./catfasta2phyml/catfasta2phyml.pl --concatenate --verbose --fasta *_trimmed.output > MultipleSequenceAlignment.fasta`

Output: `MultipleSequenceAlignment.fasta`

#### 2.5 Phylogeny Inference with *raxml*

Get a best tree for bootstraping: `raxmlHPC-PTHREADS -T 36 -p 12345 -N 20 -s Nov2022_${genus}_${value}.fasta -m GTRCAT -n run1`

100 bootstraps: `raxmlHPC-PTHREADS -T 36 -p 12345 -b 12345 -s Nov2022_${genus}_${value}.fasta -N 100 -m GTRCAT -n run2`

Get the ML tree: `raxmlHPC-PTHREADS -T 36 -f b -t RAxML_bestTree.run1  -z RAxML_bootstrap.run2 -m GTRCAT -n Final`

Output: maximum likelihood tree with 100 bootstraps ``

``

### 3. Practice

Due to computing power restrictions, we will use 2 genes (in practice, there will be hundreds or thousands) to demonstrate the pipeline. Assuming those 2 genes are from OrthoFinder or Target Capture sequencing results, we will align, trim both genes respectively, then concatenate the alignments, infer the phylogeny with raxml. In practice, we will write bash scripts with loops to automate the process, but in this practice, we will do them one by one.


I need to find example dataset later.     



### 4. Software set-up

#### 4.1 conda set-up

#### 4.2 install softwares with `conda install`

#### 4.3 install softwares with github
