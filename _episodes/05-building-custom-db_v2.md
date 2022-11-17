---
title: "Creating A Reference Database"
teaching: 15
exercises: 30
questions:
- "What reference databases are available for metabarcoding"
- "How do I make a custom database?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

# Reference databases

## Introduction

As we have previously discussed, eDNA metabarcoding is the process whereby we amplify a specific gene region of the taxonomic group of interest from an environmental sample. Once the DNA is amplified, we sequence it and up till now, we have been filtering the data based on quality and assigned sequences to samples. The next step in the bioinformatic process will be to assign a taxonomy to each of the sequences, so that we can determine what species are detected through our eDNA analysis.

There are multiple ways to assign taxonomy to a sequence. While this workshop won't go into too much detail about the different taxonomy assignment methods, any method used will require a reference database.

Several reference databases are available online. The most notable ones being NCBI, EMBL, BOLD, SILVA, and RDP. However, recent research has indicated a need to use custom curated reference databases to increase the accuracy of taxonomy assignment. While certain reference databases are available (RDP: 16S microbial; MIDORI: COI eukaryotes; etc.), this necessary data source is missing in most instances. We will, therefore, show you how to build your own custom curated reference database using a program we developed and was recently accepted for publication in Molecular Ecology Resources, called CRABS: Creating Reference databases for Amplicon-Based Sequencing. For this workshop, we'll be using an older version of CRABS that is installed on NeSI. However, if you'd like to try making your own reference database for your own study, we suggest to install it on your computer via Docker, conda, or GitHub (<a href="https://github.com/gjeunen/reference_database_creator" target="_blank" rel="noopener noreferrer"><b>CRABS (EMP)</b></a>).

## CRABS - a new program to build custom reference databases

The CRABS workflow consists of seven parts, including:

- (i) downloading and importing data from online repositories into CRABS;
- (ii) retrieving amplicon regions through *in silico* PCR analysis;
- (iii) retrieving amplicons with missing primer-binding regions through pairwise global alignments using results from *in silico* PCR analysis as seed sequences;
- (iv) generating the taxonomic lineage for sequences;
- (v) curating the database via multiple filtering parameters;
- (vi) post-processing functions and visualizations to provide a summary overview of the final reference database; and
- (vii) saving the database in various formats covering most format requirements of taxonomic assignment tools.

Today, we will build a custom curated reference database of shark (Chondrichthyes) sequences from the 16S gene region. The reason why we're focusing on sharks in this tutorial, is that the dataset needed to be downloaded from NCBI is small and won't overload the NCBI servers if we download sequences all together. Since our sample data includes both fish and sharks, we have placed a CRABS-generated reference databases containing fish and sharks already in the `refdb` folder that we will be using during taxonomy assignment.

Before we get started, we will quickly go over the help information of the program. As before, we will first have to source the `moduleload` script to load the program.

```bash
source moduleload
crabs -h
```

```
usage: crabs [-h] [--version] {db_download,db_import,db_merge,insilico_pcr,pga,assign_tax,dereplicate,seq_cleanup,db_subset,visualization,tax_format} ...

creating a curated reference database

positional arguments:
  {db_download,db_import,db_merge,insilico_pcr,pga,assign_tax,dereplicate,seq_cleanup,db_subset,visualization,tax_format}

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
```
{: .output}

As you can see, the help documentation shows you how to use the program, plus displays all the functions that are currently available within CRABS. To display the help documentation of a function, we can type the following command:

```bash
crabs db_download -h
```

```
usage: crabs db_download [-h] -s SOURCE [-db DATABASE] [-q QUERY] [-o OUTPUT] [-k ORIG] [-d DISCARD] [-e EMAIL] [-b BATCHSIZE]

downloading sequence data from online databases

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        specify online database used to download sequences. Currently supported options are: (1) ncbi, (2) embl, (3) mitofish, (4) bold, (5) taxonomy
  -db DATABASE, --database DATABASE
                        specific database used to download sequences. Example NCBI: nucleotide. Example EMBL: mam*. Example BOLD: Actinopterygii
  -q QUERY, --query QUERY
                        NCBI query search to limit portion of database to be downloaded. Example: "16S[All Fields] AND ("1"[SLEN] : "50000"[SLEN])"
  -o OUTPUT, --output OUTPUT
                        output file name
  -k ORIG, --keep_original ORIG
                        keep original downloaded file, default = "no"
  -d DISCARD, --discard DISCARD
                        output filename for sequences with incorrect formatting, default = not saved
  -e EMAIL, --email EMAIL
                        email address to connect to NCBI servers
  -b BATCHSIZE, --batchsize BATCHSIZE
                        number of sequences downloaded from NCBI per iteration. Default = 5000
```
{: .output}

