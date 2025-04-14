# **Analyzing DiMeLo-seq** 
*YNA lab bioinformatics training series, pt 3*



## What is $$\textnormal{\color{green}DiMeLo-seq}$$? 

**Di**rected **Me**thylation and **Lo**ng-Read sequencing (**DiMeLo-seq**) is a method to assess the position of proteins of interest on long reads. DiMeLo-seq is particularly helpful for assessing protein position in repetitive regions of the genome, for assigning reads to haplotypes/alleles (allele phasing) in repetitive and non-repetitive regions, and for evaluating epigenetic spatial distrubtion on single DNA strands. 

The $$\textnormal{\color{green}DiMeLo-seq}$$ technique starts with antibody binding to the protein of interest, followed by introduction of antibody-binding protein A fused to the nonspecific deoxyadenosine methyltransferase Hia5 enzyme. Upon additiona of SAM, Hia5 modifies adenosines surrounding the protein of interest to N6-methyl-deoxyadenosine which naturally occurrs at a low abundance in human DNA. During analysis, the position of the protein of interest is inferred from the position of N6-methyl-deoxyadenosine on sequencing reads. 
