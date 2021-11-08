---
title: "Taxonomy Assignment"
teaching: 15
exercises: 30
questions:
- "What are the main strategies for assigning taxonomy to OTUs"
- "What is the difference between Sintax and Naive Bayes"
objectives:
- "Assign taxonomy using Sintax"
- "Use `less` to explore a long help output"
- "Write a script to assign taxonomy"
keypoints:
- "The main taxonomy assignment strategies are global alignments, local alignments, and machine learning approaches"
- "The main difference is that Naive Bayes requires the reference data to be trained, whereas Sintax looks for only kmer-based matches"
---

# Taxonomy Assignment: Different Approaches

There are three basic approaches to taxonomy classification (and endless variations of each of these): Global alignment, local alignment, and Naive Bayes, or machine learning approaches in general. 

![alt text](../fig/methodComparison.png)

for a discussion of them see the <a href="https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-018-0470-z" target="_blank" rel="noopener noreferrer"><b>paper</b></a>.


## Aligment-Based Methods

BLAST is one of the most common methods of searching DNA sequences. BLAST is an acronym for Basic Local Alignment Search Tool. It is called this because it looks for any short, matching sequence, or *local alignments*, with the reference. This is contrasted with a *global alignment*, which tries to find the best match across the entirety of the sequence.  


![alt text](../fig/globalVlocal_image.png)

<br>

Though BLAST is widely used, it is not necessarily the best way to search for matching sequences in metabarcoding studies. 

<br><br>

## Using Machine Learning to Assign Taxonomy

Another way to assign taxonomy utilises machine learning algorithms. 

![alt text](../fig/machineLearningExamples.png)

<br><br>



## Use the program Sintax to classify our OTUs

The program <a href="https://www.drive5.com/usearch/manual/cmd_sintax.html" target="_blank" rel="noopener noreferrer"><b>Sintax</b></a> is similar to the Naive Bayes classifier used in Qiime and other programs, but is simplified and can run much faster. Nevertheless, we have found it as accurate as Naive Bayes for the fish dataset we are using in this workshop. As with the previous steps, we will be using a bash script to run the classification. As before, we will use VSEARCH to run Sintax.

Go to the `scripts` folder and create a file called *classify_sintax.sh*, using either Nano (`nano classify_sintax.sh`) or from the Launcher in Jupyter. 

The VSEARCH command has many functions and many options. In the command line, you can list these more easily using `less`:

```
vsearch -h | less
```

Looking at a long help command this way allows you to scroll up and down the help options. For the sintax script, you will need to fill in these parameters:

- `--sintax`
- `--db`
- `--sintax_cutoff`
- `--tabbedout`

> ## Write a script to run sintax through VSEARCH
> 
> Hints:
> 
> Start to fill in the neccessary components. We will go through each of the arguments.
> 
> When you start searching with `less`, you can quit anytime by just entering `q`. 
> 
> You can also search for a specific part of the help output by entering a forward slash (`/`), followed by the term you are looking for
> 
> You can go to top of the search by entering `gg`, and to the bottom by entering `G`.
> 
> Try to figure out which parameter you need to fill in for each of these options. 
> 
> To get you started, the `--sintax` argument has to go right after `vsearch`, and its parameter is the name of the OTU file you created in the last lesson
>
>> ## Solution
>> 
>> ~~~bash
>> module load eDNA
>> 
>> cd ../taxonomy
>>
>> vsearch --sintax \
>>  ../otus/<OTU_FILE> \
>>  --db ../references/<REFERENCE-FILE> \
>>  --sintax_cutoff 0.8 \
>>  --tabbedout <OUTPUT-FILE>
>> ~~~
>> After the first line, we just put the name of the OTU file from the clustering lesson, remember the path is needed to point the program to where the file is
>>
>> On the next line, after the `--db`, put the path `../references/` and the reference file. 
>>
>> The next line is for the confidence cut off. If the estimated confidence of a taxonomic assignment falls below this number, the assignment will not be recorded. >> The default for this is 0.6, but we will use a higher cutoff of `0.8` for the `--sintax_cutoff` argument.
>>
>> The final line is to name the output file. You can use any name, but it is better to include some information such as the name of the OTUs and program used    >> (sintax), in case want to compare the results with other runs. Also include the suffix `.tsv`, which will tell Jupyter that it is a table file.
>>
> {: .solution}
{: .challenge}


