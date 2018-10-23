# GBS-Pacecar: A GBS read simulator to put your de novo GBS pipeline through the paces!

### Introduction
GBS-PaceCar is a Perl script that simulates a set of single-end (SE) Illumina GBS reads with specifiable variant frequencies for use in evaluating the performance of *de novo* (i.e. reference-independent) GBS analytical pipelines.

### Approach
The simulator assumes the following:

1. Library preparation is based on restriction by the two enzymes PstI (rare cutter) and MspI (frequent cutter), as described by Poland et al. (2012);
2. Within a genome, the number of GBS fragments (PstI cut site – sequence – MspI cut site) is finite and designated as **`-n`**;
3. The **n** GBS fragments within a genome will be of variable length;
4. Some proportion of these **n** fragments will carry no sequence variation in the population, some proportion will carry a SNP, and some proportion will carry an indel;
5. Overall read depth will vary among individuals and among fragments within individuals; and
6. Random sequencing error of approximately 1%. 

Working under the assumptions listed above, the script can simulate SNPs as well as short (up to 10 bp) indels, with the flags **`-snps`** and **`-indels`** controlling the proportion of base GBS fragments (**`-n`**) carrying SNPs and indels, respectively. 
Each flag should range from 0 – 1, and their sum must be lower or equal to 1 (**`-snps`** + **`-indels`** ≤ 1). The default setting (**`-n`** 1000 **`-snps`** 0.25 **`-indels`** 0.10) means that out of 1,000 base GBS fragments in the genome, 250 will carry an induced SNP and 100 will carry an induced indel. 
To simulate only one type of variant, simply set to zero the proportion of the unwanted variant type (i.e. to simulate only SNPs, set **`-indel`** 0). Only one variant (SNP or indel) is simulated per base GBS fragment, and the flags **`–minl`** and **`–maxl`** control the minimum and the maximum lengths of the base fragments. 
A comprehensive report of all polymorphic fragments, including induced variant positions, types, and other features, can be found in the output files [GBS-PaceCar_Induced_SNPs.txt] and [GBS-PaceCar_Induced_Indels.txt]. Once the base GBS fragments are created, complete with their allelic states, they are sampled to generate SE Illumina sequence data for a population of **`-g`** genotypes (individuals). 
Specifically, for each individual, a genotypic state is randomly assigned at each locus (primary homozygous, heterozygous, or alternate homozygous) and reads are sampled accordingly. The reads randomly vary in depth among individuals and among loci within an individual, and all carry randomly induced sequencing error, at a rate of approximately 1%. 
The flags **`–mind`** and **`–maxd`** control the depth, such that there will be between **`–mind`** and **`–maxd`** reads for all individuals at all loci. Because they are present throughout the population, the induced SNPs and indels are high frequency variants that can be distinguished from the random sequencing errors that are created for each genotype separately.

GBS-PaceCar can produce two kinds of outputs. The first is a single compressed FASTQ file of all raw, barcoded NGS sequences, as they would appear off the sequencer, post CASAVA processing. To produce this output, the ***Raw*** option must be used as the first argument in the command line; and a [barcode ID file] (**`-b`**) is required. 
The barcode ID file is a two-column, tab-delimited file with the barcode sequences in the first column and the genotype names in the second. The number of rows in this file must match the number of genotypes declared using the **`-g`** flag. 
Once the reads are simulated for each individual, the appropriate barcode sequences are attached, followed by the PstI cut site, allowing standard demultiplexing and analysis via common de novo GBS bioinformatic tools like TASSEL-UNEAK (Lu et al. 2013) and GBS-SNP-CROP (Melo et al., 2016).
If the first argument of the command line is ***Parsed***, the genotype output mode is activated. In this mode, only the high-quality, processed sequences (i.e. trimmed of barcodes, restriction cut sites, and Illumina adapters) are produced. 
These parsed sequences are compressed into individual FASTQ files, one for each genotype (**`-g`**). If both outputs are desired, the option ***Both*** can be used. 

