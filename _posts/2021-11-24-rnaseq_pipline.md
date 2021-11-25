---
layout: post
title: "”RNASeq_Pipline”"
subtitle: "”2021_update”"
date: 2021-11-24 21:37:06
header-style: text
catalog: true
author: "Yuan"
tags: [fastqc, multiqc, salmon]
---

> RNASeq: from fastq to transcript counts

> VS code is the best text editor!

# Updated RNASeq pipline (1)
I update my pipline for analyzing RNASeq data recently, using salmon 1.5.2.
This blog include fastq file quality control using fastqc, viewing multiple fastqc results using multiqc, getting salmon index using transcriptome and genome, and salmon indexing.

## Quality Control

Fastqc could be donwloaded as jar file where we could find the fastqc excute file in the subfolder:  
FastQC.app/Contents/MacOS/fastqc
We could add a link to this fastqc in the system path so fastqc could be called from terminal or bash scripts.
```bash
source ~/.bash_profile
for i in {1..5}
do
	sample="sample$i"
    subfolder="projectName"
    sample2="/Volumes/MyRNASeqData//$subfolder/$sample"
    echo ${sample2}
	fastqc -t 8 -o fastqc ${sample2}_R1.fastq.gz ${sample2}_R2.fastq.gz
done

#cd fastqc
#multiqc .
```

Multiqc could be installed through pip, and running multiqc is simple: just "cd" to the fastqc output folder, and run "multiqc ."
```bash
pip install multiqc    # Install
cd fastqc-output
multiqc .              # Run
```
fastqc would give you an idea how the sequencing data looks like, including the read quality, high frequence sequences, GC content, potential adaptor contaminations, etc. If we could high content of adaptor sequences, we might need to trime the fastq files before indexing.

## Get Salmon index
According to the latest Salmon manual and Rob(Author of Salmon,https://www.biostars.org/p/456231/), the lastest Salmon generally recmend decoy-aware transcritome alignment.
The decoy sequences are regions of the target genome that are sequence similar to annotated transcripts. These are the regions of the genome most likely to cause mismapping (e.g. transcribed pseudogenes, etc.). There are 3 ways to run salmon : (a) with just the annotated transcriptome being indexed (b) with the annotated transcriptome and a small set of decoys computed using MASHMAP to search transcripts against the genome and (c) with the annotated transcriptome and using the entire genome as decoy sequence.

The (a) method requires the fewest resources, (b) requires a good deal of resources to run the MASHMAP step, but the resulting index is similar to that of (a) and it avoids the most obvious cases of misalignment. (c) results in the largest index, but it's the most effective at avoiding potentially spurious mappings.

So, I choose option (c) here. 
1. Download genome and transriptome files (Mouse as an example, For rat, using the name toplevel.fa.gz[which is the same to primary_assembly])
   
   Genome file: [Mus_musculus.GRCm39.dna.primary_assembly.fa.gz](http://ftp.ensembl.org/pub/release-104/fasta/mus_musculus/dna/Mus_musculus.GRCm39.dna.primary_assembly.fa.gz)

   cDNA: [Mus_musculus.GRCm39.cdna.all.fa.gz](http://ftp.ensembl.org/pub/release-104/fasta/mus_musculus/cdna/Mus_musculus.GRCm39.cdna.all.fa.gz)

   ncRNA: [Mus_musculus.GRCm39.ncrna.fa.gz](http://ftp.ensembl.org/pub/release-104/fasta/mus_musculus/ncrna/Mus_musculus.GRCm39.ncrna.fa.gz)
   
2. Generate decoys.txt
   ```bash
    grep "^>" <(gunzip -c Mus_musculus.GRCm39.dna.primary_assembly.fa.gz) | cut -d " " -f 1 > decoys.txt
    sed -i.bak -e 's/>//g' decoys.txt
    ```
3. Generate 
    ```bash
    cat Mus_musculus.GRCm39.cdna.all.fa.gz Mus_musculus.GRCm39.ncrna.fa.gz Mus_musculus.GRCm39.dna.primary_assembly.fa.gz > Mus_musculus.GRCm39.gentrome.fa.gz
    ```
4. Indexing
    ```
    salmon index -t Mus_musculus.GRCm39.gentrome.fa.gz -i mouse_salmon_index_104 --decoys decoys.txt -k 31
    ```

## Run Salmon Index

Run Salmon index with validateMappings
```bash
source ~/.bash_profile
for i in {1..154}
do
	sample="sample_$i"
	subfolder="sample001_051"
	if [ "$i" -gt 51 ]; then
    	subfolder="sample052_103"
	fi
	if [ "$i" -gt 103 ]; then
		subfolder="sample104_154"
	fi
	sample2="/Volumes/MyRNASeqData/ProjectName/data/$subfolder/$sample"
	echo ${sample2}
	salmon quant -i /directorytocDNA/release104/mouse/mouse_salmon_index_104 \
				 -l A \
				 -1 ${sample2}_R1.fastq.gz -2 ${sample2}_R2.fastq.gz \
				 -o salmonResult_V15/${sample}.quant \
				 --validateMappings \
				 --seqBias \
				 --useVBOpt \
				 --numBootstraps 50
done
```

---
