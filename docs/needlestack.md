<img align="center" src="https://cloud.githubusercontent.com/assets/3366818/12507528/99afd6fa-c0f6-11e5-993b-5ac925d178fa.png" width="700">

# A multi-sample somatic variant caller

<img align="center" src="https://cloud.githubusercontent.com/assets/3366818/10489240/7da79144-729c-11e5-8cb3-0225106d9b06.jpg" width="600">

[![Join the chat at https://gitter.im/iarcbioinfo/needlestack](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/iarcbioinfo/needlestack?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![Circle CI](https://circleci.com/gh/IARCbioinfo/needlestack/tree/master.svg?style=shield&circle-token=402d456a7c50af352bb4e1a52425ce0fe645f78f)](https://circleci.com/gh/IARCbioinfo/needlestack/tree/master) [![Docker Hub](https://img.shields.io/badge/docker-ready-blue.svg)](https://hub.docker.com/r/iarcbioinfo/needlestack/) 

Warning: development in progress, unreliable results warranted. 

Please wait upcoming publication before using it in production. 

Contact: follm@iarc.fr

Needlestack development is support by the US National Cancer Institute (grant number R21CA175979) and the French Institut national du cancer.

## Description

Needlestack is an ultra-sensitive multi-sample variant caller for Next Generation Sequencing (NGS) data. It is based on the idea that analysing several samples together can help estimate the distribution of sequencing errors to accurately identify variants. It has been initially developed for somatic variant calling using very deep NGS data from circulating free DNA, but is also applicable to lower coverage data like Whole Exome Sequencing (WES) or even Whole Genome Sequencing (WGS). It is a highly scalable and reproducible pipeline thanks to the use of [nextflow](http://www.nextflow.io/) and [docker](https://www.docker.com) technologies. 

Here is a summary of the method:

- At each position and for each candidate variant, we model sequencing errors using a negative binomial regression with a linear link and a zero intercept. The data is extracted from the BAM files using [samtools](http://www.htslib.org).
- Genetic variants are detected as being outliers from the error model. To avoid these outliers biasing the regression we use a robust estimator for the negative binomial regression (published [here](http://www.ncbi.nlm.nih.gov/pubmed/25156188) with code available [here](https://github.com/williamaeberhard/glmrob.nb)).
- We calculate for each sample a p-value for being a variant (outlier from the regression) that we further transform into q-values to account for multiple testing.

## Input

- A set of [BAM files](https://samtools.github.io/hts-specs/) (called `*.bam`) grouped in a single folder along with their [index files](http://www.htslib.org/doc/samtools.html) (called `*.bam.bai`). A minimum of 20 BAM files is recommended. 
- A reference [fasta file](https://en.wikipedia.org/wiki/FASTA_format) (eventually compressed with [bgzip](http://www.htslib.org/doc/tabix.html)) along with its [faidx index](http://www.htslib.org/doc/faidx.html) (and `*.gzi` faidx index if compressed).
- Optionally a [bed file](https://genome.ucsc.edu/FAQ/FAQformat.html#format1), otherwise the variant calling is performed on the whole reference provided.

## Quick start

Needlestack works under most Linux distributions and Apple OS X.

1. Install [java](https://java.com/download/) JRE if you don't already have it (7 or higher).

2. Install [nextflow](http://www.nextflow.io/).

	```bash
	curl -fsSL get.nextflow.io | bash
	```
	And move it to a location in your `$PATH` (`/usr/local/bin` for example here):
	```bash
	sudo mv nextflow /usr/local/bin
	```
3. Install [docker](https://www.docker.com).
	
	This is very system specific (but quite easy in most cases), follow  [docker documentation](https://docs.docker.com/installation/). Also follow the optional configuration step called `Create a Docker group` in their documentation.

4. Optionally download a sample dataset.

	```bash
	git clone --depth=1 https://github.com/mfoll/NGS_data_test.git
	```
5. Run the pipeline.
	
	Here on the example dataset downloaded above:
	```bash
	cd NGS_data_test/1000G_CEU_TP53/
	nextflow run iarcbioinfo/needlestack -with-docker  \
	         --bed TP53_all.bed --bam_folder BAM/ --fasta_ref 17.fasta.gz
	```
	
	You will find a [VCF file](https://samtools.github.io/hts-specs/) called `all_variants.vcf` in the `BAM/` folder once done.
	
	The first time it will take more time as the pipeline will be downloaded from github and the docker container from [dockerhub](https://hub.docker.com/r/iarcbioinfo/needlestack/).

	Creating an alias for the long command above can be useful. For example:
	```bash
	alias needlestack='nextflow run iarcbioinfo/needlestack -with-docker'
	```
	
	If you want to permanantly add this alias (and not just for your current session), add the above  line to your `~/.bashrc` file (assuming you are using bash).
	
	It will allow you to do this:
	```bash
	needlestack --bed TP53_all.bed --bam_folder BAM/ --fasta_ref 17.fasta.gz
	```
	
6. Update the pipeline

	You can update the nextflow sofware and the pipeline itself simply using:
	```bash
	nextflow -self-update
	nextflow pull iarcbioinfo/needlestack
	```

	You can also automatically update the pipeline when you run it by adding the option `-latest` in the `nextflow run` command. Doing so you will always run the latest version from [Github](https://github.com/iarcbioinfo/needlestack).

	Official releases can be found [here](https://github.com/iarcbioinfo/needlestack/releases/). There is a corresponding official [docker container](https://hub.docker.com/r/iarcbioinfo/needlestack/) for each release and one can run a particular version using (for example for v0.3):
	```bash
	nextflow run iarcbioinfo/needlestack -r v0.3 -with-docker \
	         --bed TP53_all.bed --bam_folder BAM/ --fasta_ref 17.fasta.gz
	```

## Detailed instructions

### Nextflow and Docker

If you can't install [docker](https://www.docker.com) or don't want to use it, the pipeline will also work if you install [perl](https://www.perl.org),  [bedtools](http://bedtools.readthedocs.org/en/latest/), [samtools](http://www.htslib.org) and Rscript from [R](https://www.r-project.org) and put them in your path (executables are assumed to be respectively called `perl`, `bedtools`, `samtools` and `Rscript`). In this case, remove the `-with-docker` option from step 5 above.

The exact same pipeline can be run on your computer or on a HPC cluster, by adding a [nextflow configuration file](http://www.nextflow.io/docs/latest/config.html) to choose an appropriate [executor](http://www.nextflow.io/docs/latest/executor.html). For example to work on a cluster using [SGE scheduler](https://en.wikipedia.org/wiki/Oracle_Grid_Engine), simply add a file named `nextflow.config` in the current directory (or `~/.nextflow/config` to make global changes) containing:  
```java
process.executor = 'sge'
```

Other popular schedulers such as LSF, SLURM, PBS, TORQUE etc. are also compatible. See the nextflow documentation [here](http://www.nextflow.io/docs/latest/executor.html) for more details. Also have a look at the [other parameters for the executors](http://www.nextflow.io/docs/latest/config.html#scope-executor), in particular `queueSize` that defines the number of tasks the executor will handle in a parallel manner. Parallelism in needlestack is managed by splitting the genomic regions in pieces of equal sizes (`--nsplit`). Note that dealing with very large regions can take a large amount of memory, therefore splitting more is more memory-efficient. 

### Parameters

Type `--help` to get the full list of options. `--bam_folder` and `--fasta_ref` are compulsary. The optional parameters with default values are:

| Parameter | Default value | Description |
|-----------|--------------:|-------------| 
| min_dp    |            30 | Minimum median coverage to consider a site. In addition, at least 10 samples have to be covered by min_dp. |
| min_ao | 3 | Minimum number of non-ref reads in at least one sample to consider a site |
| nsplit | 1 | Split the bed file in nsplit pieces and run in parallel |
| min_qval | 50 | qvalue threshold in [Phred scale](https://en.wikipedia.org/wiki/Phred_quality_score) to consider a variant |
| sb_type | SOR | Strand bias measure, either SOR, RVSB or FS |
| sb_snv | 100 or 1000 | Strand bias threshold for SNVs (100 (1000 if FS) = no filter) |
| sb_indel | 100 or 1000 | Strand bias threshold for indels (100 (1000 if FS) = no filter)|
| map_qual | 20 | Min mapping quality (passed to samtools) |
| base_qual | 20 | Min base quality (passed to samtools) |
| max_DP | 30000 | Downsample coverage per sample (passed to samtools) |
| use_file_name |   | Put this argument to use the bam file names as sample names. By default the sample name is extracted from the bam file SM tag. |
| all_SNVs |   | Put this argument to output all SNVs, even when no variant is detected. Note that positions with zero coverage for all samples might still be missing depending on how the region split is performed |
| no_plots |   | Put this argument to remove pdf plots of regressions from the output |
| no_labels |   | Put this argument for not labeling the outliers on regression plots |
| no_indels |   | Put this argument to do not perform the variant calling on insertions and deletions |
| no_contours |   | Put this argument to do not plot qvalues contours (for qvalue threshold={10,30,50,70,100} by default) and do not plot minimum detectable allelic fraction in function of coverage |
| out_folder | --bam_folder | Output folder, by default equals to the input bam folder |
| out_vcf | all_variants.vcf | File name of final VCF |
| bed |   | BED file containing a list of regions (or positions) where needlestack should be run |
| region |   | A region in format CHR:START-END where calling should be done |
| pairs_file | | A tab-delimited file containing two columns (normal and tumor sample names) for each sample in line. This enables matched tumor/normal pair calling features (see below) |
| power_min_af |  | Allelic fraction used to classify genotypes to 0/0 or ./. depending of the power to detect a variant at this fraction (see below) |
| extra_robust_gl | | Add this argument to perform extra-robust regression (useful for common germline SNPs, see below) |
| sigma_normal | 0.1 | Sigma parameter for negative binomial modeling of expected germline allelic fraction. We strongly recommend not to change this parameter unless you really know what it means |
| input_vcf |   | A VCF file (from GATK) where calling should be done. Needlestack will extract DP and AO from this VCF (DP and AD fields) and annotate it with phred q-value score (`FORMAT/QVAL` field), error rate (`INFO/ERR`) and over-dispersion sigma parameter (`INFO/SIG`). WARNING: by default, only work with split (coreutils) version > 8.13 |

By default, if neither `--bed` nor `--region` are provided, needlestack would run on whole reference, building a bed file from fasta index inputed.
If `--bed` and `--region` are both provided, it should run on the region only.

Simply add the parameters you want in the command line like `--min_dp 1000` for example to change the min coverage or `--all_SNVs` to output all sites.

### Germline, somatic, matched Tumor-Normal pairs calling and contamination

When using matched tumor/normal, Needlestack can classify variants (VCF `FORMAT/STATUS` field) according to the following table:
![Status Table](STATUS_TABLE.png)

For this one need to provide a tab-delimited file containing two columns with normal and tumor sample names using the `--pairs_file` option. The first line of this file is a header with `TUMOR` and `NORMAL` keywords. When one normal or one tumor is missing, one can write `NA`. In this mode, the parameter `power_min_af` defines the allelic fraction in the tumor one is trying to detect to classify genotypes as `./.` or `0/0` depending on the power to detect this allelic fraction. Variants found as somatic in a tumor, but germline in another sample of the series will be flagged as `POSSIBLE_CONTAMINATION`. We found this particularly important, as needlestack is very sensitive to low allelic fractions, to filter out contamination among samples for pooled exome capture.

In other cases (when there is no `--pairs_file` parameter defined), genotypes are defined as `./.` or `0/0` assuming one is looking for allelic fractions expected for germline variants (negative binomial distribution centered at 0.5 with over-dispersion parameter sigma=`sigma_normal`, with `sigma_normal=0.1` by default). If you are looking for somatic variants without matched-normal and assuming you are interesting to correctly distinguish `./.` and `0/0`genotypes, you can set the `power_min_af` parameter to the lowest allelic fraction of somatic variants you are interested with (and your coverage allows you to find).  Note that this is by far not the most common situation, and that in most cases you don't have to worry about the `power_min_af` parameter.

## Notes

### Common variants

Needlestack is made to identify rare variants (i.e. only a few samples in your set of samples have a particular variant), because of the robust regression used. Therefore, common SNPs (>10%) or strong somatic hotspots will be missed. The optional `extra_robust_gl` can overcome partially this issue for common germline mutations: it first discard high allelic fraction (>20%, assuming these are likely true variants) before fitting the regression model when between 10% and 50% of sample have such a high allelic fraction. A flag is written in the VCF `INFO/WARN` field when this happened (`EXTRA_ROBUST_GL`). Additionally, when an other allele than the reference allele is the most common, it is taken as the reference allele and a flag is also written in the VCF (`INFO/WARN=INV_REF`).

### Strand bias

For conventional variant callers, GATK WES/WGS [recommended values](http://gatkforums.broadinstitute.org/discussion/5533/strandoddsratio-computation) for SOR strand bias are SOR < 4 for SNVs and < 10 for indels. We haven't found this particularly useful, but a more detailed evaluation is necessary. For amplicon based targeted sequencing, RVSB>0.85 seems to reduce some erros. There is no hard filter by default as this is easy to do afterward using [bcftools filter](http://samtools.github.io/bcftools/bcftools.html#filter) command.

### Reproducibility

A good practice is to keep (and publish) the `.nextflow.log` file create during the pipeline process, as it contains useful information for reproducibility (and for debugging in case of problem). You should keep the `trace.txt` file containing even more information to keep for records. Nextflow also creates a nice processes execution timeline file (a web page) in `timeline.html`.

## Pipeline execution DAG
<img align="center" src="https://cloud.githubusercontent.com/assets/3366818/15250619/eb6afcea-1925-11e6-9f91-e1d6ceecbe19.jpg" width="600">
