# HipSTR
**H**aplotype-based **i**mputation, **p**hasing and genotyping of **STR**s

#### Author: Thomas Willems <twillems@mit.edu>
#### License: GNU v2

## Introduction
Short tandem repeats [(STRs)](http://en.wikipedia.org/wiki/Microsatellite) are highly repetitive genomic sequences comprised of repeated copies of an underlying motif. Prevalent in most organisms' genomes, STRs are of particular interest because they mutate much more rapidly than most other genomic elements. As a result, they're extremely informative for genomic identification, ancestry inference and genealogy.

Despite their utility, STRs are particularly difficult to genotype. The repetitive sequence responsible for their high mutability also results in frequent alignment errors that can complicate and bias downstream analyses. In addition, PCR stutter errors often result in reads that contain additional or fewer repeat copies than the true underlying genotype. 

**HipSTR** was specifically developed to deal with these errors in the hopes of obtaining more robust STR genotypes. In particular, it accomplishes this by:

1. Learning locus-specific PCR stutter models using an [EM algorithm] (http://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm)
2. Mining candidate STR alleles from population-scale sequencing data
2. Utilizing phased SNP haplotypes to genotype, phase and/or impute STRs
3. Employing a specialized hidden Markov model to align reads to candidate sequences while accounting for stutter



## Installation
HipSTR requires a standard c++ compiler, [CMake](http://www.cmake.org/download/) as well as Java version 1.7 or later.
To obtain HipSTR and all of its associated submodules, use:

    % git clone --recursive https://github.com/tfwillems/HipSTR.git

To build, use Make:

    % cd HipSTR
    % make

On Mac, before running Make, change the line in *vcflib/smithwaterman/Makefile* from `LDFLAGS=-Wl,-s` to `LDFLAGS=-Wl`

## Quick Start
To run HipSTR in its most broadly applicable mode, run it on **all samples concurrently** using the syntax:
```
./HipSTR --bams          run1.bam,run2.bam,run3.bam,run4.bam
         --fasta         /data/
         --regions       str_regions.bed
         --stutter-out   stutter_models.txt
         --str-vcf       str_calls.vcf.gz
```
* **--bam** :  a comma-separated list of [BAM](https://samtools.github.io/hts-specs/SAMv1.pdf) files generated by any indel-sensitive aligner (e.g. [lobSTR](http://lobstr.teamerlich.org/index.html) or [BWA-MEM](http://bio-bwa.sourceforge.net/bwa.shtml)) and sorted and indexed using samtools
* **--regions** : a [BED](https://genome.ucsc.edu/FAQ/FAQformat.html#format1) file containing the coordinates for each STR region of interest
* **--fasta** : the directory containing [FASTA files] (https://en.wikipedia.org/wiki/FASTA_format) for each chromosome in the BED file. In the above usage example, if *str_regions.bed* contains chr1, chr2, and chr10, the corresponding files would be */data/chr1.fa*, */data/chr2.fa* and */data/chr10.fa*

For each region in *str_regions.bed*, **HipSTR** will:

1. Learn locus-specific stutter models and output them to *stutter_models.txt*
2. Use the stutter model and haplotype-based alignment algorithm to genotype each individual
3. Output the resulting STR genotypes to *str_calls.vcf.gz*, a [bgzipped] (http://www.htslib.org/doc/tabix.html) [VCF](http://samtools.github.io/hts-specs/VCFv4.2.pdf) file
4. The resulting VCF file will contain calls for each sample contained within any of the BAM files' read groups. *HipSTR* will automatically combine all reads for a sample using this read group information.

## In-depth usage
**HipSTR** has a variety of usage options designed to accomodate scenarios in which the sequencing data varies in terms of the number of samples and the coverage. Most scenarios will fall into one of the following categories:

1. 200 or more low-coverage (~5x) samples
    * Sufficient reads for stutter estimation
    * Sufficient reads to detect candidate STR alleles
    * **Use de novo stutter estimation + STR calling with de novo allele generation**
2. 50 or more high-coverage (~30x) samples
    * Sufficient reads for stutter estimation
    * Sufficient reads to detect candidate STR alleles
    * **Use de novo stutter estimation + STR calling with de novo allele generation**
3. Handful of low-coverage  (~5x) samples
    * Insufficient reads for stutter estimation
    * Insufficient reads to detect candidate STR alleles
    * **Use external stutter models + STR calling with a reference panel**
4. Handful of high-coverage (~30x) samples
    * Insufficient samples for stutter estimation
    * Sufficient reads to detect candidate STR alleles
    * **Use external stutter models + STR calling using with de novo allele generation**
    
#### 1. De novo stutter estimation + STR calling with de novo allele generation
This mode is identical to the one suggested in the **Quick Start** section as it suits most applications. HipSTR will output the learned stutter models to *stutter_models.txt* and the STR genotypes in bgzipped VCF format to *str_calls.vcf.gz* 
```
./HipSTR --bams             run1.bam,run2.bam,run3.bam,run4.bam
         --fasta            /data/
         --regions          str_regions.bed
         --stutter-out      stutter_models.txt
         --str-vcf          str_calls.vcf.gz
```

#### 2. External stutter models + STR calling with de novo allele generation
The sole difference in this mode is that we no longer output stutter models using the **--stutter-out** option and instead input them from a file using the **--stutter-in** file. For more details on the stutter model file format, see below. For humans, we've provided a file containing stutter models for each STR locus under PCR or PCR-free conditions at ...
```
./HipSTR --bams             run1.bam,run2.bam,run3.bam,run4.bam
         --fasta            /data/
         --regions          str_regions.bed
         --stutter-in       ext_stutter_models.txt
         --str-vcf          str_calls.vcf.gz
```

#### 3. External stutter models + STR calling with a reference panel
This analysis model is extremely similar to mode #2, except that we provide an additional VCF file containing known STR genotypes at each locus using the **--str-vcf** option. **HipSTR** will not identify any additional candidate STR alleles in the BAMs when this option is specified, so it's best to use a VCF that contains STR genotypes for a wide range of populations and individuals. For humans, we've provided such a file based on the [1000 Genomes Project](http://www.1000genomes.org/) at ... 
```
./HipSTR --bams             run1.bam,run2.bam,run3.bam,run4.bam
         --fasta            /data/
         --regions          str_regions.bed
         --stutter-in       ext_stutter_models.txt
         --ref-vcf          ref_strs.vcf.gz
         --str-vcf          str_calls.vcf.gz
```


### STR imputation


## Additional Usage Options

## Alignment Visualization