### Step 1: download sequencing data from online repositories

As a first step, we will download the 16S shark sequences from NCBI. While there are multiple repositories supported by CRABS, we'll focus on this one today.

```bash
crabs db_download -s ncbi -db nucleotide -q '16S[All Fields] AND ("Chondrichthyes"[Organism] OR Chondrichthyes[All Fields])' -o sharks_16S_ncbi.fasta -e johndoe@gmail.com
```

Let's look at the top 2 lines of the downloaded fasta file to see what CRABS created:

```bash
head -2 sharks_16S_ncbi.fasta
```

```
>LC738810
ATCCCAGTGAGATCGCCCCTACTCATCCTTTAAGAAAACAGGGAGCAGGTATCAGGCACGCCCCTATCAGCCCAAGACACCTTGCTTAGCCACGCCCCCAAGGGACTTCAGCAGTGATTAATATTAGATAATGAACGAAAGTTTGATCTAGTTAAGGTTATTAGAGCCGGCCAACCTCGTGCCAGCCGCCGCGGTTATACGAGCGGCCCAAATTAATGAGATAACGGCGTAAAGAGTGTATAAGAAAAACCTTCCCCTATTAAAGATAAAATAGTGCCCAACTGTTATACGCACCCGCACCAATGAAATCCATTACAAAAGGAACTTTATTATAACAAGGCCTCTTAAAACACGATAGCTAAAAAACAAACTGGGATTAGATACCCCACTATGTTTAGCCCTAAACCTAGGTGTTTAATTACCCAAACACCCGCCAGAGAACTACAAGCGCCAGCTTAAAACCCAAAGGACTTGGCGGTGCCTTAGACCCCCCTAGAGGAGCCTGTTCTAGAACCGATAATCCACGTTCAACCTCACCACCCCTTGCTCTTTCAGCCTATATACCGCCGTCGCCAGCCTGCCCCATGAGGGTCAAACAGCAAGCACAAAGAATTAAACTCCAAAACGTCAGGTCGAGGTGTAGCCCATGAGGTGGGAAGAAATGAGCTACATTTTCTATTAAGATAAACAGATAAATAATGAAATTAAATTTGAAGTTGGATTTAGCAGTAAGATATAAATAGTATATTAATCTGAAAATGGCTCTGAGGCGCGCACACACCGCCCGTCACTCTCCTTAACAAATTAATTAAATATATAATAAAATACTCAAATAAGAGGAGGCAAGTCGTAACATGGTAAGCGTACTGGAAAGTGCGCTTGGAATCAAAATGTAGTTAAAAAGTACAACACCCCCCTTACACAGAGGAAATATCCATGCAAGTCGGATCATTTTGAACTTTATAGTTAACCCAATACACCCGATTAATTATAATCCCCCCCCAAAACGATTTTTAAAAATTAACCAAAACATTTATTATTCTTAGTATAGGAGATAGAAAAG
```
{: .output}

Here, we can see that the CRABS format is a simple 2-line fasta format with accession numbers as headers and the sequence on the line below.

### Step 2: extract amplicon regions through *in silico* PCR analysis

Once we have downloaded the sequencing data, we can start with a first curation step of the reference database. We will be extracting the amplicon region from each sequence through an *in silico* PCR analysis. With this analysis, we will locate the forward and reverse primer binding regions and extract the sequence in between. This will significantly reduce file sizes when larger data sets are downloaded, while also keeping only the necessary information.

The information on the primer sequences can be found in the text file `crabs_primer_info.txt`. Use the `head` command to visualise the information and copy paste it in the code.

```bash
crabs insilico_pcr -h
```

```
usage: crabs insilico_pcr [-h] -i INPUT -o OUTPUT -f FWD -r REV [-e ERROR] [-t THREADS]

curating the downloaded reference sequences with an in silico PCR

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        input filename
  -o OUTPUT, --output OUTPUT
                        output file name
  -f FWD, --fwd FWD     forward primer sequence in 5-3 direction
  -r REV, --rev REV     reverse primer sequence in 5-3 direction
  -e ERROR, --error ERROR
                        number of errors allowed in primer-binding site. Default = 4.5
  -t THREADS, --threads THREADS
                        number of threads used to compute in silico PCR. Default = autodetection
```
{: .output}