Now you are done. Save and close the file. Make the script executable and run it on the command line.

```
chmod a+x classify_sintax.sh

./classify_sintax.sh
```

### 

Have a look at the output file. Find the file in the taxonomy folder and double click to open it in Jupyter. You will see it is divided into two main parts. The first includes the name of each rank (e.g. *p:Chordata* to indicate the phylum Chordata), followed by the confidence for the assignment at that rank in parentheses. The second section, which follows the *+*, has only the name of each rank and includes only those ranks whose confidence is above the `--sintax_cutoff` parameter used. 


> ## Study Questions
>
> Name an OTU that would have better resolution if the cutoff were set to default
> 
> Name an OTU that would have worse resolution if the cutoff were set higher
> 
{: .challenge}


## Visualisation in Qiime

In Qiime, all the visuals are run as interactive web apps--they need to run on a web browser. In this course we will focus on creating graphs and plots using R. However, we have provided some links to visuals for the OTUs and taxonomy for this data set, in order to show how some of the visuals work. 

For all these plots, they need to be opened on a webpage called Qiime2View. Click on the following link to open it in a new tab:

<a href="https://view.qiime2.org/" target="_blank" rel="noopener noreferrer"><b>Qiime2View webpage</b></a>

To see a table of the taxonomy assignment of each OTU, paste the following link into the Qiime2View webpage (do not click on it).

**https://otagoedna.github.io/2019_11_28_edna_course/example_viz/fish_NB_taxonomy_VIZ.qzv**

Then go to the Qiime2View website, click on 'file from the web', and paste the link in the box that opens up.

The next plot is a barplot graph of the taxonomy. A barplot graph is a good way to compare the taxonomic profile among samples. 

Paste the following link into the Qiime2View page:

**https://otagoedna.github.io/2019_11_28_edna_course/example_viz/fish_NB_taxonomy_barplot.qzv**

We are also including a table of the OTU sequences.

**https://otagoedna.github.io/2019_11_28_edna_course/example_viz/zotus_tabulation.qzv**

This visual shows the sequence for each OTU. You can run a BLAST search on each OTU by clicking on the sequence. However, the search can take a while, depending on how busy the NCBI server is.

<br><br>

## Converting the output file

While the sintax output file is very informative, we will convert it to a simpler format that will make it easier to import into other downstream applications. Fortunately, the eDNA module has a script that will convert the file into several other formats. At the terminal prompt, go to the taxonomy folder and enter the script name and the help argument

```bash
convert_tax_format.py -h
```

You will see the options:

```
-i file to convert
-o output file
-f output format
```
{: .output}

This program will run very fast, so you can run it on the command line. Check the output. You should see a new table with each taxonomic rank in its own column. (**Hint:** if you name the output file with `.tsv` at the end, then it will open in Jupyter as a table.)


<br><br>



<br><br>

>## Extra exercise: run BLAST taxonomy assignment
>
> Qiime2 has two other methods for taxonomy assignment. If you have some time, you can run BLAST as well to find taxonomy. 
>
> https://docs.qiime2.org/2021.4/plugins/available/feature-classifier/classify-consensus-blast/
>
> Hints:
>
> - You will not use the trained data, instead for `--i-reference-reads` use the fasta.qza, and for `--i-reference-taxonomy` use the taxon.qza file. Both are in the reference folder
>
> - Add the options `--p-perc-identity` and `--p-query-cov`, and try increasing the value for these. 
>
> - Run it as a Slurm job. Increase the time given to 30 minutes.
{: .challenge}


{% include links.md %}
