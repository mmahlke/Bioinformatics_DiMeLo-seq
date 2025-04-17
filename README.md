# **Analyzing DiMeLo-seq** 
*YNA lab bioinformatics training series, pt 3*



## What is $$\textnormal{\color{green}DiMeLo-seq}$$? 

**Di**rected **Me**thylation and **Lo**ng-Read sequencing (**DiMeLo-seq**) is a method to assess the position of proteins of interest on long reads. $$\textnormal{\color{green}DiMeLo-seq}$$ is particularly helpful for assessing protein position in repetitive regions of the genome, for assigning reads to haplotypes/alleles (allele phasing) in repetitive and non-repetitive regions, and for evaluating epigenetic spatial distrubtion on single DNA strands. 

$$\textnormal{\color{green}DiMeLo-seq}$$ starts by permeabilizing intact cells and introducing an antibody for the protein of interest. After the antibody binds the protein, antibody-binding **protein A** fused to the deoxyadenosine methyltransferase **Hia5 enzyme** (**pA-Hia5**) is added, bringing **Hia5 enzyme** in close proximity to the protein of interest. Next, **Hia5** is activated, allowing it to modify adenosines surrounding the protein of interest to N6-methyl-deoxyadenosine. Since methylated adenosines (mA) naturally occurr at a low abundance in human DNA, the position of the protein of interest is inferred from the occurrence of mA on sequencing reads during analysis. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/Dimelo_2.png" alt="Dimelo-seq method and applications" style="width:85%; height:85%;">
</div>

*Image from Altemose et al., 2022*

Please refer to [this paper](https://www.nature.com/articles/s41592-022-01475-6) for more information about DimeLo-seq.

## Oxford Nanopore Sequencing Technologies 
$$\textnormal{\color{green}DiMeLo-seq}$$ is built upon Oxford Nanopore Technologies (ONT) sequencing platforms that allow for sequecing of continuous ultra-long DNA molecules. This approach is different than a traditional short read sequencing approach, like Illumina sequencing. 

Illumina             |  ONT
:-------------------------:|:-------------------------:
![Illumina Sequencing](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/illumina.png) |  ![ONT sequencing](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/nanopore-2.png)
[*Source*](https://praxilabs.com/en/blog/2021/02/08/dna-sequencing-definition-importance-methods-facts-and-more/) | [*Source*](https://quickbiology.com/ngs-service-provider/oxford-nanopore-sequencing-new-service-quick-biology-inc)

Illumina sequencing relies on bridge amplification of the existing DNA to produce a robust, detectable fluorescent signal that is translated into a DNA sequence. ONT sequencing does not amplify DNA, so you only read native DNA sequences. Most importantly for DiMeLo-seq, ONT sequencing determines the base at each position along a DNA strand based on how it disrupts the ionic current passing through the nanopore. This enables the identification of modified DNA bases, such as  5-methylcytosine (DNA methylation) or N6-methyl-deoxyadenosine (mA), that are characterized by large structural additions to a normal DNA base. 

Current flow through nanopores             |  Methylated adenosine
:-------------------------:|:-------------------------:
![Current and nanopores](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/current.png) |  ![m6A](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/m6A.png)
[*Source*](https://nanoporetech.com/document/nanopore-sensing) | [*Source*](https://www.frontiersin.org/journals/cell-and-developmental-biology/articles/10.3389/fcell.2021.694673/full)

While ONT is generally thought to be less accurate than PacBio long read sequencing, ONT reaches >99% accuracy in basecalling of individual reads, and the accuracy of consensus sequences reaches above 99% by increasing sequencing coverage. For instance, the CHM13 T2T assembly averages at **Q51**. For sequencing data, Q value (Phred score) is a standardized way to represent the quality of each base call in a sequencing read. Q values are calculated using the following formula: Q = -10 * log10(P), where P is the probability of error. 

| Phred Quality Score |	Probability of incorrect base call	| Base call accuracy |
| :-------------: | :-------------: | :-------------: |
| 10	| 1 in 10	| 90% |
| 20	| 1 in 100	| 99% |
| 30	| 1 in 1000	| 99.9% |
| 40	| 1 in 10,000	| 99.99% |
| 50	| 1 in 100,000	| 99.999% |
| 60	| 1 in 1,000,000	| 99.9999% |

ONT's modified basecalling is also exceedingly accurate, with DNA methylation being identified with an accuracy that meets or surpasses Bisulfite sequencing, which is considered to be the [most accurate method](https://pmc.ncbi.nlm.nih.gov/articles/PMC3233226/) for identifying DNA methylation. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/bisulfite_ont.png" alt="mCG accuracy" style="width:50%; height:50%;">
</div>


## DiMeLo-seq data analysis
Raw ONT data is output in .pod5 (new) or .fast5 (old) formats and represents the raw changes in current measured at a given nanopore over the length of a DNA molecule passing through the nanopore. To translate that into a DNA sequence, we apply basecalling models that identify characteristic changes in current as different DNA bases,  producing long read sequences in .fasta format. There are currently basecalling mdoels to identify 5mC, 5hmC, 4mC + 5mC and 6mA for DNA and m6A and pseudouridine for RNA.  

Most ONT data analysis uses ONT-specific tools. The current tool for basecalling is Dorado. 

Go into R10 chip analysis

Should be simpler than others

Talk about visualizing the data

Making an evironment

And visualizing the data on IGV interactive session on CRC cluster
