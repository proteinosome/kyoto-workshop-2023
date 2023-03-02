PacBio Data Hands-on
================
Khi Pin, Chua
19/01/2022

- [PacBio Data Hands-on](#pacbio-data-hands-on)
  - [Using `Conda` environment](#using-conda-environment)
  - [What tools do we have?](#what-tools-do-we-have)
  - [FASTQ and FASTA file](#fastq-and-fasta-file)
  - [What’s in a PacBio HiFi bam file?](#whats-in-a-pacbio-hifi-bam-file)
  - [Anything else in a sequence BAM file?](#anything-else-in-a-sequence-bam-file)
  - [Aligned BAM file](#aligned-bam-file)
  - [SMRT Link without GUI and `pbcromwell`](#smrt-link-without-gui-and-pbcromwell)
    - [Installing SMRT Link command line tools](#installing-smrt-link-command-line-tools)
    - [Reproducible and easy pipeline with `pbcromwell`](#reproducible-and-easy-pipeline-with-pbcromwell)

## Using `Conda` environment

-   Anaconda is a package used to manage software packages and their
    dependencies.
-   Anaconda “pull” packages from “repositories” and automatically find
    out what are the softwares or libraries needed to make sure the
    software you want to use can work.
    -   Instead of installing 10 packages that’s required before you can
        use `pbmm2` for example, `conda install -c bioconda pbmm2` will
        take care of that for you (using `bioconda` as the main package
        repository)
-   In addition, sometimes different softwares can conflict with each
    other. E.g. software 1 requires libraryX version 1.1 but software 2
    requires libraryX version 1.2. `Conda` solves this problem by
    creating “virtual environment” so you can have software 1 in its own
    “container” that contains library X version 1.1 and software 2 in
    another “container” that contains library X version 1.2. As a
    result, they will not conflict with each other.
-   You can create an environment called ENVX with
    `conda create -n ENVX`. In this workshop, we’ve created 2
    environment that you will use. One is called `isoseqtoy` and another
    one called `SQANTI3.env`.
-   **Tips** `conda install` can sometimes take a long time to resolve
    dependencies. Check out `mamba`:
    <https://github.com/mamba-org/mamba> that can speed up the process
    (by a large margin most of the time!)

## What tools do we have?

-   `pbmm2`, `lima`, `isoseq3` and `samtools` are command line tools
    that can be installed via Anaconda
-   We’ve already installed these for you (You can try to install it on
    your own HPC after the workshop!).
-   You can go into the `isoseqtoy` environment using the command
    `conda activate isoseqtoy`.

``` bash
conda activate isoseqtoy
# Check the versions of the softwares
pbmm2 --version
lima --version
isoseq3 --version
samtools --version
```

    ## Could not find conda environment: isoseqtoy
    ## You can list all discoverable environments with `conda info --envs`.
    ## 
    ## pbmm2 1.7.0 (commit 1.7.0)
    ## lima 2.2.0 (commit v2.2.0)
    ## isoseq3 3.4.0 (commit v3.4.0)
    ## samtools 1.14
    ## Using htslib 1.14
    ## Copyright (C) 2021 Genome Research Ltd.
    ## 
    ## Samtools compilation details:
    ##     Features:       build=configure curses=yes 
    ##     CC:             gcc
    ##     CPPFLAGS:       
    ##     CFLAGS:         -Wall -g -O2
    ##     LDFLAGS:        
    ##     HTSDIR:         htslib-1.14
    ##     LIBS:           
    ##     CURSES_LIB:     -lncursesw
    ## 
    ## HTSlib compilation details:
    ##     Features:       build=configure plugins=no libcurl=yes S3=yes GCS=yes libdeflate=no lzma=yes bzip2=yes htscodecs=1.1.1-1-ged325d7
    ##     CC:             gcc
    ##     CPPFLAGS:       
    ##     CFLAGS:         -Wall -g -O2 -fvisibility=hidden
    ##     LDFLAGS:        -fvisibility=hidden 
    ## 
    ## HTSlib URL scheme handlers present:
    ##     built-in:     preload, data, file
    ##     S3 Multipart Upload:  s3w, s3w+https, s3w+http
    ##     Amazon S3:    s3+https, s3+http, s3
    ##     Google Cloud Storage:     gs+http, gs+https, gs
    ##     libcurl:  imaps, pop3, gophers, http, smb, gopher, sftp, ftps, imap, smtp, smtps, rtsp, scp, ftp, telnet, mqtt, https, smbs, tftp, pop3s, dict
    ##     crypt4gh-needed:  crypt4gh
    ##     mem:  mem

-   To go back to the previous environment (i.e. before
    `conda activate ENV`), you can type `conda deactivate`. To go back
    to the base environment, type `conda activate base`.

## FASTQ and FASTA file

-   FASTQ and FASTA files are common file format used in many sequencing
    platforms and universally accepted by many tools. Let’s look at a
    FASTA file first:

``` bash
# Go into the workshop directory
cd ~/pacbio_data_cli
# The FASTQ and FASTA files are compressed using Gzip. You can use
# zcat instead of cat to look at gzipped text file. 
zcat alz.ccs.toy.fasta.gz | 
  head | 
  cut -c -120
```

    ## >m64014_190506_005857/85/ccs
    ## AAGCAGTGGTATCAACGCAGAGTACTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTGAGG
    ## AAAACCCGGTAATGATGTCGGGGTTGAGGGATAGGAGGAGAATGGGGGATAGGTGTATGA
    ## ACATGAGGGTGTTTTCTCGTGTGAATGAGGGTTTTATGTTGTTAATGTGGTGGGTGAGTG
    ## AGCCCCATTGTGTTGTGGTAAATATGTAGAGGGAGTATAGGGCTGTGACTAGTATGTTGA
    ## GTCCTGTAAGTAGGAGAGTGATATTTGATCAGGAGAACGTGGTTACTAGCACAGAGAGTT
    ## CTCCCAGTAGGTTAATAGTGGGGGGTAAGGCGAGGTTAGCGAGGCTTGCTAGAAGTCATC
    ## AAAAAGCTATTAGTGGGAGTAGAGTTTGAAGTCCTTGAGAGAGGATTATGATGCGACTGT
    ## GAGTGCGTTCGTAGTTTGAGTTTGCTAGGCAGAATAGTAATGAGGATGTAAGTCCGTGGG
    ## CGATTATGAGAATGACTGCGCCGGTGAAGCTTCAGGGGGTTTGGATGAGAATGGCTGTTA

-   A FASTQ file is similar to FASTA, but with the information of
    per-base quality (encoded in phred-scale using ASCII character. See
    for example:
    <https://www.drive5.com/usearch/manual/quality_score.html>) and
    strand information (Not relevant for PacBio’s HiFi).

``` bash
cd ~/pacbio_data_cli
# Now let's look at a FASTQ version
zcat alz.ccs.toy.fastq.gz | 
  head -8 | 
  cut -c -120
```

    ## @m64014_190506_005857/85/ccs
    ## AAGCAGTGGTATCAACGCAGAGTACTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTGAGGAAAACCCGGTAATGATGTCGGGGTTGAGGGATAGGAGGAGAATGGGGGATAGGTGTATGA
    ## +
    ## ~~~~~~~~~~~~~~~~~~~~~~~~~%~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ## @m64014_190506_005857/2592/ccs
    ## AAGCAGTGGTATCAACGCAGAGTACTTTTTTTTTTTTTTTTTTTTTTTTTTTTTGAATTCCATCAGATTTACTATACGGAACATCAGTAGTGACAGATTGCACTTCTTACTTAATAACAG
    ## +
    ## ~~~~~~~~~~~~~~~~~~~~~~~~~+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## What’s in a PacBio HiFi bam file?

-   SAM stands for “Sequence Alignment Format/Map” which is a format
    used to store sequencing data.
-   BAM is a binary-compressed version of SAM file so it has a much
    lower file-size.
-   PacBio’s BAM file is fully compatible with the SAM specification:
    <https://samtools.github.io/hts-specs/SAMv1.pdf>
-   `samtools` is an industry-standard tool to view/manipulate BAM/SAM
    formats. For example, you can look at the alignment using
    `samtools view` (works with both SAM and BAM format)
-   Output from `samtools view` are tab-delimited, meaning each column
    is separated by tab and can be treated like a “tsv” file.
-   Let’s use some of the command line knowledges we’ve gained from the
    previous course:
    -   `|` is the pipe operator used to chain multiple commands
        together.
    -   `head` is used to read just the first record of the BAM.
    -   `tr` is used to change “tabs” (represented by “`\t`”) into new
        line (represented by “`\n`”) so we can see each column on a
        separate line.
    -   `awk` is a very powerful tool for text manipulation. Here, we
        use it to add “Column X” in front of each of the column of the
        sequence alignment
    -   PacBio sequences are long! We use `cut -c -100` to trim the
        output to just 100 characters so it’s easier to view

``` bash
# Here's an example CCS BAM file
samtools view alz.ccs.toy.bam | 
  head -1 | 
  tr '\t' '\n' | 
  awk 'str_line="Column-"FNR":"{print str_line,$0}' OFS=$'\t' |
  cut -c -100
```

    ## Column-1:    m64014_190506_005857/85/ccs
    ## Column-2:    4
    ## Column-3:    *
    ## Column-4:    0
    ## Column-5:    255
    ## Column-6:    *
    ## Column-7:    *
    ## Column-8:    0
    ## Column-9:    0
    ## Column-10:   AAGCAGTGGTATCAACGCAGAGTACTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTGAGGAAAACCCGGTAATGATGTCGGGGTTGAGG
    ## Column-11:   ~~~~~~~~~~~~~~~~~~~~~~~~~%~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ## Column-12:   RG:Z:7ae86b2d
    ## Column-13:   ec:f:29.8258
    ## Column-14:   np:i:29
    ## Column-15:   rq:f:0.999842
    ## Column-16:   sn:B:f,15.6053,22.4039,5.90035,10.4158
    ## Column-17:   zm:i:85

-   Column 2-9 are usually meant for storing alignment information.
    However, PacBio’s BAM files from the instrument do not have any
    alignment. Hence, any values in these columns \*\*before alignment\*
    are just “placeholders” meant to comply with the SAM format.
-   **Question**: Why does PacBio uses BAM format instead of just
    `FASTQ/FASTA`?
-   For CCS BAM file, two important columns are column 14 and 15. `np`
    indicates the number of full passes used to generate the CCS read,
    and `rq` represents the average per-base QV score for the read.
    Here, this read has an accuracy of `0.999842`. QV score can be
    calculated as:
     − 10 \* *l**o**g*<sub>10</sub>(1−*A**c**c**u**r**a**c**y*)
-   **Tricks** In bash, we can use the `bc` tool to carry out math
    operation (Pipe the equation you want to evaluate to `bc -l`). Here
    for example, this read is QV38:

``` bash
# To let log10, divide the natural log of your value by the natural log of 10
echo '-10*l(1 - 0.999842)/l(10)' | bc -l
```

    ## 38.01342913045577376801

-   PacBio provides a web page documenting any PacBio-specific extension
    to the standard SAM format (e.g. the `ip` and `pw` tags for
    kinetics):
    <https://pacbiofileformats.readthedocs.io/en/10.0/BAM.html>

## Anything else in a sequence BAM file?

-   Try `samtools view -H alz.ccs.toy.bam`
-   **Question**: Can you find out what “`-H`” means? (**Hint**: How to
    get help for a command line tool?)

``` bash
samtools view -H alz.ccs.toy.bam 
```

    ## @HD  VN:1.5  SO:unknown  pb:3.0.1
    ## @RG  ID:7ae86b2d PL:PACBIO   DS:READTYPE=CCS;BINDINGKIT=101-717-300;SEQUENCINGKIT=101-644-500;BASECALLERVERSION=5.0.0;FRAMERATEHZ=100.000000 PU:m64014_190506_005857 PM:SEQUELII CM:S/P3-C1/5.0-8M
    ## @PG  ID:ccs-4.2.0    PN:ccs  VN:4.2.0    DS:Generate circular consensus sequences (ccs) from subreads.   CL:ccs ccs 1perc.bam 1perc.ccs.bam --log-level INFO --min-rq 0.9
    ## @PG  ID:samtools PN:samtools PP:ccs-4.2.0    VN:1.14 CL:samtools view -s 0.1 -bh isoseq3/alz.ccs.bam
    ## @PG  ID:samtools.1   PN:samtools PP:samtools VN:1.14 CL:samtools view -H alz.ccs.toy.bam

-   The “`-H`” parameter outputs just the header of a BAM/SAM file. A
    header contains information such as the sequencing platform,
    chemistry version, sample names, and even commands used to generate
    the BAM/SAM file.
-   Some PacBio’s tools rely on the header to run. E.g. Modern version
    of CCS will not be able to run if the chemistry is too old (E.g. RS
    II).
-   Can you find out what platform this example dataset was sequenced on
    and what was the version of CCS used?

## Aligned BAM file

-   Let’s try doing your first reference genome alignment! Type the
    following command (It should take only around 2 mins):

``` bash
# The "\" here is to allow us to split the command
# into multiple lines to make the command looks "cleaner" 
pbmm2 align hg38.mmi alz.ccs.toy.bam alz.ccs.toy.aligned.bam \
  --preset ISOSEQ --sort -j 4 --log-level INFO \
  --log-file pbmm2_c4.log
```

-   **Question**: Can you look at the content of the first read and
    observe any difference?

``` bash
samtools view alz.ccs.toy.aligned.bam | 
  head -1 | 
  tr '\t' '\n' | 
  awk 'str_line="Column-"FNR":"{print str_line,$0}' OFS=$'\t' |
  cut -c -100
```

    ## Column-1:    m64014_190506_005857/130744625/ccs
    ## Column-2:    0
    ## Column-3:    chr1
    ## Column-4:    184918
    ## Column-5:    5
    ## Column-6:    56S34=1X19=1X61=1X8=1X19=1X31=1X15=1X8=1X4=1X5=1X42=1X45=1X129=140N46=1X23=757N48=1X104=65
    ## Column-7:    *
    ## Column-8:    0
    ## Column-9:    15612
    ## Column-10:   AAGCAGTGGTATCAACGCAGAGTACTTTTTTTTTTTTTTTTTTTTTTTTTTTTTTCGGTTTCTGCTCAGTTCTTTATTGATTGGTGTGC
    ## Column-11:   ~~~~~~~~~~~~~~~~~~~~~~~~~3~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ## Column-12:   RG:Z:7ae86b2d
    ## Column-13:   ec:f:38.3301
    ## Column-14:   np:i:37
    ## Column-15:   rq:f:0.999991
    ## Column-16:   sn:B:f,14.5022,20.4547,5.73951,9.6271
    ## Column-17:   zm:i:130744625
    ## Column-18:   mg:f:98.1636

-   **Question**: Which one is the “`CIGAR`” string and can you look for
    the definition of what the CIGAR string here means?
-   **Question**: Can you tell which position in the reference genome is
    this alignment?
-   One cool tool in `samtools` is that you can view the alignment on
    the command line (substitute “`chr1:185300`” with your alignment, if
    it’s different):

``` bash
samtools tview -d T alz.ccs.toy.aligned.bam -p chr1:185300 --reference hg38.fa.gz
```

    ##  185301    185311    185321    185331    185341    185351    185361             
    ## CAAAGGCTCCTCCGGGCCCCTCACCAGCCCCAGGTCCTTTCCCAGAGATGCCTGGAGGGAAAAGGCTGAGTGAGGGTGGT
    ## ..................................................KK...K...KKKK..K..K...K.......
    ## ..................................................>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

-   Press `?` in `samtools tview` to look for help. You can press `q` to
    exit the view. Use the arrow keys to move around the alignment.
-   The first row is the reference sequence, the second row is the
    “consensus sequence” from the alignments. From third row onwards
    these are the alignments mapping to this region.
-   **Tricks**: We’ve heard about awk being awesome, why do you want to
    know about it? Try:

``` bash
# What does this do? Swap 14 to 15, what do you need to change?
samtools view alz.ccs.toy.bam | awk '{gsub("np:i:","",$14); total+=$14;count++} END {print total/count}'
```

    ## 24.2683

## SMRT Link without GUI and `pbcromwell`

### Installing SMRT Link command line tools

-   In many HPC, you often don’t have the permission to run a service at
    the background for the graphical user interface (GUI) version of
    SMRT Link.
-   SMRT Link provides a “`--smrttools-only`” option during installation
    to install SMRT Link on just the command line. Execute the
    following:

``` bash
# Remove the installation if it already exists
rm -rf ~/softwares/SMRTLink-10.2
# Install SMRT Link, "--rootdir" specifies the installation directory
~/Downloads/smrtlink_10.2.0.133434.run --rootdir ~/softwares/SMRTLink-10.2 --smrttools-only
```

-   The installation will install SMRT Link command line tools at
    “`~/softwares/SMRTLink-10.2`”.
-   The tools from SMRT Link installation are meant to be stable for
    SMRT Link, so the version may be different from what’s available on
    Bioconda.
-   Most of the tools in SMRT Link will be located at
    “`~/softwares/SMRTLink-10.2/smrtcmds/bin`”. For example, you can run
    `pbmm2` by typing (Compare the version to that of the beginning
    using conda):

``` bash
# Go to base environment first to avoid package conflict
conda activate base
~/softwares/SMRTLink-10.2/smrtcmds/bin/pbmm2 --version
```

    ## pbmm2 1.8.0 (commit v1.7.0-9-g3c16a4d)

### Reproducible and easy pipeline with `pbcromwell`

-   Cromwell is a “workflow language”
    (<https://github.com/broadinstitute/cromwell>) that is used in SMRT
    Link to design and run bioinformatics pipelines. In bioinformatics,
    you often have many steps that you will run again and again. A
    pipeline puts together those steps and make it easy for you to rerun
    (thus reproducible) the steps easily.
-   `pbcromwell` is a tool that can be used to run SMRT Link pipeline on
    the command line without going through the GUI. With the
    “`--smrttools-only`” SMRT Link installation, you can run the full
    pipelines just like how you can with the GUI!
-   There are many other useful tools that do not require installing a
    GUI. For example, one of the most common requests are to generate
    plots and summary metrics similar to what you see in SMRT Link. This
    can be done via `runqc-reports` tool provided by SMRT Tools.
-   Let’s try this while I explain step by step:

``` bash
~/softwares/SMRTLink-10.2/smrtcmds/bin/pbindex alz.ccs.toy.bam

~/softwares/SMRTLink-10.2/smrtcmds/bin/dataset create --type ConsensusReadSet \
  alz.ccs.toy.consensusreadset.xml alz.ccs.toy.bam
  
~/softwares/SMRTLink-10.2/smrtcmds/bin/runqc-reports alz.ccs.toy.consensusreadset.xml
  
cp ~/workshop_data/isoseq3/isoseq_primers.fasta .

~/softwares/SMRTLink-10.2/smrtcmds/bin/dataset create --type BarcodeSet \
  isoseq_primers.barcodeset.xml isoseq_primers.fasta
  
~/softwares/SMRTLink-10.2/smrtcmds/bin/pbcromwell --help
# By default pbcromwell runs locally. If you have a HPC you can configure
# Cromwell to use the job scheduler on your HPC such as SGE/Slurm etc
~/softwares/SMRTLink-10.2/smrtcmds/bin/pbcromwell configure
~/softwares/SMRTLink-10.2/smrtcmds/bin/pbcromwell show-workflows
~/softwares/SMRTLink-10.2/smrtcmds/bin/pbcromwell show-workflow-details pb_isoseq3

~/softwares/SMRTLink-10.2/smrtcmds/bin/pbcromwell run pb_isoseq3 -e alz.ccs.toy.consensusreadset.xml \
  -e isoseq_primers.barcodeset.xml \
  --task-option filter_min_qv=10 \
  --config cromwell.conf --nproc 4
```

-   It should take a few mins for the analysis to finish. You should see
    a folder named `cromwell_out` ( can be changed using `--output-dir`
    parameter). Inside this folder, the `outputs` folder contains the
    results of the pipeline.
-   The `.json` files typically contains the important numbers from the
    pipeline. For example, you can look at the file
    `barcode_isoseq3.report.json` by using `less`. A fantastic tool to
    work with `.json` file is `jq` (already installed for you) and you
    can format the file into a table similar to what you see on the GUI:

``` bash
cat cromwell_out/outputs/barcode_isoseq3.report.json | 
  jq -r '.attributes | (["Type","Value"] | (., map(length*"-"))), (.[] | [.name, .value]) | @tsv' | 
  column -t -s$'\t'
```

    ## Type                                                         Value
    ## ----                                                         -----
    ## Reads                                                        42224
    ## Reads with 5' and 3' Primers                                 37809
    ## Non-Concatamer Reads with 5' and 3' Primers                  37627
    ## Non-Concatamer Reads with 5' and 3' Primers and Poly-A Tail  37542
    ## Mean Length of Full-Length Non-Concatamer Reads              2933
    ## Unique Primers                                               1
    ## Mean Reads per Primer                                        37809
    ## Max. Reads per Primer                                        37809
    ## Min. Reads per Primer                                        37809
    ## Reads without Primers                                        4415
    ## Percent Bases in Reads with Primers                          0.8972253037887876
    ## Percent Reads with Primers                                   0.8954386131110269

-   The `runqc-reports` tool will generate a summary statistics for CCS
    in a file called “`ccs.report.json`”. Again, you can use the awesome
    `jq` to look at the stats in table format:

``` bash
cat ccs.report.json | 
  jq -r '.attributes | (["Type","Value"] | (., map(length*"-"))), (.[] | [.name, .value]) | @tsv' | 
  column -t -s$'\t'
```

-   If the `jq` command is too hard to remember, PacBio’s json is not
    hard to read directly using the old trusty `less`.

-   In the `cromwell_out/cromwell-executions` folder, all the
    intermediate results and outputs are stored in separate folder for
    different stages of the pipelines. For example, if you want to look
    at `lima` step:

``` bash
# Cromwell stores each "job" using a unique ID everytime you run a new job. 
# The wildcard "*" here is the unique ID that will be different from others
ls -lhtr cromwell_out/cromwell-executions/pb_isoseq3/*/call-lima_isoseq/execution
```

    ## total 89M
    ## -rw-rw-r-- 1 kpin kpin 8.0K Jan 11 21:46 script
    ## -rw-rw-r-- 1 kpin kpin    0 Jan 11 21:46 stderr.background
    ## -rw-rw-r-- 1 kpin kpin  286 Jan 11 21:46 script.submit
    ## -rw-rw-r-- 1 kpin kpin  651 Jan 11 21:46 script.background
    ## -rw-rw-r-- 1 kpin kpin    6 Jan 11 21:46 stdout.background
    ## -rw-rw-r-- 1 kpin kpin    0 Jan 11 21:46 stdout
    ## -rw-rw-r-- 1 kpin kpin    0 Jan 11 21:46 stderr
    ## -rw-rw-r-- 1 kpin kpin   98 Jan 11 21:46 fl_transcripts.lima.guess.txt
    ## -rw-rw-r-- 1 kpin kpin 7.5M Jan 11 21:46 fl_transcripts.lima.report
    ## -rw-rw-r-- 1 kpin kpin 5.5M Jan 11 21:46 fl_transcripts.lima.clips
    ## -rw-rw-r-- 1 kpin kpin  76M Jan 11 21:46 fl_transcripts.5p--3p.bam
    ## -rw-rw-r-- 1 kpin kpin 433K Jan 11 21:46 fl_transcripts.5p--3p.bam.pbi
    ## -rw-rw-r-- 1 kpin kpin  909 Jan 11 21:46 fl_transcripts.lima.summary.txt
    ## -rw-rw-r-- 1 kpin kpin   88 Jan 11 21:46 fl_transcripts.lima.counts
    ## -rw-rw-r-- 1 kpin kpin  585 Jan 11 21:46 fl_transcripts.json
    ## -rw-rw-r-- 2 kpin kpin 3.4K Jan 11 21:46 fl_transcripts.5p--3p.consensusreadset.xml
    ## -rw-rw-r-- 1 kpin kpin    2 Jan 11 21:46 rc
    ## drwxrwxr-x 2 kpin kpin 4.0K Jan 11 21:46 glob-2fafc865f103dbf71884e943029cd0fb
    ## -rw-rw-r-- 1 kpin kpin   43 Jan 11 21:46 glob-2fafc865f103dbf71884e943029cd0fb.list

-   If you want to understand how the pipeline runs the individual step,
    there’s a `script` file in the `execution` folder and you can find
    the detailed command used to generate the data.
-   For example, if I want to see how SMRT Link pipeline runs lima:

``` bash
# The grep command looks for the lima command and print 8 lines after the match (-A8)
grep -A8 '^lima' \
  cromwell_out/cromwell-executions/pb_isoseq3/*/call-lima_isoseq/execution/script
```

    ## lima \
    ##   -j 32 \
    ##   --isoseq \
    ##   --peek-guess \
    ##   --ignore-biosamples \
    ##   --alarms alarms.json \
    ##   cromwell_out/cromwell-executions/pb_isoseq3/c1b548a2-58bb-4962-b61a-324cd5f2eee5/call-lima_isoseq/inputs/233360854/filtered.consensusreadset.xml \
    ##   cromwell_out/cromwell-executions/pb_isoseq3/c1b548a2-58bb-4962-b61a-324cd5f2eee5/call-lima_isoseq/inputs/-1912619708/isoseq_primers.barcodeset.xml \
    ##   fl_transcripts.json

-   Finally, one additional advantage of running the full pipeline is
    that SMRT Link generates some useful QC plots that you can find by:

``` bash
# The "-L" option tells "find" to search in symlinks, too
# "! -name "*thumb*" will remove all the files that has "thumb" in the filenames,
# Those are the thumbnails used in SMRT Link web interface but is not useful for us
# on the command line
find -L ./cromwell_out -name '*.png' ! -name '*thumb*'
```

-   Downside of all these? There’s no nice web pages to click and
    navigate on.
-   For even more advanced users, you can customize the pipeline by looking into the `WDL` workflow, which is out of the scope for our workshop.