```bash
crabs insilico_pcr -i sharks_16S_ncbi.fasta -o sharks_16S_ncbi_insilico.fasta -f GACCCTATGGAGCTTTAGAC -r CGCTGTTATCCCTADRGTAACT
```

### Step 3: assigning a taxonomic lineage to each sequence

Before we continue curating the reference database, we will need assign a taxonomic lineage to each sequence. The reason for this is that we have the option to curate the reference database on the taxonomic ID of each sequence. For this module, we normally need to download the taxonomy files from NCBI using the `db_download --source taxonomy` function. However, these files are already available to you for this tutorial. Therefore, we can skip the download of these files. CRABS extracts the necessary taxonomy information from these NCBI files to provide a taxonomic lineage for each sequence using the `assign_tax` function.

Let's call the help documentation of the `assign_tax` function to see what parameters we need to fill in.

```bash
crabs assign_tax -h
```

```
usage: crabs assign_tax [-h] -i INPUT -o OUTPUT -a ACC2TAX -t TAXID -n NAME [-w WEB] [-r RANKS] [-m MISSING]

creating the reference database with taxonomic information

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        input file containing the curated fasta sequences after in silico PCR
  -o OUTPUT, --output OUTPUT
                        curated reference database output file
  -a ACC2TAX, --acc2tax ACC2TAX
                        accession to taxid file name
  -t TAXID, --taxid TAXID
                        taxid file name
  -n NAME, --name NAME  phylogeny file name
  -w WEB, --web WEB     retrieve missing taxonomic information through a web search. Default = "no"
  -r RANKS, --ranks RANKS
                        taxonomic ranks included in the taxonomic lineage. Default = "superkingdom+phylum+class+order+family+genus+species"
  -m MISSING, --missing MISSING
                        filename for sequences for which CRABS could not generate a taxonomic lineage (e.g., novel species, typo in species name)
```
{: .output}

```bash
crabs assign_tax -i sharks_16S_ncbi_insilico.fasta -o sharks_16S_ncbi_insilico_tax.tsv -a nucl_gb.accession2taxid -t nodes.dmp -n names.dmp
```

The output file of the `assign_tax` function is a tab-delimited file, whereby taxonomy information and sequence is all encountered on a single line. Let's use the `head` function to determine the structure:

```bash
head -10 sharks_16S_ncbi_insilico_tax.tsv
```

