# **Analyzing DiMeLo-seq** 
*YNA lab bioinformatics training series, pt 3*



## What is $$\textnormal{\color{green}DiMeLo-seq}$$? 

**Di**rected **Me**thylation and **Lo**ng-Read sequencing (**DiMeLo-seq**) is a method to assess the position of proteins of interest on long reads. $$\textnormal{\color{green}DiMeLo-seq}$$ is particularly helpful for assessing protein position in repetitive regions of the genome, for assigning reads to chromosome haplotypes/alleles (allele phasing) in repetitive and non-repetitive regions, and for evaluating epigenetic spatial distrubtion on single DNA strands. 

$$\textnormal{\color{green}DiMeLo-seq}$$ starts by permeabilizing intact cells and introducing an antibody for the protein of interest. After the antibody binds the protein, antibody-binding **protein A** fused to the deoxyadenosine methyltransferase **Hia5 enzyme** (**pA-Hia5**) is added, bringing **Hia5 enzyme** in close proximity to the protein of interest. Next, **Hia5** is activated, allowing it to modify adenosines surrounding the protein of interest to N6-methyl-deoxyadenosine. Since methylated adenosines (mA) naturally occurr at a low abundance in human DNA, the position of the protein of interest is inferred from the occurrence of mA on sequencing reads during analysis. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/Dimelo_2.png" alt="Dimelo-seq method and applications" style="width:85%; height:85%;">
</div>

*Image from Altemose et al., 2022*

