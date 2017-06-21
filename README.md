# Quality control for MinION sequencing data

## What?

An R script to do some basic QC on data from Oxford Nanopore's MinION sequencer, using the `sequencing_summary.txt` file from Albacore as input.

## Why?

Because other tools focus on getting data out of the fastq or fast5 files, which is slow and arduous. The benefit of this script is that it works on a single, small, .txt summary file. So it's a lot quicker than most other things out there: it takes about a minute to analyse a 4GB flowcell on my laptop. 

## Quick start

```
Rscript minion_QC.R sequencing_summary.txt output_directory
```

* *minion_QC.R*: path to this script
* *sequencing_summary.txt*: path to a sequencing_summary.txt file from Albacore
* *output_directory*: path to an output directory. Files will be overwritten.

## Dependencies
A recent version of R, and install the following:

```
install.packages("ggplot2")
install.packages("viridis")
install.packages("reshape2")
install.packages("plyr")
```

## Output details
More details on rationale are in [this blog post](robertlanfear.com/blog/files/nanopore_performance.html), example output is in the `/example_output` folder of this repository.

### summary.yaml

Simple text summary of the data in yaml format (opens in any text editor, but can also be read by any coding language). Ultralong reads are defined as the maximum possible fraction of the data that has an N50>100KB, following the definition given in [this preprint](biorxiv.org/content/early/2017/04/20/128835). Here's the first part of the example file:

```yaml
input file: /Users/roblanfear/Desktop/sequencing_summary.txt
All reads:
  total.gigabases: 3.8433736
  N50.length: 34354.0
  mean.length: 16150.1
  median.length: 7176.0
  max.length: 826249.0
  mean.q: 9.5
  median.q: 10.1
  reads:
    '>20kb': 83614
    '>50kb': 14651
    '>100kb': 253
    '>200kb': 10
    '>500kb': 1
    '>1m': 0
    ultralong: 574
  gigabases:
    '>20kb': 3.113988
    '>50kb': 0.9333141
    '>100kb': 0.0303313
    '>200kb': 0.0030453
    '>500kb': 0.0008262
    '>1m': 0.0e+00
    ultralong: 0.0606444
```

### length_histogram.png
Read length, on a log10 scale, on the X axis, and counts on the Y axis.
![length_histogram](example_output/length_histogram.png)

### q_histogram.png
Mean Q score for a read on the X axis, and counts on the Y axis. 
![q_histogram](example_output/q_histogram.png)

### epb_histogram.png
Events per base (i.e. numbe of events for each read, divided by the number of bases called for that read) on the X axis, and counts on the Y axis. We have found this measure to be potentially useful in finding dodgy reads, see robertlanfear.com/blog/files/nanopore_performance.html for more.
![epb_histogram](example_output/epb_histogram.png)

### length_vs_q.png
Read length (log10 scale) on the X axis, mean Q score on the Y axis. Points are coloured by the events per base, scaled such that all events per base >10 are recorded as a 10. The latter makes it easier to distinguish 'good' and 'bad' reads. 
![length_vs_q](example_output/length_vs_q.png)

### yield_summary.png
Minimum read length on the X axis, and the yield of bases with reads at least that long on the Y axis. This is just like the 'reads' table in the `summary.txt` output, but done across all read lengths up to 100KB. I cut off at 100KB because you (probably) don't have most of your data at those lenghts. Good on you if you do though.
![yield_summary](example_output/yield_summary.png)

### channel_summary.png
Histograms of total bases, total reads, mean read length, and median read length that show the variance across the 512 available channels. Repeated for all data and reads with Q>10.
![channel_summary](example_output/channel_summary.png)

### flowcell_channels_epb.png
This one's busy, but hopefully useful. The 512 channels are laid out as on the R9.5 flowcell. Then each sub-panel of the plot simply plots out the time of the run in hours on the X axis, and the events per base (log scale, cut off at 10 events per base) on the Y axis. This gives a little insight into exactly what was going on in each of your channels over the course of the run. Blow it up big!
![flowcell_channels_epb](example_output/flowcell_channels_epb.png)