```
OM687170	1206156	Eukaryota	Chordata	Chondrichthyes	Myliobatiformes	Myliobatidae	Rhinoptera	Rhinoptera_brasiliensis	ACTTAAGTTACCTTTAAAATATTAAATTTAACCCCCTTTGGGCATAAATAATAAACTAATTTCCTAACTTAACTTGTTTTTGGTTGGGGCGACCAAGGGGAAAAACAAAACCCCCTTACCGAATGTGTGAGATATCACTTAAAAATTAGAGTCACATCTCTAATCAATAGAAAATCTAACGAACAATGACCCAGGAATCCATTTATCCTGATCAATGAACCA
OP391486	195309	Eukaryota	Chordata	Chondrichthyes	Chimaeriformes	Chimaeridae	Hydrolagus	Hydrolagus_mitsukurii	AATATATTAATAAAAGGAATTTAGAACCCTCAGGGGATAAGAAATGGACTTAGCATAATAAAATTATTTTTGGTTGGGGCAACCACGGGGTATAACCTAACCCCCGTATCGATTGGGCAAAAAATGCCTAAAAAATAGAACGACAGTTCTATTTAATAAAATATTTAACGAGTAACGATCCAGAGATATCTGATTAATGAACCA
NC_066805	68454	Eukaryota	Chordata	Chondrichthyes	Carcharhiniformes	Scyliorhinidae	Scyliorhinus	Scyliorhinus_stellaris	ACATAAATTAATTATGTACATATTAATAATCCCAGGACATAAACAAAAAATATAATATTTCTAATTTAACTATTTTTGGTTGGGGTGACCAAGGGGAAAAATGAATCCCCCTTATCGACCGAGTATTCAAGTACTTAAAAGTTAGAATTACAATTCTAACCGATAAAATTTTTACCGAAAAATGACCCAGGATTTTCCTGATCAATGAACCA
NC_066688	195333	Eukaryota	Chordata	Chondrichthyes	Rhinopristiformes	Rhynchobatidae	Rhynchobatus	Rhynchobatus_djiddensis	ACTTAAGTTAATTACTCCATATAAACCACCCCTCCGGGTATAAAACAACATAAATAAATCTAACTTAACTTGTTTTTGGTTGGGGCGACCAAGGGGAAAAACAAAACCCCCTTATCGATTGAGTATTCAACATACTTAAAAATTAAAACGAATAGTTTTAATTAATAGAACATCTAACGAACAATGACCCAGGACACATCCTGATCAATGAACCA
MN389524	2060314	Eukaryota	Chordata	Chondrichthyes	Myliobatiformes	Dasyatidae	Neotrygon	Neotrygon_indica	ACTTAAGTTACCTTAACACCAAAAATTACACCCCAGGGTATAAAAACTAGAAATAACCTCTAACTTAACTTGTTTTCGGTTGGGGCGACCAAGGAGTAAAACAAAACCCCCTTATCGAATGTGTGAGAAATCACTCAAAATTAGAATTACAATTCTAATTAATAGAAGATCTAACGAACAATGACCCAGGACCTCATCCTGATCAATGAACCA
OM404361	195316	Eukaryota	Chordata	Chondrichthyes	Torpediniformes	Narcinidae	Narcine	Narcine_timlei	AATTAAATTAATAAAACAAGACATTTTAAACGACAGTTATAAACACACACCTTATTTTAATTTAACATTTTTGGTTGGGGCAACCAAGGGGAACAAAAAAACCCCCTTATCGATTAAGTATTATATGTACTTAAAATAATAGAAAGACTTTTCAAATTTACAGAACATCTGACGAAATATGACCCAAGAATCTTCTTGATCAATGAACCA
LC636781	2704970	Eukaryota	Chordata	Chondrichthyes	Myliobatiformes	Dasyatidae	Hemitrygon	Hemitrygon_akajei	ACTTAAGTTATTTTTAAACATCAAAAATTTCCCACCTTCGGGCATAAAATAACATAAAATAAATCTTCTAACTTAACTTGTTTTTGGTTGGGGCGACCGAGGGGTAAAACAAAACCCCCTTATCGAATGGGTGAGAAATCACTCAAAATTAGAACGACAGTTCTAATTAATAGAAAATCTAACGAACAATGACCCAGGAAACTATATCCTGATCAATGAACCA
ON065568	195333	Eukaryota	Chordata	Chondrichthyes	Rhinopristiformes	Rhynchobatidae	Rhynchobatus	Rhynchobatus_djiddensis	ACTTAAGTTAATTACTCCATATAAACCACCCCTCCGGGTATAAAACAACATAAATAAATCTAACTTAACTTGTTTTTGGTTGGGGCGACCAAGGGGAAAAACAAAACCCCCTTATCGATTGAGTATTCAACATACTTAAAAATTAAAACGAATAGTTTTAATTAATAGAACATCTAACGAACAATGACCCAGGACACATCCTGATCAATGAACCA
ON065567	335006	Eukaryota	Chordata	Chondrichthyes	Rhinopristiformes	Rhynchobatidae	Rhynchobatus	Rhynchobatus_australiae	ACTTAAGTTAATTACTCCATATAAACCACCCCTTCGGGCATAAAACAACATAAATAAATCTAACTTAACTTGTTTTTGGTTGGGGCGACCAAGGGGAAAAACAAAACCCCCTTATCGACTGAGTATTCAACATACTTAAAAATTAAAACGAATAGTTTTAATTAATAGAACATCTAACGAACAATGACCCAGGACACACATCCTGATCAATGAACCA
LC723525	68454	Eukaryota	Chordata	Chondrichthyes	Carcharhiniformes	Scyliorhinidae	Scyliorhinus	Scyliorhinus_stellaris	ACATAAATTAATTATGTACATATTAATAATCCCAGGACATAAACAAAAAATATAATATTTCTAATTTAACTATTTTTGGTTGGGGTGACCAAGGGGAAAAATGAATCCCCCTTATCGACCGAGTATTCAAGTACTTAAAAGTTAGAATTACAATTCTAACCGATAAAATTTTTACCGAAAAATGACCCAGGATTTTCCTGATCAATGAACCA
```
{: .output}

