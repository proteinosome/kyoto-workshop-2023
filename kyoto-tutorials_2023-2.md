# Welcome to the Kyoto University's long-reads workshop focusing on PacBio's HiFi sequencing

Author: Khi Pin, Chua

Updated: 2023-3-1

- [Welcome to the Kyoto University's long-reads workshop focusing on PacBio's HiFi sequencing](#welcome-to-the-kyoto-universitys-long-reads-workshop-focusing-on-pacbios-hifi-sequencing)
- [Disclaimer](#disclaimer)
- [Introduction](#introduction)
- [Housekeeping](#housekeeping)
- [Introduction to HiFi reads](#introduction-to-hifi-reads)
  - [Anything else in a sequence BAM file?](#anything-else-in-a-sequence-bam-file)
  - [QC of sequencing data on command-line](#qc-of-sequencing-data-on-command-line)
- [Alignment of HiFi reads](#alignment-of-hifi-reads)
  - [Visualizing alignment with 5mC information in IGV](#visualizing-alignment-with-5mc-information-in-igv)
- [Call small variants on the BAM file with Google DeepVariant](#call-small-variants-on-the-bam-file-with-google-deepvariant)
  - [Phasing aligned BAM files with WhatsHap](#phasing-aligned-bam-files-with-whatshap)
- [Call structural variants using pbsv](#call-structural-variants-using-pbsv)
- [Genotyping of tandem repeats using TRGT](#genotyping-of-tandem-repeats-using-trgt)
- [pb-human-wgs-workflow is a complete pipeline to characterize human genomes](#pb-human-wgs-workflow-is-a-complete-pipeline-to-characterize-human-genomes)
  - [pb-CpG-tools summarises methylation probability from multiple reads across the genome](#pb-cpg-tools-summarises-methylation-probability-from-multiple-reads-across-the-genome)
- [Bonus: Paraphase for SMN1/SMN2](#bonus-paraphase-for-smn1smn2)

# Disclaimer

This tutorial is created for the Kyoto University Symposium's workshop for long-reads sequencing and the materials below are updated to the best of our knowledge as of February 2023. Recommendations and data types/characteristics are not guaranteed to be the same in the future and may change.

# Introduction

This tutorial will guide you through using a small dataset of HiFi reads (nearby chromosome 6p23) to look at interesting variants. After this workshop, you will be able to understand what HiFi reads are and how to use both PacBio's and third party tools to analyze a human genome.

Note that we've installed SMRT Link 11.1 with the `--smrttools-only` flag which will make all workflows and softwares that come with SMRT Link available on the command line without using the graphical user interface (GUI). SMRT Link is a software that provides many useful utilities and a user-friendly interface to interact and analyze PacBio's data. For the purpose of this workshop however, we will be using commmand-line mostly to carry out more advanced downstream analysis.

# Housekeeping

All datasets are located at `/data/kpin/`. The folder structure here is similar to how we usually set it up to run the full `pb-human-wgs-workflow` snakemake workflow that is available on [PacBio's GitHub](https://github.com/PacificBiosciences/pb-human-wgs-workflow-snakemake). A complete tutorial on how to use the workflow is available [here](https://github.com/PacificBiosciences/pb-human-wgs-workflow-snakemake/blob/main/Tutorial.md). The `resources` and `reference` folders are available on request if you have your own HiFi human genomes you would like to analyze in the future. Today, we will manually carry out some of the important steps in the pipeline to better understand how it works.

Majority of tools are available as part of SMRT Link 11.1 installation at `/data/kpin/softwares/smrtlink_11.1/smrtcmds/bin`. E.g. if you export the PATH:

```
export PATH=${PATH}:/data/kpin/softwares/smrtlink_11.1/smrtcmds/bin
```

You should be able to run most of the tools in this tutorial. Try:

```
pbmm2 --version

## pbmm2 1.9.0
## 
## Using:
##   pbmm2    : 1.9.0 (commit v1.9.0-2-gbec8e28)
##   pbbam    : 2.2.0 (commit v2.2.0-1-g8c081f6)
##   pbcopper : 2.1.0 (commit v2.1.0)
##   boost    : 1.76
##   htslib   : 1.13
##   minimap2 : 2.15
##   zlib     : 1.2.11
```

There are a few other useful tools installed such as `csvtk`, `seqkit` and `datamash` that can be useful for some simple manipulation on command line. These can be activated using `conda`:

```
conda activate /home/lecturer14/miniconda3/envs/common-tools

csvtk --help
```

Now let's copy the test dataset to your own home folder for this tutorial:

```
cd ~

mkdir -p smrtcells/ready/

cp -r /data/kpin/smrtcells/ready/small_HG005 ~/smrtcells/ready/
```

# Introduction to HiFi reads

Let's take a peek at HiFi reads!

- PacBio’s BAM file is fully compatible with the SAM specification:
    <https://samtools.github.io/hts-specs/SAMv1.pdf>
- The full documentation of PacBio's BAM format is [here](https://pacbiofileformats.readthedocs.io/en/11.0/BAM.html).

```
samtools view smrtcells/ready/small_HG005/m64017_200723_190224.hifi_reads.bam |
    head -1 |
    tr '\t' '\n' |
    awk '{print "Col "FNR":", $0}' | 
    less

## Col 1: m64017_200723_190224/225/ccs
## Col 2: 4
## Col 3: *
## Col 4: 0
## Col 5: 255
## Col 6: *
## Col 7: *
## Col 8: 0
## Col 9: 0
## Col 10: CAGCCTGGCTGACAGAGCAAGACTCCATCTCAAAAAAAAAGAAAAGCACTAAACAAAAAAGACTGTCAACC...
## Col 11: zobS~Z7_a^^j\T^qPQ]z>^aWH|Kkbq^'uuuuuuusa;{qsbelcpX~~b+yyyyRHraodsaQ~Q~...
## Col 12: RG:Z:ffdb5bc6
## Col 13: ec:f:6.91816
## Col 14: np:i:6
## Col 15: rq:f:0.997783
## Col 16: sn:B:f,11.8698,17.2987,4.53111,7.83714
## Col 17: zm:i:225
## Col 18: Mm:Z:C+m,120,43,80,57,4,24,0,10,25,22,49,19,7,2,55,47,20,8,29,49,12,60,...
## Col 19: Ml:B:C,143,27,110,113,1,252,253,54,113,247,12,174,252,240,42,22,45,36,...
```

- Column 2-9 are usually meant for storing alignment information.
    However, PacBio’s BAM files from the instrument do not have any
    alignment. Hence, any values in these columns \*\*before alignment\*
    are just “placeholders” meant to comply with the SAM format.

- For CCS BAM file, two important columns are column 14 and 15. `np`
    indicates the number of full passes used to generate the CCS read,
    and `rq` represents the average per-base QV score for the read.
    Here, this read has an accuracy of `0.997783`. QV score can be
    calculated as:
     −10 * log10(1 - accuracy))
- **Tricks** In bash, we can use the `bc` tool to carry out math
    operation (Pipe the equation you want to evaluate to `bc -l`). Here
    for example, this read is QV27:

``` bash
# To let log10, divide the natural log of your value by the natural log of 10
echo '-10*l(1 - 0.999842)/l(10)' | bc -l
```

This is a `BAM` file straight out of the PacBio's instrument.

Quiz: Why do you think PacBio provides `BAM` file instead of `FASTQ`/`FASTA`?

Bonus: In the newest Revio instrument, the quality scores in `BAM` file are now binned into discrete Q values. This has minimal effect on the downstream applications but allows PacBio to compress the `BAM` file by up to 50%, saving a lot of precious disk space.

## Anything else in a sequence BAM file?

There are many useful information in the header of the BAM file:

``` bash
samtools view -H smrtcells/ready/small_HG005/m64017_200723_190224.hifi_reads.bam
```

    ## @HD  VN:1.5  SO:unknown  pb:3.0.1
    ## @RG  ID:7ae86b2d PL:PACBIO   DS:READTYPE=CCS;BINDINGKIT=101-717-300;SEQUENCINGKIT=101-644-500;BASECALLERVERSION=5.0.0;FRAMERATEHZ=100.000000 PU:m64014_190506_005857 PM:SEQUELII CM:S/P3-C1/5.0-8M
    ## @PG  ID:ccs-4.2.0    PN:ccs  VN:4.2.0    DS:Generate circular consensus sequences (ccs) from subreads.   CL:ccs ccs 1perc.bam 1perc.ccs.bam --log-level INFO --min-rq 0.9
    ## @PG  ID:samtools PN:samtools PP:ccs-4.2.0    VN:1.14 CL:samtools view -s 0.1 -bh isoseq3/alz.ccs.bam
    ## @PG  ID:samtools.1   PN:samtools PP:samtools VN:1.14 CL:samtools view -H alz.ccs.toy.bam

- The “`-H`” parameter outputs just the header of a BAM/SAM file. A
    header contains information such as the sequencing platform,
    chemistry version, sample names, and even commands used to generate
    the BAM/SAM file.
- Some PacBio’s tools rely on the header to run. E.g. Modern version
    of CCS will not be able to run if the chemistry is too old (E.g. RS
    II).
- Can you find out what platform this example dataset was sequenced on
    and what was the version of CCS used?

## QC of sequencing data on command-line

Normally, PacBio's instruments' users will use SMRT Link's (GUI) to carry out sequencing setup and QC. The Data Management interface on SMRT Link provides a nice HTML page to visualize the various sequencing metrics. However, with the command line SMRT Link installation, we can also produce many useful plots just like SMRT Link's GUI to look at the metrics of our small HG005 dataset. This is done using the `runqc-reports` tool:

```
# pbindex creats an index file for PacBio's unaligned BAM
pbindex smrtcells/ready/small_HG005/m64017_200723_190224.hifi_reads.bam

mkdir m64017_200723_190224_runqc_reports 

# Create dataset XML
dataset create --type ConsensusReadSet \
  smrtcells/ready/small_HG005/m64017_200723_190224.consensusreadset.xml \
  smrtcells/ready/small_HG005/m64017_200723_190224.hifi_reads.bam

runqc-reports smrtcells/ready/small_HG005/m64017_200723_190224.consensusreadset.xml \
  -o m64017_200723_190224_runqc_reports

## SUMMARY METRICS:
##     CCS Analysis Report (ccs.report.json):
##         HiFi Reads                 : 4252
##         HiFi Yield (bp)            : 68229950
##         HiFi Read Length (mean, bp): 16046
##         HiFi Read Quality (median) : Q28
##         HiFi Read Quality (median) : 28
```

Bonus: Can we use samtools to calculate the mean number of passes? Yes, we can!

```
samtools view smrtcells/ready/small_HG005/m64017_200723_190224.hifi_reads.bam | awk '{gsub("np:i:","",$14); total+=$14;count++} END {print total/count}'
```

Note that this is an older dataset in 2020. Our chemistry and softwares had improved and in most of the newer dataset you should see a median read quality of around Q33 to Q34 for a standard human WGS library (~15 kbp insert). There are many new datasets available on PacBio's [website](https://www.pacb.com/connect/datasets/) including those from the latest Revio instrument.

Bonus: There's a small Revio HG002 dataset in the `/data/kpin/revio_hg002` folder. Can you QC this dataset and look at the data quality in the latest dataset?

# Alignment of HiFi reads

As introduced, HiFi reads are both long and extremely accurate. While HiFi reads are highly accurate (on average Q30, or 99.9% accurate), the error profile is different from that of short-reads sequencing. As such, using alignment tools commonly used for short-reads may not provide satisfactory results (e.g. `bwa mem`). The most well-established long-reads aligment tool is called `minimap2` (available on [GitHub](https://github.com/lh3/minimap2)). You can use `minimap2` to align HiFi reads `FASTQ/FASTA`.

However, to make it easier for customers and to standardize best practices, PacBio has adapted `minimap2` into `pbmm2`, and recommend customers to use `pbmm2` for HiFi reads alignment. Let's align our small dataset to the human genome. Note that with `pbmm` you can directly align PacBio's `BAM` file without having to convert it into `FASTQ`/`FASTQ`

```
# --sort will sort the output automatically
# --unmapped will keep unmapped record
# --log-level and --log-file parameters allow more verbose logs
# into a file for troubleshooting purpose if needed
# Note we've already generated the mmi index for hg38 here to
# speed up the analysis, but you can also align to the hg38 FASTA
# directly (slower as it'll need to generate the index first)

pbmm2 align --num-threads 2 \
  --preset CCS \
  --sort --unmapped \
  --sample small_HG005 \
  --log-level INFO --log-file pbmm2.log \
  /data/kpin/reference/human_GRCh38_no_alt_analysis_set.mmi \
  smrtcells/ready/small_HG005/m64017_200723_190224.hifi_reads.bam \
  m64017_200723_190224.hg38.bam
```

Bonus: Take a look at the bam file with `samtools` similar to above and see what has changed.

## Visualizing alignment with 5mC information in IGV

In IGV (version 2.14 and above), you can visualize 5mC methylation in the alignment as long as the `MM` and `MC` tags exist. Let's download the example aligned `BAM` at `/data/kpin/example_output/small_HG005.GRCh38.deepvariant.haplotagged.bam` and its corresponding `bai` index, then open it in `IGV`.

# Call small variants on the BAM file with Google DeepVariant

With the reads now aligned to hg38 genome, we can use Google DeepVariant which is known as the best small variants caller for HiFi reads to call SNPs and indels in our example dataset. The latest DeepVariant v1.5.0 even comes with a model that works for the latest `Revio` instrument.

DeepVariant can be run using `singularity` as a `docker` container. We have already "pulled" the latest `DeepVariant` docker container using `singularity` to a local SIF image at
`/data/kpin/softwares/deepvariant_latest.sif` so you can directly run DeepVariant using the image:

```
# We need to index the aligned bam file
samtools index m64017_200723_190224.hg38.bam

singularity exec --bind /usr/lib/locale/,/data/kpin/reference \
  /data/kpin/softwares/deepvariant_latest.sif \
  /opt/deepvariant/bin/run_deepvariant \
  --model_type PACBIO \
  --ref /data/kpin/reference/human_GRCh38_no_alt_analysis_set.fasta \
  --reads m64017_200723_190224.hg38.bam \
  --output_vcf m64017_200723_190224.deepvariant.vcf \
  --num_shards 2 \
  --regions chr6
```

The command above will call small variants in chromosome 6 using hg38 as the reference. Binding `/usr/lib/locale/` prevents locale-related issue and binding `/data/kpin/reference` allows `singularity` access to the reference folder. This process will too long to run for the workshop, so we will use the example output VCF already generated at `/data/kpin/example_output/small_HG005.GRCh38.deepvariant.vcf.gz`.

Let's take a look at the example VCF file:

```
bcftools view /data/kpin/example_output/small_HG005.GRCh38.deepvariant.vcf.gz | less
```

If you are familiar with VCF format, DeepVariant's output should be familiar to you. Note that by default DeepVariant provides all calls including variants that do not pass its internal heuristics. To look at how many variants were called, we can filter using the "FILTER" column:

```
bcftools filter -i 'FILTER="PASS"' /data/kpin/example_output/small_HG005.GRCh38.deepvariant.vcf.gz | grep -v "^#" | wc -l
```

## Phasing aligned BAM files with WhatsHap

After small variants calling, a typical step here would be to phase the aligned `BAM` file using the called variants. The recommended software currently is [WhatsHap](https://whatshap.readthedocs.io/en/latest/guide.html) and you can phase the `BAM` file with the following command:

```
whatshap phase --indels \
  --chromosome chr6 \
  --output small_HG005.GRCh38.deepvariant.phased.vcf.gz \
  --reference /data/kpin/reference/human_GRCh38_no_alt_analysis_set.fasta \
  /data/kpin/example_output/small_HG005.GRCh38.deepvariant.vcf.gz \
  m64017_200723_190224.hg38.bam

# Get phasing statistics
whatshap stats --gtf whatshap.phaseblock.gtf \
  --tsv whatshap.stats.tsv \
  --block-list whatshap.blocklist \
  --chr-lengths /data/kpin/reference/human_GRCh38_no_alt_analysis_set.chr_lengths.txt \
  small_HG005.GRCh38.deepvariant.phased.vcf.gz

# Index the VCF file first
tabix small_HG005.GRCh38.deepvariant.phased.vcf.gz

# Tag the haplotypes in aligned BAM file
whatshap haplotag \
  --output-threads 2 \
  --output m64017_200723_190224.hg38.haplotagged.bam \
  --reference /data/kpin/reference/human_GRCh38_no_alt_analysis_set.fasta \
  small_HG005.GRCh38.deepvariant.phased.vcf.gz \
  m64017_200723_190224.hg38.bam
```

Note that as we're using a subset of chromosome 6 for demonstration purpose, the phase block is not actually correctly calculated and is not representative of real world performance.

The `haplotag` command will add a `HP` tag into the aligned `BAM` file that can be used by IGV to visualize or group reads into the phased haplotype. Let's try that in IGV and zoom into `ATXN1`, a gene related to ataxia.

# Call structural variants using pbsv

PacBio develops a tool called `pbsv` that is often used to call structural variants in HiFi sequencing data. There are two steps in `pbsv`, one is called `discover` and another one called `call`. `pbsv discover` is used to look for any potential structural variants "signatures" in the alignment BAM file. Signatures are then clustered together to group breakpoints close to each other. Finally, `pbsv call` is used to call SVs based on the signatures. `pbsv` can also carry out joint calling from multiple samples to increase sensitivity.

Let's try running `pbsv` on our example dataset.

```
pbsv discover --hifi \
  --log-level INFO \
  --log-file pbsv-discover.log \
  --region chr6 \
  --tandem-repeats /data/kpin/reference/human_GRCh38_no_alt_analysis_set.trf.bed \
  m64017_200723_190224.hg38.bam \
  pbsv.svsig.gz 

pbsv call --hifi \
  -m 20 \
  --log-level INFO \
  --log-file pbsv-call.log \
  --num-threads 4 \
  /data/kpin/reference/human_GRCh38_no_alt_analysis_set.fasta \
  pbsv.svsig.gz \
  pbsv.vcf
```

The types of SV called by `pbsv` is documented in details on PacBio's GitHub [page](https://github.com/PacificBiosciences/pbsv). Let's visit the GitHub page to understand `pbsv` in more details.

# Genotyping of tandem repeats using TRGT

Recently, PacBio has developed a new tool ([GitHub](https://github.com/PacificBiosciences/trgt)) specifically to genotype tandem-repeats. This includes both STR and VNTR. TRGT takes in a bed file indicating regions in the genome that are known to be polymorphic in both repeats motif and length. A companion tool called TRVZ can then be used to visualize the repeat structures as well as 5mC methylation around the repeats.

There are on-going efforts to provide a more comprehensive catalogue of repeats. The bed file used here is a set of repeats loci discovered to be polymorphic from a set of short-reads genomes previously. This also includes a set of 56 known loci known to be potentially pathogenic if the repeats are expanded. These are for example ATXN1, FMR1, RFC1 etc.

Let's try to genotype our demo dataset. This dataset only contains a small subset of the whole genome but is useful to demonstrate how to use TRGT and TRVZ.

```
/data/kpin/softwares/trgt --threads 4 \
  --genome /data/kpin/reference/human_GRCh38_no_alt_analysis_set.fasta \
  --repeats /data/kpin/reference/pathogenic_repeats.hg38.bed \
  --reads m64017_200723_190224.hg38.haplotagged.bam \
  --output-prefix m64017_200723_190224.trgt

samtools sort m64017_200723_190224.trgt.spanning.bam > m64017_200723_190224.trgt.spanning.sorted.bam

samtools index m64017_200723_190224.trgt.spanning.sorted.bam

/data/kpin/softwares/trvz \
  --genome /data/kpin/reference/human_GRCh38_no_alt_analysis_set.fasta \
  --image ATXN1.png --repeat-id ATXN1 \
  --repeats /data/kpin/reference/pathogenic_repeats.hg38.bed \
  --spanning-reads m64017_200723_190224.trgt.spanning.sorted.bam \
  --vcf m64017_200723_190224.trgt.vcf.gz
```

There are many useful information for each of the repeat loci genotyped. For example, `AL` tells us how long the repeats are for each haplotype, and `SD` provides the depth supporting each allele. The definitions of each tag is documented [here](https://github.com/PacificBiosciences/trgt/blob/main/docs/vcf_files.md)

Bonus: Take a look at the Revio dataset. Genotype the pathogenic repeat in this dataset and visualize the repeat structure.

# pb-human-wgs-workflow is a complete pipeline to characterize human genomes

As you can see, there are various steps to characterize a human genome, and we have not even annotated the variants! This is why PacBio maintains a `Snakemake` workflow called [pb-human-wgs-workflow](https://github.com/PacificBiosciences/pb-human-wgs-workflow-snakemake) that includes all the above steps we have learned and many other steps including quality control (e.g. depth of coverage), variants annotation (consequence and genes' names) and joint variant calling (e.g. trio).

The pipeline allows reproducible research and once it is set up, it can process many samples in scale by utilizing large computing clusters because `Snakemake` natively supports many different job schedulers.

Let's take a look at the output of this workflow provided on pb-human-wgs-workflow's output documentation page and our example output. The analysis has already been done for you, but if you are interested, this was how it was done:

```
bash workflow/process_smrtcells.local.sh

bash workflow/process_sample.local.sh small_HG005
bash workflow/process_sample.local.sh small_HG006
bash workflow/process_sample.local.sh small_HG007

bash workflow/process_cohort.local.sh small_HG005_trio
```

By splitting the workflow into multiple stages, users can process sequencing output (SMRT Cells) individually whever they receive the data, then process the full sample when all the sequencing data for that sample has been collected. Finally, the `process_cohort` step takes in a `cohort.yaml` file specifying the relationships between all samples and carry out variants annotation and filtering. Note that despite the name, `process_cohort` can also be done on a singleton for variants annotation and filtering.

Bonus: Can we use the output of `process_cohort` to look for any interesting structural variant and visualize them in IGV?

## pb-CpG-tools summarises methylation probability from multiple reads across the genome

The 5mC probability in BAM file we saw came from multiple reads. While it is incredible that we can actually look at single-molecule methylation, we often want to make use of all molecules in order to decide for example if a CpG island is hyper- or hypo-methylated. This is where [pb-CpG-tool](https://github.com/PacificBiosciences/pb-CpG-tools) comes into the picture. In the pb-human-wgs-workflow's output folder, you can find a folder called `5mC_cpg_pileup`. Download the `/data/kpin/samples/small_HG005/5mc_cpg_pileup/small_HG005.GRCh38.hap[12].denovo.bed` bed files (both hap1 and hap2) and open it in your IGV that has the aligned `BAM` file opened.

# Bonus: Paraphase for SMN1/SMN2

Paraphase is another tool recently developed by PacBio to resolve complex genes loci in the human genome that are relevant to many important diseases. The tool currently supports resolving the highly complex SMN1/SMN2 that differ from each other mostly in a single nucleotide, i.e. c.840C (SMN1) and c.840T (SMN2). Paraphase is freely available on PacBio's [GitHub](https://github.com/PacificBiosciences/paraphase). Let's try using Paraphase on the example HG01175 data (If you clone the GitHub for Paraphase, it comes with a set of examples `BAM` file including HG01175).

```
conda activate /home/lecturer14/miniconda3/envs/paraphase

paraphase \
  -b /data/kpin/softwares/paraphase/examples/data/HG01175.bam \
  -o HG01175_paraphase
```

Let's visualize the SMN1/SMN2 haplotypes in IGV.
