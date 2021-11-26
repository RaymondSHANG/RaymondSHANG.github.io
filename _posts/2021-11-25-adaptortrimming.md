---
layout: post
title: "Adaptor.Trimming”
subtitle: "2021_update"
date: 2021-11-25 18:57:27
header-style: text
catalog: true
author: "Yuan"
tags: [fastp, Cutadapt, BBDuk, Trimmomatic]
---

> All tools work well enough

# RNAseq pipline update(2)
During the library preparation process, Illumina adapter sequences are annealed to sequencing reads.  The adapter sequences are required for attaching reads to flow cells and for attaching indexes to reads.  When sequencing is complete it’s important to remove or trim off, the adapter sequences from the reads.  The exact DNA sequence of the adapters depends on the library preparation kit that is used for sequencing.  The adapter sequence for Illumina Nextera XT kit is:
```
CTGTCTCTTATACACATCT
```
The adapter sequences for other kits may be different, so be sure to check which kit was used for library prep and get the appropriate sequence from the adapter sequences manual.

I found high content of Illumina Adaptor sequence in the fastq files from fastqc output. After searching carefully through google and google scholar, 4 widely used adaptor trimming programs were found, namely:
1. [fastp](https://github.com/OpenGene/fastp)
2. [Cutadapt](https://cutadapt.readthedocs.io/en/stable/) or [trim_galore](https://github.com/FelixKrueger/TrimGalore), which is fastq+cutadapt
3. [BBDuk](https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbduk-guide/) is part of [bbtools](https://jgi.doe.gov/data-and-tools/bbtools/)
4. [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)
   
## fastp
fastp is the latest among these 4. fastp is developed in C++ with multithreading supported to afford high performance. It is much faster than Cutadapt and Trimmomatic, according to their original paper published in 2018.
```bash
#For single end
fastp -i in.fq -o out.fq
#or for paired end
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```
If you don't want to process all the data, you can specify --reads_to_process to limit the reads to be processed. This is useful if you want to have a fast preview of the data quality, or you want to create a subset of the filtered data.

### Adapter trimming using fastp

Adapter trimming is enabled by default, but you can disable it by -A or --disable_adapter_trimming. Adapter sequences can be automatically detected for both PE/SE data.

1. For SE data, the adapters are evaluated by analyzing the tails of first ~1M reads. This evaluation may be inacurrate, and you can specify the adapter sequence by -a or --adapter_sequence option. If adapter sequence is specified, the auto detection for SE data will be disabled.
2. <b>For PE data, the adapters can be detected by per-read overlap analysis, which seeks for the overlap of each pair of reads</b>. This method is robust and fast, so normally you don't have to input the adapter sequence even you know it. But you can still specify the adapter sequences for read1 by --adapter_sequence, and for read2 by --adapter_sequence_r2. If fastp fails to find an overlap (i.e. due to low quality bases), it will use these sequences to trim adapters for read1 and read2 respectively.
3. For PE data, the adapter sequence auto-detection is disabled by default since the adapters can be trimmed by overlap analysis. However, you can specify --detect_adapter_for_pe to enable it.
4. For PE data, fastp will run a little slower if you specify the sequence adapters or enable adapter auto-detection, but usually result in a slightly cleaner output, since the overlap analysis may fail due to sequencing errors or adapter dimers.
5. Implementations
   The most widely used adapter is the Illumina TruSeq adapters. If your data is from the TruSeq library, you can add 
   ```
   --adapter_sequence=AGATCGGAAGAGCACACGTCTGAACTCCAGTCA --adapter_sequence_r2=AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT 
   ```
   to your command lines, or enable auto detection for PE data by specifing detect_adapter_for_pe.
6. fastp contains some built-in known adapter sequences for better auto-detection. If you want to make some adapters to be a part of the built-in adapters, please file an issue.
7. You can also specify --adapter_fasta to give a FASTA file to tell fastp to trim multiple adapters in this FASTA file. Here is a sample of such adapter FASTA file:
    ```
    >Illumina TruSeq Adapter Read 1
    AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
    >Illumina TruSeq Adapter Read 2
    AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT
    >polyA
    AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    ```
    The adapter sequence in this file should be at least 6bp long, otherwise it will be skipped. And you can give whatever you want to trim, rather than regular sequencing adapters (i.e. polyA).

8. fastp first trims the auto-detected adapter or the adapter sequences given by --adapter_sequence | --adapter_sequence_r2, then trims the adapters given by --adapter_fasta one by one.

9. The sequence distribution of trimmed adapters can be found at the HTML/JSON reports.

### Other Usage
```
usage: fastp -i <in1> -o <out1> [-I <in1> -O <out2>] [options...]
options:
  # I/O options
  -i, --in1                          read1 input file name (string)
  -o, --out1                         read1 output file name (string [=])
  -I, --in2                          read2 input file name (string [=])
  -O, --out2                           read2 output file name (string [=])
      --unpaired1                      for PE input, if read1 passed QC but read2 not, it will be written to unpaired1. Default is to discard it. (string [=])
      --unpaired2                      for PE input, if read2 passed QC but read1 not, it will be written to unpaired2. If --unpaired2 is same as --unpaired1 (default mode), both unpaired reads will be written to this same file. (string [=])
      --failed_out                     specify the file to store reads that cannot pass the filters. (string [=])
      --overlapped_out                 for each read pair, output the overlapped region if it has no any mismatched base. (string [=])
  -m, --merge                          for paired-end input, merge each pair of reads into a single read if they are overlapped. The merged reads will be written to the file given by --merged_out, the unmerged reads will be written to the files specified by --out1 and --out2. The merging mode is disabled by default.
      --merged_out                     in the merging mode, specify the file name to store merged output, or specify --stdout to stream the merged output (string [=])
      --include_unmerged               in the merging mode, write the unmerged or unpaired reads to the file specified by --merge. Disabled by default.
  -6, --phred64                      indicate the input is using phred64 scoring (it'll be converted to phred33, so the output will still be phred33)
  -z, --compression                  compression level for gzip output (1 ~ 9). 1 is fastest, 9 is smallest, default is 4. (int [=4])
      --stdin                          input from STDIN. If the STDIN is interleaved paired-end FASTQ, please also add --interleaved_in.
      --stdout                         output passing-filters reads to STDOUT. This option will result in interleaved FASTQ output for paired-end input. Disabled by default.
      --interleaved_in                 indicate that <in1> is an interleaved FASTQ which contains both read1 and read2. Disabled by default.
      --reads_to_process             specify how many reads/pairs to be processed. Default 0 means process all reads. (int [=0])
      --dont_overwrite               don't overwrite existing files. Overwritting is allowed by default.
      --fix_mgi_id                     the MGI FASTQ ID format is not compatible with many BAM operation tools, enable this option to fix it.
  
  # adapter trimming options
  -A, --disable_adapter_trimming     adapter trimming is enabled by default. If this option is specified, adapter trimming is disabled
  -a, --adapter_sequence               the adapter for read1. For SE data, if not specified, the adapter will be auto-detected. For PE data, this is used if R1/R2 are found not overlapped. (string [=auto])
      --adapter_sequence_r2            the adapter for read2 (PE data only). This is used if R1/R2 are found not overlapped. If not specified, it will be the same as <adapter_sequence> (string [=])
      --adapter_fasta                  specify a FASTA file to trim both read1 and read2 (if PE) by all the sequences in this FASTA file (string [=])
      --detect_adapter_for_pe          by default, the adapter sequence auto-detection is enabled for SE data only, turn on this option to enable it for PE data.
    
  # global trimming options
  -f, --trim_front1                    trimming how many bases in front for read1, default is 0 (int [=0])
  -t, --trim_tail1                     trimming how many bases in tail for read1, default is 0 (int [=0])
  -b, --max_len1                       if read1 is longer than max_len1, then trim read1 at its tail to make it as long as max_len1. Default 0 means no limitation (int [=0])
  -F, --trim_front2                    trimming how many bases in front for read2. If it's not specified, it will follow read1's settings (int [=0])
  -T, --trim_tail2                     trimming how many bases in tail for read2. If it's not specified, it will follow read1's settings (int [=0])
  -B, --max_len2                       if read2 is longer than max_len2, then trim read2 at its tail to make it as long as max_len2. Default 0 means no limitation. If it's not specified, it will follow read1's settings (int [=0])

  # duplication evaluation and deduplication
  -D, --dedup                          enable deduplication to drop the duplicated reads/pairs
      --dup_calc_accuracy              accuracy level to calculate duplication (1~6), higher level uses more memory (1G, 2G, 4G, 8G, 16G, 24G). Default 1 for no-dedup mode, and 3 for dedup mode. (int [=0])
      --dont_eval_duplication          don't evaluate duplication rate to save time and use less memory.

  # polyG tail trimming, useful for NextSeq/NovaSeq data
  -g, --trim_poly_g                  force polyG tail trimming, by default trimming is automatically enabled for Illumina NextSeq/NovaSeq data
      --poly_g_min_len                 the minimum length to detect polyG in the read tail. 10 by default. (int [=10])
  -G, --disable_trim_poly_g          disable polyG tail trimming, by default trimming is automatically enabled for Illumina NextSeq/NovaSeq data

  # polyX tail trimming
  -x, --trim_poly_x                    enable polyX trimming in 3' ends.
      --poly_x_min_len                 the minimum length to detect polyX in the read tail. 10 by default. (int [=10])
  
  # per read cutting by quality options
  -5, --cut_front                      move a sliding window from front (5') to tail, drop the bases in the window if its mean quality < threshold, stop otherwise.
  -3, --cut_tail                       move a sliding window from tail (3') to front, drop the bases in the window if its mean quality < threshold, stop otherwise.
  -r, --cut_right                      move a sliding window from front to tail, if meet one window with mean quality < threshold, drop the bases in the window and the right part, and then stop.
  -W, --cut_window_size                the window size option shared by cut_front, cut_tail or cut_sliding. Range: 1~1000, default: 4 (int [=4])
  -M, --cut_mean_quality               the mean quality requirement option shared by cut_front, cut_tail or cut_sliding. Range: 1~36 default: 20 (Q20) (int [=20])
      --cut_front_window_size          the window size option of cut_front, default to cut_window_size if not specified (int [=4])
      --cut_front_mean_quality         the mean quality requirement option for cut_front, default to cut_mean_quality if not specified (int [=20])
      --cut_tail_window_size           the window size option of cut_tail, default to cut_window_size if not specified (int [=4])
      --cut_tail_mean_quality          the mean quality requirement option for cut_tail, default to cut_mean_quality if not specified (int [=20])
      --cut_right_window_size          the window size option of cut_right, default to cut_window_size if not specified (int [=4])
      --cut_right_mean_quality         the mean quality requirement option for cut_right, default to cut_mean_quality if not specified (int [=20])
  
  # quality filtering options
  -Q, --disable_quality_filtering    quality filtering is enabled by default. If this option is specified, quality filtering is disabled
  -q, --qualified_quality_phred      the quality value that a base is qualified. Default 15 means phred quality >=Q15 is qualified. (int [=15])
  -u, --unqualified_percent_limit    how many percents of bases are allowed to be unqualified (0~100). Default 40 means 40% (int [=40])
  -n, --n_base_limit                 if one read's number of N base is >n_base_limit, then this read/pair is discarded. Default is 5 (int [=5])
  -e, --average_qual                 if one read's average quality score <avg_qual, then this read/pair is discarded. Default 0 means no requirement (int [=0])

  
  # length filtering options
  -L, --disable_length_filtering     length filtering is enabled by default. If this option is specified, length filtering is disabled
  -l, --length_required              reads shorter than length_required will be discarded, default is 15. (int [=15])
      --length_limit                 reads longer than length_limit will be discarded, default 0 means no limitation. (int [=0])

  # low complexity filtering
  -y, --low_complexity_filter          enable low complexity filter. The complexity is defined as the percentage of base that is different from its next base (base[i] != base[i+1]).
  -Y, --complexity_threshold           the threshold for low complexity filter (0~100). Default is 30, which means 30% complexity is required. (int [=30])

  # filter reads with unwanted indexes (to remove possible contamination)
      --filter_by_index1               specify a file contains a list of barcodes of index1 to be filtered out, one barcode per line (string [=])
      --filter_by_index2               specify a file contains a list of barcodes of index2 to be filtered out, one barcode per line (string [=])
      --filter_by_index_threshold      the allowed difference of index barcode for index filtering, default 0 means completely identical. (int [=0])

  # base correction by overlap analysis options
  -c, --correction                   enable base correction in overlapped regions (only for PE data), default is disabled
      --overlap_len_require            the minimum length to detect overlapped region of PE reads. This will affect overlap analysis based PE merge, adapter trimming and correction. 30 by default. (int [=30])
      --overlap_diff_limit             the maximum number of mismatched bases to detect overlapped region of PE reads. This will affect overlap analysis based PE merge, adapter trimming and correction. 5 by default. (int [=5])
      --overlap_diff_percent_limit     the maximum percentage of mismatched bases to detect overlapped region of PE reads. This will affect overlap analysis based PE merge, adapter trimming and correction. Default 20 means 20%. (int [=20])

  # UMI processing
  -U, --umi                          enable unique molecular identifier (UMI) preprocessing
      --umi_loc                      specify the location of UMI, can be (index1/index2/read1/read2/per_index/per_read, default is none (string [=])
      --umi_len                      if the UMI is in read1/read2, its length should be provided (int [=0])
      --umi_prefix                   if specified, an underline will be used to connect prefix and UMI (i.e. prefix=UMI, UMI=AATTCG, final=UMI_AATTCG). No prefix by default (string [=])
      --umi_skip                       if the UMI is in read1/read2, fastp can skip several bases following UMI, default is 0 (int [=0])

  # overrepresented sequence analysis
  -p, --overrepresentation_analysis    enable overrepresented sequence analysis.
  -P, --overrepresentation_sampling    One in (--overrepresentation_sampling) reads will be computed for overrepresentation analysis (1~10000), smaller is slower, default is 20. (int [=20])

  # reporting options
  -j, --json                         the json format report file name (string [=fastp.json])
  -h, --html                         the html format report file name (string [=fastp.html])
  -R, --report_title                 should be quoted with ' or ", default is "fastp report" (string [=fastp report])
  
  # threading options
  -w, --thread                       worker thread number, default is 3 (int [=3])
  
  # output splitting options
  -s, --split                        split output by limiting total split file number with this option (2~999), a sequential number prefix will be added to output name ( 0001.out.fq, 0002.out.fq...), disabled by default (int [=0])
  -S, --split_by_lines               split output by limiting lines of each file with this option(>=1000), a sequential number prefix will be added to output name ( 0001.out.fq, 0002.out.fq...), disabled by default (long [=0])
  -d, --split_prefix_digits          the digits for the sequential number padding (1~10), default is 4, so the filename will be padded as 0001.xxx, 0 to disable padding (int [=4])
  
  # help
  -?, --help                         print this message
```


## Cutadapt
 Cutadapt is written in Python 3, first published in 2011.

```bash
cutadapt -a AGATCGGAAGAGC -A AGATCGGAAGAGC -o out.1.fastq -p out.2.fastq RB6891_1_R1.fastq.gz RB6891_1_R2.fastq.gz
```
## BBDuk
"Duk" stands for Decontamination Using Kmers. BBDuk is basic on Java and is ultrafast, developped before 2014 and still actively maintained. It can do lots of different things:

### Adapter trimming:
```bash
bbduk.sh -Xmx8g in=reads.fq out=clean.fq ref=adapters.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo
#or 
bbduk.sh -Xmx8g in1=read1.fq in2=read2.fq out1=clean1.fq out2=clean2.fq ref=adapters.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo
```

(if your data is very low quality, you may wish to use more sensitive settings of hdist=2 k=21)

where "ktrim=r" is for right-trimming (3' adapters), "ktrim=l" is for left-trimming (5' adapters), and "ktrim=N" just masks the kmers with "N". 

"k" specifies the kmer size to use (must be at most the length of the adapters) while "mink" allows it to use shorter kmers at the ends of the read (for example, k=11 for the last 11 bases). 

"hdist" means "hamming distance"; this allows one mismatch. 

Instead of "ref=file" you can alternately (or additionally) say "literal=ACTGGT,TTTGGTG" for those two literal strings. The BBTools package currently includes Truseq and Nextera adapters sequences in /bbmap/resources/ as truseq.fa.gz, truseq_rna.fa.gz, and nextera.fa.gz. You can specify whether or not BBDuk looks for the reverse-complement of the reference sequences as well as the forward sequence with the flag "rcomp=t" or "rcomp=f"; by default it looks for both. You can also restrict kmer operations such as adapter-trimming to only read 1 or read 2 with the "skipr1" or "skipr2" flags, or restrict them to the leftmost or rightmost X bases of a read with the restrictleft or restrictright flags. For normal paired-end fragment libraries, I recommend adding the flags "tbo", which specifies to also trim adapters based on pair overlap detection (which does not require known adapter sequences), and "tpe", which specifies to trim both reads to the same length (in the event that an adapter kmer was only detected in one of them).

### Quality trimming:
```bash
bbduk.sh -Xmx1g in=reads.fq out=clean.fq qtrim=rl trimq=10
#or
bbduk.sh -Xmx1g in=read1.fq in=read2.fq out1=clean1.fq out2=clean2.fq qtrim=rl trimq=10
```

This will quality-trim to Q10 using the Phred algorithm, which is much more accurate(debatable!!) than naive trimming. "qtrim=rl" means it will trim the left and right sides; you can instead set "qtrim=l" or "qtrim=r".

### Contaminant filtering:
```bash
bbduk.sh -Xmx1g in=reads.fq out=unmatched.fq outm=matched.fq ref=phix.fa k=31 hdist=1 stats=stats.txt
#or
bbduk.sh -Xmx1g in1=r1.fq in2=r2.fq out1=unmatched1.fq out2=unmatched2.fq outm1=matched1.fq outm2=matched2.fq ref=phix.fa k=31 hdist=1 stats=stats.txt
```

This will remove all reads that have a 31-mer match to phix (a common Illumina spikein, which is included in /bbmap/resources/), again allowing one mismatch. The "outm" stream will catch reads that matched a reference kmers. This allows you to split a set of reads based on the presence of something. "stats" will produce a report of which contaminant sequences were seen, and how many reads had them.

### Kmer masking:
```bash
bbduk.sh -Xmx1g in=ecoli.fa out=ecoli_masked.fq ref=salmonella.fa k=25 ktrim=N rskip=0
```

This will mask all 25-mers in E.coli that are also shared by Salmonella, by converting them to N. You can change them to some other letter instead, like X.
### pairs interleaved in a single file
BBDuk can handle single-ended reads, pairs in two files, or pairs interleaved in a single file (which will be autodetected based on read names, or can be overridden with the 'int=t' or 'int=f' flag). For example:
```bash
bbduk.sh -Xmx1g in=reads.fq out1=clean1.fq out2=clean2.fq minlen=25 qtrim=r trimq=10
```

This will process the input as interleaved (which is forced since there are two output files). It will trim to Q10 on the right end only, and throw away reads shorter than 25bp after trimming. If one read is too short and the other read is OK, both will be thrown away, to preserve pair ordering. This can be changed with the "rieb" (removeIfEitherBad) flag; setting it to false will keep the reads unless both of them are too short.

BBDuk can process fasta, fastq, scarf, qual, and sam files, raw, gzipped, or bzipped. It can also handle references that are very large (even the human genome) in a reasonable amount of time and memory, if you increase the -Xmx parameter; it needs around 15 to 30 bytes per kmer. It can do operations such as quality-trimming and contaminant-filtering in the same pass, but not two different operations using kmers (such as left and right kmer trimming), although BBDuk2 can do that. BBDuk can also do various other filtering procedures such as complexity filtering, length filtering, gc-content filtering, average-quality filtering, chastity-filtering, and filtering by number of Ns.

### bbduk helpfile
For more details and features, you can run bbduk.sh with no parameters for the help menu, or ask a question in this thread.

Note! If your OS does not support shellscripts, you would replace
bbduk.sh -Xmx1g
with
java -ea -Xmx1g -cp C:\bbmap\current\ jgi.BBDukF
...but leave the rest of the command the same. "C:\bbmap\current" would vary depending on where you unzipped bbmap.

The "-Xmx1g" flag tells BBDuk to use 1GB of RAM. When using the shellscript, BBDuk does not strictly need that flag and can autodetect the amount of memory available. When using a large reference, or a large value of "hdist" or "edist" (hamming distance and edit distance, both of which greatly increase the amount of memory needed), you may need to set this higher.

P.S. When processing dual files, instead of "in1=r1.fq in2=r2.fq out1=clean1.fq out2=clean2.fq", you can simply use "in=r#.fq out=clean#.fq". The "#" symbol will be replaced by 1 and 2.

## Trimmomatic

```bash
java jar trimmomatic-0.39.jar PE -threads 4 read1.fastq.gz read2.fastq.gz read1_paired.fastq.gz read1_unpaired.fastq.gz read2_paired.fastq.gz read2_unpaired.fastq.gz ILLUMINACLIP:adapters.fasta:2:30:10
```

---