> ## Count the number of sequences in the output file of the `assign_tax` function
> Before we continue, let's first determine how many sequences are included in our reference database.
>
> Hint 1: think about the file structure.
> 
>> ## Solution
>> ~~~
>> wc -l sharks_16S_ncbi_insilico_tax.tsv
>> ~~~
>>
>> `wc` is a command that stands for `word count`
>> `-l` indicates we want to count lines, rather than words in our document
> {: .solution}
{: .challenge}

### Step 4: curate the reference database

Next, we will run two different functions to curate the reference database, the first is to dereplicate the data using the `dereplicate` function, the second is to filter out sequences that we would not want to include in our final database using the `seq_cleanup` function.

```bash
crabs dereplicate -i sharks_16S_ncbi_insilico_tax.tsv -o sharks_16S_ncbi_insilico_tax_derep.tsv -m uniq_species
crabs seq_cleanup -i sharks_16S_ncbi_insilico_tax_derep.tsv -o sharks_16S_ncbi_insilico_tax_derep_clean.tsv -e yes -s yes -na 0
```

After curation, we're left with 552 16S shark sequences in our reference database.

### Step 5: export the reference database to a format supported by the taxonomy assignment software

Lastly, we can export the reference database to one of several formats that are implemented in the most commonly used taxonomic assignment tools using the `tax_format` function. Since we'll be using SINTAX to assign taxonomy to our OTUs in the following section, let's export our shark database to SINTAX format as well.

```bash
crabs tax_format -i sharks_16S_ncbi_insilico_tax_derep_clean.tsv -o sharks_16S_ncbi_insilico_tax_derep_clean_sintax.fasta -f sintax
```

Let's open the reference database to see what the final database looks like!

```bash
head sharks_16S_ncbi_insilico_tax_derep_clean_sintax.fasta
```

```
>OM687170;tax=d:Eukaryota,p:Chordata,c:Chondrichthyes,o:Myliobatiformes,f:Myliobatidae,g:Rhinoptera,s:Rhinoptera_brasiliensis
ACTTAAGTTACCTTTAAAATATTAAATTTAACCCCCTTTGGGCATAAATAATAAACTAATTTCCTAACTTAACTTGTTTTTGGTTGGGGCGACCAAGGGGAAAAACAAAACCCCCTTACCGAATGTGTGAGATATCACTTAAAAATTAGAGTCACATCTCTAATCAATAGAAAATCTAACGAACAATGACCCAGGAATCCATTTATCCTGATCAATGAACCA
>OP391486;tax=d:Eukaryota,p:Chordata,c:Chondrichthyes,o:Chimaeriformes,f:Chimaeridae,g:Hydrolagus,s:Hydrolagus_mitsukurii
AATATATTAATAAAAGGAATTTAGAACCCTCAGGGGATAAGAAATGGACTTAGCATAATAAAATTATTTTTGGTTGGGGCAACCACGGGGTATAACCTAACCCCCGTATCGATTGGGCAAAAAATGCCTAAAAAATAGAACGACAGTTCTATTTAATAAAATATTTAACGAGTAACGATCCAGAGATATCTGATTAATGAACCA
>NC_066805;tax=d:Eukaryota,p:Chordata,c:Chondrichthyes,o:Carcharhiniformes,f:Scyliorhinidae,g:Scyliorhinus,s:Scyliorhinus_stellaris
ACATAAATTAATTATGTACATATTAATAATCCCAGGACATAAACAAAAAATATAATATTTCTAATTTAACTATTTTTGGTTGGGGTGACCAAGGGGAAAAATGAATCCCCCTTATCGACCGAGTATTCAAGTACTTAAAAGTTAGAATTACAATTCTAACCGATAAAATTTTTACCGAAAAATGACCCAGGATTTTCCTGATCAATGAACCA
>NC_066688;tax=d:Eukaryota,p:Chordata,c:Chondrichthyes,o:Rhinopristiformes,f:Rhynchobatidae,g:Rhynchobatus,s:Rhynchobatus_djiddensis
ACTTAAGTTAATTACTCCATATAAACCACCCCTCCGGGTATAAAACAACATAAATAAATCTAACTTAACTTGTTTTTGGTTGGGGCGACCAAGGGGAAAAACAAAACCCCCTTATCGATTGAGTATTCAACATACTTAAAAATTAAAACGAATAGTTTTAATTAATAGAACATCTAACGAACAATGACCCAGGACACATCCTGATCAATGAACCA
>MN389524;tax=d:Eukaryota,p:Chordata,c:Chondrichthyes,o:Myliobatiformes,f:Dasyatidae,g:Neotrygon,s:Neotrygon_indica
ACTTAAGTTACCTTAACACCAAAAATTACACCCCAGGGTATAAAAACTAGAAATAACCTCTAACTTAACTTGTTTTCGGTTGGGGCGACCAAGGAGTAAAACAAAACCCCCTTATCGAATGTGTGAGAAATCACTCAAAATTAGAATTACAATTCTAATTAATAGAAGATCTAACGAACAATGACCCAGGACCTCATCCTGATCAATGAACCA
```
{: .output}

