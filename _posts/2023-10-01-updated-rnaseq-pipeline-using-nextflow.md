---
layout: post
title: "updated RNAseq pipeline using nextflow"
subtitle: "This is a subtitle"
date: 2023-10-01 15:08:26
header-style: text
catalog: true
author: "Yuan"
tags: [RNASeq,NextFlow,nf]
---
{% include linksref.html %}

## 🚀 Overview

As RNA-Seq continues to be a core technology for transcriptomic analysis, having a reproducible, scalable, and modular pipeline is essential. In this post, I’ll walk through recent updates to our RNA-Seq pipeline built with [Nextflow](https://www.nextflow.io/), including improvements in preprocessing, quantification, and downstream QC.

---

## 🔧 Why Nextflow?

[Nextflow](https://www.nextflow.io/) enables scalable and reproducible scientific workflows using software containers. It integrates smoothly with cloud and HPC environments and is particularly well-suited for complex bioinformatics pipelines.

Key advantages:
- Easy parallelization
- Support for Docker/Singularity
- Seamless integration with GitHub and CI/CD tools
- Built-in traceability and logging

## 🗂 Directory Structure

```bash
rna-seq-pipeline/
├── main.nf
├── nextflow.config
├── modules/
│   ├── qc/
│   ├── align/
│   ├── quant/
├── results/
│   └── multiqc/
└── resources/
    ├── genome.fa
    └── annotation.gtf
```

## Example running bash code


<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a class="noCrossRef" href="#about" data-toggle="tab">About</a></li>
    <li><a class="noCrossRef" href="#bash" data-toggle="tab">Bash pipeline</a></li>
    <li><a class="noCrossRef" href="#nextflow" data-toggle="tab">Next Flow Main</a></li>
</ul>

<div class="tab-content">
<div role="tabpanel" class="tab-pane active" id="about" markdown="1">
## Directory Structure

```bash
rna-seq-pipeline/
├── main.nf
├── nextflow.config
├── modules/
│   ├── qc/
│   ├── align/
│   ├── quant/
├── results/
│   └── multiqc/
└── resources/
    ├── genome.fa
    └── annotation.gtf
```

</div>


<div role="tabpanel" class="tab-pane" id="bash" markdown="1">


```bash
nextflow run main.nf \
  --reads "data/*_R{1,2}.fastq.gz" \
  --genome GRCh38 \
  --aligner salmon \
  -profile docker
```
</div>

<div role="tabpanel" class="tab-pane" id="nextflow" markdown="1">

```groovy
#!/usr/bin/env nextflow

nextflow.enable.dsl=2

// Define parameters
params.reads = "data/*_R{1,2}.fastq.gz"
params.genome = "resources/genome.fa"
params.gtf = "resources/annotation.gtf"
params.aligner = "salmon" // or 'star'
params.outdir = "results"

workflow {

    reads_ch = Channel.fromFilePairs(params.reads, flat: true)

    qc_out = qc(reads_ch)
    trimmed_reads = trim(qc_out)

    if (params.aligner == 'salmon') {
        quant_out = quant_salmon(trimmed_reads)
    } else if (params.aligner == 'star') {
        aligned = align_star(trimmed_reads)
        quant_out = count_featureCounts(aligned)
    }

    multiqc_report(quant_out)
}

// QC Step
process qc {
    tag "$sample_id"
    input:
    set sample_id, file(reads) from reads_ch
    output:
    set sample_id, file("*_fastqc.zip") into qc_ch

    script:
    """
    fastqc -o ./ ${reads.join(' ')}
    """
}

// Trimming Step
process trim {
    tag "$sample_id"
    input:
    set sample_id, file(reads) from qc_ch
    output:
    set sample_id, file("*.fq.gz") into trimmed_ch

    script:
    """
    fastp -i ${reads[0]} -I ${reads[1]} -o ${sample_id}_R1_trimmed.fq.gz -O ${sample_id}_R2_trimmed.fq.gz
    """
}

// Salmon Quantification
process quant_salmon {
    tag "$sample_id"
    input:
    set sample_id, file(reads) from trimmed_ch
    output:
    set sample_id, file("quant.sf") into quant_out_ch

    script:
    """
    salmon quant -i salmon_index -l A \
        -1 ${reads[0]} -2 ${reads[1]} \
        -p 8 -o ${sample_id}_quant
    cp ${sample_id}_quant/quant.sf ./
    """
}

// STAR Alignment
process align_star {
    tag "$sample_id"
    input:
    set sample_id, file(reads) from trimmed_ch
    output:
    set sample_id, file("Aligned.out.sam") into aligned_ch

    script:
    """
    STAR --genomeDir star_index \
         --readFilesIn ${reads[0]} ${reads[1]} \
         --runThreadN 8 \
         --outFileNamePrefix ${sample_id}_ \
         --outSAMtype SAM
    cp ${sample_id}_Aligned.out.sam Aligned.out.sam
    """
}

// featureCounts (optional downstream after STAR)
process count_featureCounts {
    input:
    set sample_id, file(aligned) from aligned_ch
    output:
    set sample_id, file("counts.txt") into quant_out_ch

    script:
    """
    featureCounts -a ${params.gtf} -o counts.txt -T 4 ${aligned}
    """
}

// MultiQC summary
process multiqc_report {
    input:
    set sample_id, file(qcfiles) from quant_out_ch.collect()
    output:
    file("multiqc_report.html") into final_report

    script:
    """
    multiqc . -o ${params.outdir}/multiqc
    """
}

```
</div>

</div>



---

## 🧬 Pipeline Components

Here’s an overview of the major steps in the updated pipeline:

1. **Quality Control (FastQC + MultiQC)**  
   Initial QC of raw FASTQ files to assess base quality, adapter content, and GC bias.

2. **Trimming (TrimGalore or fastp)**  
   Adapter removal and quality trimming for cleaner downstream alignment or quantification.

3. **Alignment or Pseudoalignment**  
   - *Option A*: STAR for genome alignment  
   - *Option B*: Salmon for transcript-level quantification (faster, alignment-free)

4. **Gene-Level Quantification**  
   - Use of `tximport` for summarizing transcript-level counts (from Salmon) to gene level.

5. **QC Metrics Aggregation**  
   - Mapping rate summaries, read distribution, duplication rates, etc., compiled using MultiQC.

6. **Differential Expression (optional module)**  
   - DESeq2 or edgeR integrated as a modular postprocessing step.

---

## 📦 Tool Updates

- Replaced deprecated tools and updated versions to latest stable releases
- Added support for `fastp` as a trimming alternative
- Modular design now supports toggling between STAR and Salmon
- Improved containerization: Docker and Singularity support with `nf-core` compatibility

---


---
