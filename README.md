
# exolink -- GATK best-practices pipeline adapted to linked-reads [WIP]

This pipeline basecalls/demultiplexes and aligns [linked-reads from 10x Genomics](https://www.10xgenomics.com/linked-reads/) and calls variants with GAKT. Specifically, germline short variant discovery (SNPs and indels) is performed according to [GATK best-practices](https://software.broadinstitute.org/gatk/best-practices/workflow?id=11145).

The pipeline is written in [Nextflow](https://www.nextflow.io/) and contains seven sub-pipelines:

* `demux.nf`: basecall/demultiplex raw BCL data with [bcl2fastq](https://emea.support.illumina.com/content/dam/illumina-support/documents/documentation/software_documentation/bcl2fastq/bcl2fastq2-v2-20-software-guide-15051736-03.pdf)
* `ema_align.nf`: align reads with [EMA](https://github.com/arshajii/ema/)
* `lr_align.nf`: align reads with [longranger ALIGN](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/advanced/other-pipelines)
* `bwa_align.nf`: align reads with [BWA](http://bio-bwa.sourceforge.net/)
* `single_sample.nf `: recalibrate BAM and call variants with GATK
* `multi_sample.nf`: joint genotyping and variant annotation and fitering.
* `phase.nf`: phase variants with [HapCUT2](https://github.com/vibansal/HapCUT2)

## Workflow

* Basecall and demultiplex raw sequencing data with `demux.nf`
* Process all your samples individualy with first `ema_align.nf` and `single_sample.nf`
* Process your samples jointly with `multi_sample.nf`

## Basecalling/demultiplexing

This Nextflow pipeline basecalls and demultiplexes linked-reads from 10x Genomics. To run this pipeline, the 8-base sample indexes are needed, corresponding to the 10x Genomics indexes (e.g. `SI-GA-A1`).

This pipeline makes some assumptions about the input data. For example, it makes the assumption that it is paired-end sequencing, and therefore uses `--use-bases-mask=Y*,I*,Y*` in `bcl2fastq`, and assumes that the read lengths (and index length) is found in `RunInfo.xml`.

### Running on tiny-bcl

Here's how to run this pipeline on the "tiny-bcl" example dataset from 10x Genomics. First of all, download the tiny-bcl tar file and the Illumina Experiment Manager sample sheet: tiny-bcl-samplesheet-2.1.0.csv.

> Download the tiny-bcl data:
> https://support.10xgenomics.com/genome-exome/software/pipelines/latest/using/mkfastq#example_data

Next, edit the `[Data]` part of the samplesheet from the following:

```
Lane,Sample_ID,index,Sample_Project
5,Sample1,SI-GA-C5,tiny_bcl
```

To the following:

```
Lane,Sample_ID,index
5,Sample1,CGACTTGA
5,Sample1,TACAGACT
5,Sample1,ATTGCGTG
5,Sample1,GCGTACAC
```

Using that the index `SI-GA-C5` corresponds to the four octamers `CGACTTGA,TACAGACT,ATTGCGTG,GCGTACAC`. Notice that we also removed the `Sample_Project` column.

Provided that you've set installed all the software in `environment.yml` (or maybe used this pipeline's Docker container at https://hub.docker.com/r/olavurmortensen/demuxlink), you should be able to run the pipeline like this:

```
nextflow demuxlink/main.nf --rundir tiny-bcl-2.2.0 --outdir results --samplesheet tiny-bcl-samplesheet-2.1.0.csv
```

And get the FASTQ files in `results/fastq_out/Sample1/outs` and the FastQC reports in `results/fastqc/Sample1`.