Let's also have a look at the fish database

```bash
head fish_16S_sintax.fasta
```

```
>KY228977;tax=d:Eukaryota,p:Chordata,c:Actinopteri,o:Cypriniformes,f:Gobionidae,g:Microphysogobio,s:Microphysogobio_amurensis
ACAAAATTTAACCACGTTAAACAACTCGACAAAAAGCAAGAACTTAGTGGAGTATAAAATTTTACCTTCGGTTGGGGCGACCACGGAGGAAAAACAAGCCTCCGAGTGGAATGGGGTAAATCCCTAAAACCAAGAGAAACATCTCTAAGCCGCAGAACATCTGACCAATAATGATCCGGCCTCATAGCCGATCAACGAACCA
>KY228987;tax=d:Eukaryota,p:Chordata,c:Actinopteri,o:Scombriformes,f:Scombridae,g:Scomberomorus,s:Scomberomorus_niphonius
ACGAAGGTATGTCATGTTAAGCACCCTAAATAAAGGACTAAACCGATTGAACTATACCCCTATGTCTTTGGTTGGGGCGACCCCGGGGTAACAAAAAACCCCCGAGCGGACTGGAAGTACTAACTTCTAAAACCAAGAGCTGCAGCTCGAACAAACAGAATTTCTGACCAATAAGATCCGGCAACGCCGATCAACGGACCG
>KY230517;tax=d:Eukaryota,p:Chordata,c:Actinopteri,o:Siluriformes,f:Sisoridae,g:Glyptothorax,s:Glyptothorax_cavia
ATAAAGATCACCTATGTCAATGAGCCCCATACGGCAGCAAACTAAATAGCAACTGATCTTAATCTTCGGTTGGGGCGACCACGGGAGAAAACAAAGCTCCCACGCGGACTGGGGTAAACCCTAAAACCAAGAAAAACATTTCTAAGCAACAGAACTTCTGACCATTAAGATCCGGCTACACGCCGACCAACGGACCA
>KY231189;tax=d:Eukaryota,p:Chordata,c:Actinopteri,o:Acipenseriformes,f:Acipenseridae,g:Acipenser,s:Acipenser_gueldenstaedtii_x_Acipenser_baerii
ACAAGATCAACTATGCTATCAAGCCAACCACCCACGGAAATAACAGCTAAAAGCATAATAGTACCCTGATCCTAATGTTTTCGGTTGGGGCGACCACGGAGGACAAAATAGCCTCCATGTCGACGGGGGCACTGCCCCTAAAACCTAGGGCGACAGCCCAAAGCAACAGAACATCTGACGAACAATGACCCAGGCTACAAGCCTGATCAACGAACCA
>KY231824;tax=d:Eukaryota,p:Chordata,c:Actinopteri,o:Cypriniformes,f:Leuciscidae,g:Alburnus,s:Alburnus_alburnus
ACAAAATTCAACCACGTTAAACGACTCTGTAGAAAGCAAGAACTTAGTGGCGGATGAAATTTTACCTTCGGTTGGGGCGACCACGGAGGAGAAAGAAGCCTCCGAGTGGACTGGGCTAAACCCCAAAGCTAAGAGAGACATCTCTAAGCCGCAGAACATCTGACCAAGAATGATCCGACTGAAAGGCCGATCAACGAACCA
```
{: .output}

That's all it takes to generate your own reference database. We are now ready to assign a taxonomy to our OTUs that were generated in the previous section!

{% include links.md %}