Please refer to [Altemose et al., 2022](https://www.nature.com/articles/s41592-022-01475-6) for more information about DimeLo-seq, and [Gamarra et al., 2025](https://www.biorxiv.org/content/10.1101/2025.03.11.642717v1.full) for updates on Dimelo-seq technique.

## Oxford Nanopore Sequencing Technologies 
$$\textnormal{\color{green}DiMeLo-seq}$$ is built upon $$\textnormal{\color{skyblue}Oxford Nanopore Technologies (ONT)}$$ sequencing platforms that allow for sequecing of continuous ultra-long DNA molecules. $$\textnormal{\color{skyblue}ONT}$$ sequencing is different than a traditional short read sequencing approach, like Illumina sequencing. 

Illumina             |  ONT
:-------------------------:|:-------------------------:
![Illumina Sequencing](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/illumina.png) |  ![ONT sequencing](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/nanopore-2.png)
[*Source*](https://praxilabs.com/en/blog/2021/02/08/dna-sequencing-definition-importance-methods-facts-and-more/) | [*Source*](https://quickbiology.com/ngs-service-provider/oxford-nanopore-sequencing-new-service-quick-biology-inc)

Illumina sequencing relies on bridge amplification of the existing DNA to produce a robust, detectable fluorescent signal that is translated into a DNA sequence. 

$$\textnormal{\color{skyblue}ONT}$$ sequencing does not amplify DNA and instead reads native DNA sequences directly as they pass through nanopores. Most importantly for DiMeLo-seq, $$\textnormal{\color{skyblue}ONT}$$ sequencing determines the base at each position along a DNA strand based on how it disrupts the ionic current passing through the nanopore. This enables the identification of modified DNA bases, such as  5-methylcytosine (DNA methylation) or N6-methyl-deoxyadenosine (mA), that are characterized by large structural additions to a normal DNA base. 

Current flow through nanopores             |  Methylated adenosine
:-------------------------:|:-------------------------:
![Current and nanopores](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/current.png) |  ![m6A](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/m6A.png)
[*Source*](https://nanoporetech.com/document/nanopore-sensing) | [*Source*](https://www.frontiersin.org/journals/cell-and-developmental-biology/articles/10.3389/fcell.2021.694673/full)

$$\textnormal{\color{skyblue}ONT}$$ sequencing is performed on flow cells composed of an array of nanopores set into a membrane held by microscaffolds. Each microscaffold holds an individual electrode that senses current changes. Each nanopore is also controlled and measured by an individual channel on a corresponding integrated circuit. Essentially, this allows for simultaneous parallel sequencing of individual molecules. It also means flow cells are sensitive! Be gentle. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/flow-cell-array.png" alt="Flow cell" style="width:30%; height:30%;">
</div>

*Image modified from [Wang et al., 2021](https://www.nature.com/articles/s41587-021-01108-x)*


While $$\textnormal{\color{skyblue}ONT}$$ is generally thought to be less accurate than PacBio long read sequencing, ONT reaches >99% accuracy in $$\textnormal{\color{magenta}basecalling}$$ of individual reads, and the accuracy of consensus sequences reaches above 99% by increasing sequencing coverage. For instance, the CHM13 T2T assembly averages at **Q51**. For sequencing data, Q value (Phred score) is a standardized way to represent the quality of each base call in a sequencing read. Q values are calculated using the following formula: Q = -10 * log10(P), where P is the probability of error. 


<div align="center">
 
 Phred Quality Score |	Probability of incorrect base call	| Base call accuracy 
 :-------------: | :-------------: | :-------------: 
 10	| 1 in 10	| 90% 
 20	| 1 in 100	| 99% 
 30	| 1 in 1000	| 99.9% 
 40	| 1 in 10,000	| 99.99% 
 50	| 1 in 100,000	| 99.999% 
 60	| 1 in 1,000,000	| 99.9999% 
</div>
<br />

$$\textnormal{\color{skyblue}ONT's}$$ modified $$\textnormal{\color{magenta}basecalling}$$ is also exceedingly accurate, with DNA methylation being identified with an accuracy that meets or surpasses Bisulfite sequencing, which is considered to be the [most accurate method](https://pmc.ncbi.nlm.nih.gov/articles/PMC3233226/) for identifying DNA methylation. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/bisulfite_ont.png" alt="mCG accuracy" style="width:30%; height:30%;">
</div>

*Image above from ONT's [accuracy](https://nanoporetech.com/platform/accuracy) webpage*


Now that we have an idea what we are analyzing, let's talk about the data itself.

## $$\textnormal{\color{green}DiMeLo-seq}$$ data analysis

Raw $$\textnormal{\color{skyblue}ONT}$$ data is output in .pod5 (new) or .fast5 (old) formats and represents the raw changes in current measured at a given nanopore over the length of a DNA molecule passing through the nanopore. To translate that into a DNA sequence, we apply $$\textnormal{\color{magenta}basecalling}$$ models that identify characteristic changes in current as different DNA bases,  producing long read sequences in .fasta format. There are currently $$\textnormal{\color{magenta}basecalling}$$ models to identify 5mC, 5hmC, 4mC + 5mC and 6mA for DNA and m6A and pseudouridine for RNA.  

Most $$\textnormal{\color{skyblue}ONT}$$ data analysis uses $$\textnormal{\color{skyblue}ONT}$$-specific tools. The current tool for $$\textnormal{\color{magenta}basecalling}$$ is [Dorado](https://github.com/nanoporetech/dorado). $$\textnormal{\color{magenta}Basecalling}$$ is a resource-intensive process, so ```Dorado``` is built to be run on a graphical processing unit (GPU). **CPUs** (Central Processing Units) and **GPUs** (Graphics Processing Units) are both computing engines, but they are best used for different types of tasks. **CPUs** are designed for general-purpose computing, while **GPUs** are optimized for parallel processing and tasks that involve large datasets. This means that when we submit a script to the CRC cluster to perform $$\textnormal{\color{magenta}basecalling}$$ with ```Dorado```, we must request that it be run on the GPU computing cluster. 

Here is an examble of the SBATCH header for a script requesting to use CRC's GPU cluster:

```
#!/bin/bash
#SBATCH --job-name=dorado_basecalling
#SBATCH --cluster=gpu
#SBATCH --partition=a100
#SBATCH --nodes=1 
#SBATCH --ntasks-per-node=2 
#SBATCH --cpus-per-task=16 
#SBATCH --time=48:00:00 
#SBATCH --error=/path/to/error/job.%J.err
#SBATCH --output=/path/to/output/job.%J.out
```

```Dorado``` is pretty simple to use. ```Dorado``` can $$\textnormal{\color{magenta}basecall}$$ .pod5 raw data and align to an assembly of your choice all at once. Alignment will be performed using [minimap2](https://github.com/lh3/minimap2), an alignment tool built specifically for long-read data. However, there is another alignment tool called [winnowmap](https://github.com/marbl/Winnowmap), essentially a tweaked version of minimap designed to perform better in repetitive regions like centromeres. In my hands, I've found winnowmap and minimap to perform similarly at centromeres, but find winnowmap to perform poorly outside of repetitive regions. To avoid complicating your analysis, I suggest aligning with minimap2 unless you find an issue with your data.

A typical set of instructions for ```Dorado``` are:

```
module load dorado/0.5.3

cd /path/to/your/working/directory/

dorado basecaller hac,5mC_5hmC,6mA \
/path/to/folder/containing/pod5/files/ \
--reference /path/to/reference/genome/aligning/to.fasta \
--verbose > sample_name.bam

```
Where
+ ```hac,5mC_5hmC,6mA``` specifies to perform $$\textnormal{\color{magenta}basecalling}$$ with the high-accuracy $$\textnormal{\color{magenta}basecalling}$$ model ```hac``` and to include detection of both 5mC/5hmC in CG contexts and 6mA in all contexts
+ ```--reference``` specifies the assembly to align to after $$\textnormal{\color{magenta}basecalling}$$
+ ```--verbose ``` specifies to give verbose output
+ ```sample_name.bam``` is the name we want to give the resulting aligned .bam file

We are using the default **modified base threshold (0.05)** built in to ```Dorado```. This means if ```Dorado``` finds that the probability of a base being a modified base is less than 5%, it will report the base as unmodified. If ```Dorado``` finds the probability of a base being modified is 6%, it will report the base as modified but at a 6% possibility. You will have more opportunities to filter the data based on modification probability later. 

You can also supply options to the ```minimap2``` aligner within ```Dorado``` by adding a string like:
```
dorado basecaller --mm2-opts "-k 15 -w 100 -Y -I 8G" hac,5mC_5hmC,6mA \
```
Where -k and -w set kmer and window size respectively, and -Y turns on soft-clipping. ```minimap2``` in ```dorado``` uses hard-clipping which can invalidate modified basecalling for reads that fail to align perfectly, so be sure to test for your dataset. ```-I 8G``` controls the index size and memory usage. Read more about ```minimap2``` options [here](https://lh3.github.io/minimap2/minimap2.html).

OK, our command to ```Dorado``` has run successfully and we now have a .bam file with reads aligned to our reference and the positions of modified bases on reads documented. It's important to understand a little about how those modifications are stored in the data in case you need to access them. 


## SAM and BAM tags

Recall that .sam and .bam files are file types that have .fasta reads aligned to a genome assembly. The structure of .sam/.bam files are also discussed in the previous training session [here](https://github.com/mmahlke/YNAlab_Bioinformatics_training_pt2_ChIP_and_CR/blob/main/README.md). 

You can read more about .sam structure [here](https://samtools.github.io/hts-specs/SAMv1.pdf) and about .sam tags [here](https://samtools.github.io/hts-specs/SAMtags.pdf).

A typical read in a .sam file from an $$\textnormal{\color{skyblue}ONT}$$ sequencing run could look like this:

```
Nanopore_Sequence_Read 0	PDNC4_CHR4_HAP2	50427127	0	89S23=1D21=2X7=1D82=1X1=1I55=1I95=1D29=2D54=1I3=1D25=2I4=1D8=1I23=	*	0	528  
GTTATGTAACCTACTTGGTTCCATTACGTATTGCTGGTGCTGAAGATTGTAGGTGTCTTTGTGCAGAGTGTATGATATACACGGCGGTGCTGAAGAAAGTTATTGCGGGTGTATTTGTGCAGAAGTATATGATGTGCGCGGGCGGAGGTGCTGAAGAAAGTTGTCGGTGTCTTTGTGCAGAAGTATATGATGGCGAGGTGTTGAAGAAAGTTGTCGGTGTCTTTGTGCAGAAGTATATGATGTGCGCGGGCGGATCCGCCCGCGCATCCTTCTGCGCAAT    
"&+*+*)$#$%'&&')%&'(,)))*1555:>B@7777BBCD10.*.%%&)$$$*14//0.,.-..'(%(%&''..211--'$$%&+))56<;;998:44892.-,+)('&*)'&&((,++/0064:385566;>A>=6@<@AA?;:::=>>?==7=?@=<>>;BA@??@?=:;;;7011+,,).-,++++-&$$%)(,)*,)$%'(((&&'(&&%%%&+0///20877656??BAA@@ABBEFGC>=57793222532110,50-++$$$%&),,,+())    
MM:Z:C+h?,5,5,0,1,1,0,0,1,2,0,2,0,0,1,2,0,4;C+m?,5,5,0,1,1,0,0,1,2,0,2,0,0,1,2,0,4; 
ML:B:C,159,6,135,2,7,9,3,4,13,11,6,22,6,1,2,2,218,0,4,19,1,2,7,4,1,0,1,4,15,11,2,1,0,0
```
The fields of the above .sam file are tab-delineated and give values for:

1. ```Nanopore_Sequence_Read``` the name assigned to a single read, read ID
2. ```0``` FLAG with information about the read
3. ```PDNC4_CHR4_HAP2``` name of chromosome or contig the read is aligned to
4. ```50427127``` -1 based position where the start of the read aligns to the chromosome or contig
5. ```0``` MAPQ mapping quality, 0 value indicates the read maps to other locations (repetitive cens)
6. ```89S23=1D21......``` [CIGAR](https://jef.works/blog/2017/03/28/CIGAR-strings-for-dummies/) string, how the SAM/BAM format represents spliced alignments with insertions or deletions 
7. ```*``` RNEXT, reference seq info for the next/paired read. * for no info, these reads are not paired
8. ```0``` PNEXT, positional info for the next read
9. ```528``` TLEN, template length, distance between the leftmost and rightmost mapped bases of a sequenced DNA fragment
10. ```GTTATG``` The sequence itself
11. ```"&+*....``` ASCII of base quality. Base quality is the phred-scaled base error probability which equals −10 log10 Pr{base is wrong}. It will be the same length as the sequence. Read more about ASCII format [here](https://help.basespace.illumina.com/files-used-by-basespace/quality-scores)
12. ```MM:Z```  identifies modified bases. Here, ```C+h?``` reports the status of 5hmC and ```C+m?``` reports the status of 5mC. If mA is called, it's reported with ```A+a?```
13. ```ML:B:C``` identifies the probability of each modification listed in the MM tag being correct. The continuous probability range 0.0 to 1.0 is remapped in equal sized portions to the discrete integers 0 to 255 inclusively. This is important for understanding how the values 0 to 255 encode to probability thresholds you may want to apply between probability 0 (unlikely) to probability 1.0 (certain).

## Normalizing and defining enrichment

One thing you may find to be missing at this stage is some sort of normalization or statistical peak calling. When we think about what we are interested in getting out of $$\textnormal{\color{green}DiMeLo-seq}$$, it's a bit more complicated than ChIP-seq or CUT&RUN. We aren't enriching for reads with a pull down and amplifying them. We are inferring the position of a protein along single strands of DNA. We know there will be variability (in the context of centromeric proteins, at least) and we expect that. Yet, we still want to know wehre the proteins are **most likely** to be enriched.

Think about the data itself. Each base reports either modified or unmodified. At modified bases, we report a probability that the modification occurs. We filter out the bases with low probability. Next, we will measure the probable modifications at each position in each strand covering a region and calculate a fraction of enrichment. Theoretically, calculating a fraction of modified bases across a region should normalize for coverage level. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/dimelo_ratio_norm-01.svg" alt="Dimelo_ratio" style="width:70%; height:70%;">
</div>

<br/>
However, that won't always be perfect. There is currently no standardized normalization method or enrichment quantification for $$\textnormal{\color{green}DiMeLo-seq}$$. 

<br/> 
<br/>

**Some approaches for determining enrichment are:**

**1)** Using emperical data to set a threshold
 + use other sequencing data to define area(s) where your protein of interest is enriched 
 + observe those area(s) in your DiMeLo data to determine an enrichment threshold to apply across the genome
<br/>
 
**2)** Using fold enrichment to set a cutoff for enrichment
 + You can extract the numbers of modified bases per position using [modkit](https://github.com/nanoporetech/modkit) to calculate fold-enrichment values
   + There's no standard way to do this; you could try setting 'low' areas as 1 and then calculate the comparitive fold enrichment of 'high' values
   + You can also use modkit to transform the data into .bw tracks representing enrichment values
 + [fibertools](https://github.com/fiberseq/fibertools-rs) could also be useful for extracting and transforming raw data
   + fibertools can be used to translate m6A signature into tracks of protein position 

## Making a $$\textnormal{\color{teal}virtual environment}$$ and visualizing modified bases

For this training, we will use ```dimelo``` package to access the modification data for us! Read more about the dimelo package [here](https://streetslab.github.io/dimelo/html/index.html) and access the package [here](https://github.com/streetslab/dimelo). 

```dimelo``` visualization package is not pre-installed on the CRC cluster. We can install it in our own space by creating a $$\textnormal{\color{teal}virtual environment}$$. A $$\textnormal{\color{teal}virtual environment}$$ is like a containerized test space that is on the CRC cluster but separated from the CRC cluster. It's a bit like "they can't mess with us" and "we can't mess with them", so everyone is safe. By safe, I mean safe to install new packages, test scripts, etc without harming the existing CRC infrastructure or running into complications from interacting with the CRC infrastructure. Once we build our $$\textnormal{\color{teal}virtual environment}$$, we can install the ```dimelo``` package there and run it on our data. 

Making a $$\textnormal{\color{teal}virtual environment}$$ is a function of ```Python```. Python is a programming language. For other analyses, we have used the command-line interface to communicate directly to Linux, the CRC's operating system. ```Python``` adds a communication layer that allows us to generate more complex instructions or 'programs'. For instance, the ```dimelo``` package is written in ```Python``` programming language. 

To keep things simple, I've installed the ```dimelo``` package into a $$\textnormal{\color{teal}virtual environment}$$ called ```dimelo_vis``` in the training directory.

Let's all test whether we can use the package. If you cannot, or if you just want to practice making your own environment and installing the package yourself, please see the instructions linked [here](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Scripts/dimelo_package_installation.txt).

Log in to the cluster and request an interactive session on the htc cluster specifically.

```
srun -t 2:00:00 --cluster htc --partition htc --cpus-per-task 8 --pty bash
```

To run the ```dimelo``` package, we need to load ```python``` because the package is written in ```python```. 
```
module load python/3.7.0
```
Navigate to your working directory
```
cd /ix1/yarbely/<your_user>/training
```
And make a directory for today
```
mkdir ./Dimelo
```
Then we activate our virtual environment by using ```source activate``` and including a path to the environment location. 
```
source activate /ix1/yarbely/Data/Training/envs/dimelo_vis
```
You can tell you are operating within your virtual environment because there is a prefix before your command prompt that looks like:
```
(dimelo_vis)[user@htc-n2-013]$
``` 
Now we need to activate python with the command:
```
python
```
We can tell that the python laguage interpreter is running in the environment now because the command prompt changed from ```$   ``` to ```>>>  ```. 

So let's speak to the dimelo package in python. 
First, we tell ```dimelo``` package to import dimelo commands and recognize them as ```dm```.
```
import dimelo as dm
```
Now, we command dimelo package to prepare some figures for us. I've copied a file to the /Training directory that we can use to make some figures with. The path is specified below. 

Let's run these two commands:

```
dm.plot_browser("/ix1/yarbely/Data/Training/D2Y_mCG.bam", "mCpG", "PDNC4_Hifiasm_chr4_hap1:79000000-79500000", "CG", "/ix1/yarbely/<your_user>/training/Dimelo/CG", colorC='#053C5E', threshA=205, dotsize=2, static=True)

dm.plot_browser("/ix1/yarbely/Data/Training/D2Y_CENPA.bam", "CENPA", "PDNC4_Hifiasm_chr4_hap1:79000000-79500000", "A", "/ix1/yarbely/<your_user>/training/Dimelo/CENPA", colorC='#cc322f', threshA=205, dotsize=2, static=True)
```
Where:
+ the first argument is the path to your .bam file
+ ```mCpG``` specifies the title of the graphs
+ ```Chrx:1-999``` specifies the corrdinates to sample
+ ```CG``` specifies the modification to sample
+ ```path/out``` specifies the folder to place the results
+ ```color``` is the color to use for the dots and the line graphs
+ ```thresh*=205``` specifies the probability threshold to use. 205 = 80% likely. Anything below will not be included.
+ ```dotsize``` specifies the size of the dots in the genome browser plot showing individual reads
+ ```static=True``` allows the output to be written to a .pdf file rather than to be generated interactively in the terminal

Now let's view the data that it created for us.  There will be three files:
+ a browser track plot showing read pileup over our selected coordinates with modified bases indicated along the reads
+ a histogram plotting aggregate m6A per position
+ a histogram plotting aggregate fraction m6A/A per position

In the commands above, we are using the default window size to smooth the aggregate curves (1000 bp). You can change the smoothing window with ```smooth= ```.

## Viewing large data interactively with OnDemand and IGV

Lastly, we can also visualize the data stored in the .bam file interactively on IGV. But the file is much too large for us to download it and view it on our own computer using the IGV web app. We can view it by requesting an interactive session through [OnDemand](https://ondemand.htc.crc.pitt.edu/) connected to the CRC cluster. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/ondemand.png" alt="OnDemand" style="width:50%; height:50%;">
</div>

Once you log on, navigate to Interactive Apps and to IGV on htc.

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/IGV_ondemand.png" alt="Using IGV on htc" style="width:80%; height:80%;">
</div>


Specify the resources you want to use:

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/IGV_on_htc.png" alt="Requesting resources" style="width:80%; height:80%;">
</div>

Although we are requesting a number of cores in the above example, these large data can be cumbersome and if you remove a number and leave blank, you will recieve resources from the entire node. 


Here is a snapshot of the DiMeLo-seq .bam file we have been working with, showing mA in green and mC in red. To view modified bases in IGV, right click the file name and select 'color alignments by' then select 'base modification'. 

The location shown is HOR1 on PDNC4 Chr4 Hap2, the copy of Chr4 with normal centromere. 

Can you tell where the **active centromere** is? You can see the high variability in CENP-A position even within the reads from one young clone. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/active_cen4.png" alt="Dimelo at CEN4" style="width:100%; height:100%;">
</div>

To view this .bam file we have been working with to IGV, you first need to load the PDNC4 assembly, because that is what it is aligned to. I placed that assembly file in the training folder. Click on 'Genomes' then 'Load genome from file' and select the PDNC4 genome assembly file. Then you can load the D2Y_CENPA.bam file. 

You can also look for the neocentromere on PDNC4 Hap1 at coordinates 79000000-79500000.
