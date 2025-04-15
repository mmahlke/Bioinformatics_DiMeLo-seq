# **Analyzing DiMeLo-seq** 
*YNA lab bioinformatics training series, pt 3*



## What is $$\textnormal{\color{green}DiMeLo-seq}$$? 

**Di**rected **Me**thylation and **Lo**ng-Read sequencing (**DiMeLo-seq**) is a method to assess the position of proteins of interest on long reads. DiMeLo-seq is particularly helpful for assessing protein position in repetitive regions of the genome, for assigning reads to haplotypes/alleles (allele phasing) in repetitive and non-repetitive regions, and for evaluating epigenetic spatial distrubtion on single DNA strands. 

The $$\textnormal{\color{green}DiMeLo-seq}$$ technique starts with binding an antibody to the protein of interest, then introduction of antibody-binding **protein A** fused to the deoxyadenosine methyltransferase **Hia5 enzyme**. pA-Hia5 will bind to the antibody, then Hia5 modifies adenosines surrounding the protein of interest to N6-methyl-deoxyadenosine. Since methylated adenosines (mA) naturally occurr at a low abundance in human DNA, the position of the protein of interest is inferred from the occurrence of mA on sequencing reads during analysis. 

<div align="center">
 <img src="https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/Dimelo.png" alt="Dimelo-seq method and applications" style="width:85%; height:85%;">
</div>

*Image from Altemose et al., 2022*

Please refer to [this paper](https://www.nature.com/articles/s41592-022-01475-6) for more information about DimeLo-seq.

DiMeLo-seq is built upon Oxford Nanopore Technologies sequencing platforms that allow for sequecing of continuous ultra-long DNA molecules. This approach is different than a traditional short read sequencing approach, like Illumina sequencing. 

Illumina             |  ONT
:-------------------------:|:-------------------------:
![Illumina Sequencing](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/illumina.png) |  ![ONT sequencing](https://github.com/mmahlke/Bioinformatics_DiMeLo-seq/blob/main/Figures/nanopore-2.png)
[*Source*](https://praxilabs.com/en/blog/2021/02/08/dna-sequencing-definition-importance-methods-facts-and-more/) | [*Source*](https://quickbiology.com/ngs-service-provider/oxford-nanopore-sequencing-new-service-quick-biology-inc)

Modified basecalling

Go into R10 chip analysis

Should be simpler than others

Talk about visualizing the data

And visualizing the data on IGV interactive session on CRC cluster
