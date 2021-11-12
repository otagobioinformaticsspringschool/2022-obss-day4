---
title: "Introduction to Qiime"
teaching: 10
exercises: 5
questions:
- "How do I import today's results into Qiime?"
objectives:
- "Identify the main components of metabarcoding and how to integrate them into a Qiime workflow"
keypoints:
- "Once you import your results into Qiime, you can run any analysis with that program"
- "The OTU fasta file that we created today are equivalent to 'rep-seqs' (representative sequences) in Qiime"
- "The frequency table we created is called a 'FeatureTable' in Qiime parlance"
---


## Overview 

The Qiime2 package is an open-source system that incorporates multiple, stand-alone programs, giving you many options to run your analysis. The different programs are provided as plug-ins to provide maximum flexibility. The <a href="https://docs.qiime2.org/2021.4/" target="_blank" rel="noopener noreferrer"><b>Qiime2 docs webpage</b></a> has all the information you need to get started on processing metabarcoding data. There are multiple tutorials available, including a <a href="https://docs.qiime2.org/2021.4/tutorials/overview/" target="_blank" rel="noopener noreferrer"><b>detailed overview</b></a> of the available plugins and links to the concepts involved. 

It is possible to run the entirety of your analysis within the Qiime2 system, but today we will just show how to import the results of the earlier lessons into the Qiime format, and then you will be able to follow most of the tutorials on the website. 

First, here is a quick guided tour of the system. 

<br>


Here is an overview of the Qiime2 workflow:

![alt text](../fig/qiime2pipeline.png)

<br>

One of the main concepts of using Qiime is that all data is represented by what they refer to as <a href="https://docs.qiime2.org/2021.4/concepts/#data-files-qiime-2-artifacts" target="_blank" rel="noopener noreferrer"><i>"artifacts"</i></a>, which are actually compressed folders that contain the data and metadata about the data. This is done so that all data files are tracked and logged through the system. In order to use Qiime data must be imported into their system; as well, to use the outputs of Qiime in other programs they must be exported (though some visuals provide a way to download flat files of some outputs). 


![alt text](../fig/qii2datatypes.png)

<br>

Often you do not want to start at the beginning or finish at the end of a pipeline. Once you are able to import today's results, you will be able to start where you wish.

![alt text](../fig/customWorkflow.png)


<br><br>

## Tour of Qiime2 website

Here is a link to the resources on the <a href="https://docs.qiime2.org/2021.4/" target="_blank" rel="noopener noreferrer"><b>Qiime2 website</b></a> 
![alt text](../fig/quickTour.png)

<br><br>



## Getting started: importing files into Qiime2 format

In the command line, we will first load the Qiime2 module:


```bash
$ module purge
$ module load QIIME2/2021.4
```

To see the options for any Qiime2 command, you can use help:

```bash
$ qiime tools import --help
```



There are just a few arguments for importing files, but many possible types and formats. Two of the commands are sort of help commands in that they show what is possible to import.

We can view those by just entering the option:

```bash
$ qiime tools import --show-importable-types
```

Sometimes it is necessary to specify the format of the input file. We can view those as well:

```bash
$ qiime tools import --show-importable-formats
```

These two arguments provide a good guide when you are trying to figure out the kind of file you are importing


## Importing OTU fasta file and frequency table

To import the outputs of the Denoising and clustering lesson


```bash
module load QIIME2/2021.4
```

The Qiime import command (substitute the name of your otu file):


```bash
qiime tools import \
  --type 'FeatureData[Sequence]' \
  --input-path otus.fasta  \
  --output-path otus.qza
```




If the command/script worked then you should have a file called `otus.qza` in your `/otus` folder. All Qiime artifacts end in `.qza`, and all visuals end in `.qzv`.



We cannot import the frequency table directly into Qiime. First we have to convert our table into the `biom` format. Then the biom format will be imported to Qiime. Create a script called `import_freq_table_to_qiime.sh` and then put these commands:

```
biom convert -i otu_frequency_table.tsv \
 -o fish_frequency.biom \
 --to-hdf5

qiime tools import \
  --input-path fish_frequency.biom \
  --type 'FeatureTable[Frequency]' \
  --input-format BIOMV210Format \
  --output-path otu_frequency_table.qza

```

The output of this script should be two files: a `.biom` file and a qiime `.qza` frequency table file. 

## Importing taxonomy files into Qiime

import taxonomy output (with converter script)

import reference files


## Using your imported files in Qiime

Now that you have your 


### Taxonomy assignment with Qiime

<a href="https://docs.qiime2.org/2021.8/tutorials/moving-pictures/#taxonomic-analysis" target="_blank" rel="noopener noreferrer"><b>basic taxonomic analysis tutorial</b></a>

### Diversity analyses with Qiime

<a href="https://docs.qiime2.org/2021.8/tutorials/moving-pictures/#alpha-and-beta-diversity-analysis" target="_blank" rel="noopener noreferrer"><b>Alpha and beta diversity analysis tutorial</b></a>



#### Create a phylogeny from OTUs

There is one last example script. This is to align and create a phylogenetic tree from the OTUs. This can be done with separate commands, but Qiime provides a pipeline that 1) aligns the OTUs, 2) mask uninformative or ambiguous sites in the alignment, 3) infers a phylogenetic tree from the alignment, and then 4) roots the tree at the midpoint. You can check all the options at the <a href="https://docs.qiime2.org/2021.4/plugins/available/phylogeny/align-to-tree-mafft-fasttree/" target="_blank" rel="noopener noreferrer"><b>Plugin page</b></a>. You can also run these commands separately. There are other options for phylogeny, including inferring phylogeny with the **iqtree** and **RaXMl** programs. Have a look at all the options <a href="https://docs.qiime2.org/2021.4/plugins/available/phylogeny/" target="_blank" rel="noopener noreferrer"><b>on the main Phylogeny plugin page</b></a>. These tools can come in handy for multiple applications.  

```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences otus.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

Later we will learn how to export Qiime-formatted artifacts. 

{% include links.md %}