For each GBS-PaceCar run, a summary of the simulated reads is printed, as shown below:
```
# Summary of simulated reads:
Function = Parsed = demultiplexed sequence files
Total base GBS fragments = 1000
GBS fragments carrying a SNP = 250
GBS fragments carrying an indel = 100
Total number of simulated reads = 599794
Minimum read length = 32
Maximum read length = 150
Minimum read depth = 20
Maximum read depth = 30
Number of genotypes = 25
Average number of reads per genotype = 23991.76
Average length (bp) of the simulated GBS fragments = 90.27
Realized error rate = 1.11%
```

### The GBS-Pacecar output modes:
***Parsed***: the simulated reads will be demultiplexed across the number of genotypes declared by `**-g**`.  
***Raw***: all simulated reads will be compressed into a single FASTQ file of raw reads, as generated by CASAVA. Requires barcode ID text file (**`-b`**).
***Both***: creates both ***Parsed*** and ***Raw** outputs. Requires barcode ID file (`**-b**`).

### The GBS-Pacecar flags controlled via the command line:
**`-n`**: the number of desired GBS fragments (loci) in the genome. Default: 1000.  
**`-snps`**: proportion of fragments carrying a SNP. Default: 0.25.  
**`-indels`**: proportion of fragments carrying an indel. Default: 0.10.  
**`-minl`**:	minimum acceptable GBS fragment length. Default: 32.  
**`-maxl`**:	maximum read length, determined by sequencer. Default: 150.  
**`-g`**:	number of individuals (genotypes) in the population. Default: 25.
**`-mind`**: minimum read depth for a given GBS fragment for an individual. Default: 20.  
**`-maxd`**: maximum read depth for a given GBS fragment for an individual. Default: 30.  
**`-b`**: text file of barcodes and individual ID's: required for ***Raw*** and ***Both*** functions.

### Example command lines:
`perl GBS-Pacecar.pl Parsed –n 10000 –snps 0.2 –indels -0 –minl 50 –maxl 300 –g 10 –mind 10 –maxd 20`  

The above command will simulate single-end GBS sequence data for 10 individuals. Out of a total of 10,000 base GBS fragments, 2,000 will carry an induced SNP; no indels will be generated.
The GBS fragments will range from 50-300 bp in length, and the read depth for all individuals at all loci will range from 10-20x.
The output will be 10 compressed FASTQ files, one for each individual, containing all processed and trimmed reads.

`perl GBS-Pacecar.pl Both –n 200000 –snps 0.3 –indels -0.3 –minl 40 –maxl 150 –g 5 –mind 20 –maxd 30 –b barcodeIDs.txt`

The above command will simulate single-end GBS sequence data for 5 individuals. Out of a total of 200,000 base GBS fragments, 60,000 will carry an induced SNP and 60,000 will carry an induced indel. 
The GBS fragments will range from 40-150 bp in length, and the read depth for all individuals at all loci will range from 20-30x. 
The output will consist of: 1) 1 large, compressed FASTQ file containing all raw, unprocessed reads for all 5 individuals (Raw); and 2) 5 compressed FASTQ files, one for each individual, containing all processed and trimmed reads (Parsed).

### References
Bradbury PJ, et al. (2007) TASSEL: Software for association mapping of complex traits in diverse samples. Bioinformatics, 23:2633-2635.  
Lu F, et al. (2013) Switchgrass genomic diversity, ploidy, and evolution: novel insights from a network-based SNP discovery protocol. PLoS Genetics, DOI: 10.1371/journal.pgen.1003215.  
Melo ATO, et al. (2016) GBS-SNP-CROP: a reference-optional pipeline for SNP discovery and plant germplasm characterization using variable length, paired-end genotyping-by-sequencing data. BMC Bioinformatics, DOI: https://doi.org/10.1186/s12859-016-0879-y.  
Poland JA, et al. (2012) Development of high-density genetic maps for barley and wheat using a novel two-enzyme Genotyping-by- Sequencing approach. PLoS One, doi:10.1371/journal.pone.0032253.
