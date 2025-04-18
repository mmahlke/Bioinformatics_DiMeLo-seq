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


While $$\textnormal{\color{skyblue}ONT}$$ is generally thought to be less accurate than PacBio long read sequencing, ONT reaches >99% accuracy in basecalling of individual reads, and the accuracy of consensus sequences reaches above 99% by increasing sequencing coverage. For instance, the CHM13 T2T assembly averages at **Q51**. For sequencing data, Q value (Phred score) is a standardized way to represent the quality of each base call in a sequencing read. Q values are calculated using the following formula: Q = -10 * log10(P), where P is the probability of error. 


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

$$\textnormal{\color{skyblue}ONT's}$$ modified basecalling is also exceedingly accurate, with DNA methylation being identified with an accuracy that meets or surpasses Bisulfite sequencing, which is considered to be the [most accurate method](https://pmc.ncbi.nlm.nih.gov/articles/PMC3233226/) for identifying DNA methylation. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/bisulfite_ont.png" alt="mCG accuracy" style="width:30%; height:30%;">
</div>

*Image above from ONT's [accuracy](https://nanoporetech.com/platform/accuracy) webpage*


Now that we have an idea what we are analyzing, let's talk about the data itself.

## $$\textnormal{\color{green}DiMeLo-seq}$$ data analysis

Raw $$\textnormal{\color{skyblue}ONT}$$ data is output in .pod5 (new) or .fast5 (old) formats and represents the raw changes in current measured at a given nanopore over the length of a DNA molecule passing through the nanopore. To translate that into a DNA sequence, we apply basecalling models that identify characteristic changes in current as different DNA bases,  producing long read sequences in .fasta format. There are currently basecalling mdoels to identify 5mC, 5hmC, 4mC + 5mC and 6mA for DNA and m6A and pseudouridine for RNA.  

Most $$\textnormal{\color{skyblue}ONT}$$ data analysis uses $$\textnormal{\color{skyblue}ONT}$$-specific tools. The current tool for basecalling is [Dorado}(https://github.com/nanoporetech/dorado). Basecalling is a resource-intensive process, so ```Dorado``` is built to be run on a graphical processing unit (GPU). **CPUs** (Central Processing Units) and **GPUs** (Graphics Processing Units) are both computing engines, but they are best used for different types of tasks. **CPUs** are designed for general-purpose computing, while **GPUs** are optimized for parallel processing and tasks that involve large datasets. This means that when we submit a script to the CRC cluster to perform basecalling with ```Dorado```, we must request that it be run on the GPU computing cluster. 

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

```Dorado``` is pretty simple to use (if you don't care about centromeres that much). ```Dorado``` can basecall .pod5 raw data and align to an assembly of your choice all at once. Alignment will be performed using [minimap2](https://github.com/lh3/minimap2), an alignment tool built specifically for long-read data. However, there is another alignment tool called [winnowmap](https://github.com/marbl/Winnowmap) that performs better in repetitive regions like centromeres. To align your basecalled .fastq files with winnowmap, you need to perform several additional steps, detailed [here](insert link to script). 

A typical set of instructions for ```Dorado``` are:

```
module load dorado/0.5.3

cd /path/to/your/working/directory/

dorado basecaller hac,5mC_5hmC,6mA \
/path/to/folder/containing/pod5/files/ \
--reference /bgfs/yarbely/Data/Altemose_Oneill/PDNC4/PDNC4.hifiasm.v0.16.1_V2-2022NOV_singleLine.fasta \
--verbose > CA.HJ.LAP_cC4_12m_minimap_sc.bam

```
where
+ ```hac,5mC_5hmC,6mA``` spicifies to perform basecalling with the high-accuracy basecalling model ```hac``` and to include detection of both 5mC/5hmC in CG contexts and 6mA in all contexts
+ ```--reference``` specifies the assembly to align to after basecalling
+ ```--verbose ``` specifies to give verbose output
+ ```data.bam``` is the name we want to give the aligned .bam file

We are using the default modified base threshold (0.05) built in to ```Dorado```. This means if ```dorado``` reports that the probability of a base being a modified base is less than 5%, it will report as unmodified. If it's 6% sure that it is a modified base, it will report it as modified but at a 6% possibility. You will have opportunities to filter further based on modification probability later. 

You can also supply options to the ```minimap2``` aligner within ```Dorado``` by adding a string like:
```
dorado basecaller --mm2-opts "-k 15 -w 100 -Y" hac,5mC_5hmC,6mA \

```
Where -k and -w set kmer and window size respectively, and -Y turns on soft-clipping. ```minimap2``` in ```dorado``` uses hard-clipping which can invalidate modified basecalling for reads that fail to align perfectly, so be sure to test for your dataset.

OK, our command to ```Dorado``` has completed and we now have a .bam file with reads aligned to our reference and the positions of modified bases on reads documented. It's important to understand a little about how those modifications are stored in the data in case you need to access them. 








For this training, we will use ```dimelo``` package to access the modification data for us!

```dimelo``` visualization package is not pre-installed on the CRC cluster. We can install it in our own space by creating a virtual environment. A virtual environment is like a containerized test space that is on the CRC cluster but separated from the CRC cluster. It's a bit like "they can't mess with us" and "we can't mess with them", so everyone is safe. By safe, I mean safe to install new packages, test scripts, etc without harming the existing CRC infrastructure. Once we build our virtual environment, we can install the ```dimelo``` package there and run it on our data. 



Making an evironment

And visualizing the data on IGV interactive session on CRC cluster
