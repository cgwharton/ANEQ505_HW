# Week 4 Tutorial: Denoising & Intro to homework [Mac Commands]

**Last week on tutorial Fridays:**

- Obsidian tip and tricks
- Imported sequences
- Reviewed and practiced demultiplex sequences, understanding the demux.qzv file

**This week:**

- Review Quiz 1
- Review and practice denoising sequences
- Submitting jobs
- Discuss homework data and formatting

**Note:** I have added a terminology page to the Modules section, so you now have a list of all the jargon we use.

---

**Let's get started! (Submit denoising before going over the quiz)**

**Alpine On-Demand:** [ondemand-rmacc.rc.colorado.edu](http://ondemand-rmacc.rc.colorado.edu/ "(opens in a new window)")

Load the class node:

sinteractive --reservation=aneq505 --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

Load your Qiime environment:

module purge  
  
module load qiime2/2024.10_amplicon

Let's navigate to the correct place. Use `cd` to move into your **decomp****_tutorial dada2** folder. Use `pwd` to make sure you're in the correct place. 

`cd /scratch/alpine/$USER/decomp_tutorial/dada2`  

QUIZZZZZ

---

### **Denoising sequences to ASVs using DADA2**

![Screenshot 2026-02-12 at 2.51.48 PM.png](https://colostate.instructure.com/courses/220471/files/39016092/preview)

Now, we will denoise using [DADA2Links to an external site.](https://www.nature.com/articles/nmeth.3869 "(opens in a new window)") to generate our amplicon sequence variants. These 'ASVs' represent the 'features' in the resulting feature and representative sequences table.

Recall that denoising does three important things:

- **Quality filtering:** trims off low-quality reads at a designated location on the sequence.
- **Chimera checking:** Chimeric reads are generally considered artifacts in sequencing applications (i.e. amplicon sequencing) and need to filtered out from the data during processing. Chimeras occur when two DNA sequences join together. This is a contaminate and if they are not removed, can lead to the user falsely interpreting the sequence as a novel organism. 
- **Paired- end read joining:** Here, we have paired-end data, but each read is sequenced on its own (leading to the production of a "forward" and "reverse" read). We need to join these reads together, so that the overlapping region between them can also be used for correcting sequencing errors and potentially yield sequences of higher quality for better taxonomic annotation.  Before joining, trimming low quality bases is necessary for optimizing taxonomy annotation and sequence clustering.  
    

![Screen Shot 2022-02-10 at 12.26.20 PM-2.png](https://colostate.instructure.com/courses/220471/files/38493038/preview)

Use DADA2 to denoise our sequences. Because we have two sequencing runs, we have to do this twice.

#this will take about 22 mins total  
  
cd ../dada2  
  
#denoise for run 2  
qiime dada2 denoise-paired \  
--i-demultiplexed-seqs ../demux/demux_run2.qza \  
--p-trunc-len-f 150 \  
--p-trunc-len-r 150 \  
--p-n-threads 8 \  
--o-table table_run2.qza \  
--o-representative-sequences seqs_run2.qza \  
--o-denoising-stats dada2_stats_run2.qza  
  
  
#denoise for run 3  
qiime dada2 denoise-paired \  
--i-demultiplexed-seqs ../demux/demux_run3.qza \  
--p-trunc-len-f 150 \  
--p-trunc-len-r 150 \  
--p-n-threads 8 \  
--o-table table_run3.qza \  
--o-representative-sequences seqs_run3.qza \  
--o-denoising-stats dada2_stats_run3.qza

You might notice the --p-trunc-len line, and may be wondering what the 150 means. If you remember when we looked at **demux_seqs.qzv** in QIIME2 View, the interactive quality plot showed that these data were pretty good quality. So, we decided to keep the whole length (150 base pairs) of all reads. In practice, if you start to see the quality dip below 30, you may want to decide to trim some of those low quality base calls off, that way you can be confident in any later analyses. If you were to see quality dip at the beginning of the sequence (usually due to sequencing chemistry), or if you need to trim off your barcodes, you can use --p-trim-left to do this. 

**Important notes about denoising:**

1. Here, we have 2 sequencing runs, this happened because we had so many samples that it needed to be spread out over multiple runs. in this case its important to randomize across runs to account for batch affects. so in order to get all samples we need for the subset we chose for the tutorial data, we needed to uses runs "two and three". thats why we have to do some steps twice. after we denoise, we can then merge the sequences and table into singular files for the rest of the analysis.

2. when you merge two runs together, you MUST use the same trim and truncate parameters or they wont merge properly.

3. if you use different primers than the ones we discuss in the class (reminder we use the Earth Microbiome Protocol 515f/806r primers which also act as the illumina sequencing primers), then you will need to trim them out using cutadadpt.

Why is #3 important? lets review the amplicon itself: 

![image.png](https://colostate.instructure.com/courses/220471/files/38958093/preview)

- since we are teaching on the EMP primers which specifically also act as sequencing primers, we get more reads dedicated to our amplicon (in green) and also we dont need to trim out primers.
- additionally, remember that we have paired end reads (so we have a forward and a reverse read) that need to be joined back together. Denoising is doing that for us!

Let's get a visualization of each of these stats outputs for each run

qiime metadata tabulate \  
 --m-input-file dada2_stats_run2.qza \  
 --o-visualization dada2_stats_run2.qzv  
  
qiime metadata tabulate \  
 --m-input-file dada2_stats_run3.qza \  
 --o-visualization dada2_stats_run3.qzv

 **DADA2 denoising creates three output files:**

- stats.qza (denoising statistics)
- table.qza (ASV feature table)
- seqs.qza (sequences associated with ASVs)

 You will have a set of files for each denoising command (e.g., one set for run 2 and one set for run 3, so six files total).

**1. DADA2 STATS FILE:** 

Each row is a sample. 

Columns: 

- **Input**: number of reads per sample (same number as in the demux.qzv)
- **Filtered**: number of reads that passed dada2 filtering
- **% of input passed filter**: % of reads that passed dada2 filtering
- **Denoised**: number of reads kept after denoising
- **Merged** (only when using paired end data: number of reads merged)
- **Non-chimeric**: number of reads that are non-chimeric (recall that a chimera is a PCR artifact that is created when two different reads are combined together).
- **% of input non-chimeric:** % of reads that are non-chimeric 

On this visualization, you are checking to see if there are any samples where a large portion of the reads are removed. Having a few samples like this is ok, but if all of your samples have a large portion of reads removed, then there may be an issue with your data. 

**2. DADA2 TABLE FILE:** 

**Merge denoised tables - required when you have samples from multiple sequencing runs like we have here. Then visualize the merged table so all samples are in one place.**

qiime feature-table merge \  
--i-tables table_run2.qza \  
--i-tables table_run3.qza \  
--o-merged-table table.qza  
  
qiime feature-table summarize \  
--i-table table.qza \  
--o-visualization table.qzv \  
--m-sample-metadata-file ../metadata/metadata.txt

 The table file contains the following: 

**Features = ASVs**

**Overview tab:** this tab gives you an overview of the summary statistics. 

- **Table summary**: # of samples, number of total features (ASVs) in the dataset, and total feature frequency (how many times were the features observed across all samples?). 
- **Frequency per sample**: stats on reads per sample. Ex: on average, the frequency of features observed in a sample ~4,000. 

**Frequency per feature:** stats on how much a feature/ASV shows up. Ex: on average a feature is observed 683 times.

**3. DADA2 SEQS FILE:** 

qiime feature-table merge-seqs \  
--i-data seqs_run2.qza \  
--i-data seqs_run3.qza \  
--o-merged-data seqs.qza  
  
qiime feature-table tabulate-seqs \  
--i-data seqs.qza \  
--o-visualization seqs.qzv

This file contains the ASV Feature ID and its corresponding sequence

In your seqs.qzv file, _what happens when you click on one of the blue sequences?_

---

**Running jobs on Alpine - you will use this for demultiplexing the samples for your homework assignment** 

- Some analysis can take a while (especially with large datasets), so we'll submit it as a "**job**".
- So far we’ve been working on the **interactive node**, for jobs we’ll be using the **computing node**
- **Batch jobs:** are resource provisions that run applications on compute nodes and do not require supervision or interaction.
- **Job script:** is a set of Linux commands paired with the resource requirements for your batch job 

**Create a test job and run it**

1. Go to OnDemand and create a new file in your **slurm directory**, call it "test.sh"

2. click edit to open and edit the file. 

This can be pasted into your example script. Note, to submit jobs, the .sh file must ALWAYS have the #!/bin/bash line as the first line. the rest are job parameters and are also required, but the order does not matter. 

#!/bin/bash  
#SBATCH --job-name=test  
#SBATCH --nodes=1  
#SBATCH --ntasks=2  
#SBATCH --partition=amilan  
#SBATCH --time=01:00:00  
#SBATCH --mail-type=ALL  
#SBATCH --mail-user=YOUR_USERNAME@colostate.edu  
#SBATCH --output=slurm-%j.out  
#SBATCH --qos=normal  
  
#Activate qiime  
  
module purge  
module load qiime2/2024.10_amplicon  
  
# command goes here  
OUTFILE="message_${SLURM_JOB_ID}.txt"  
echo "status report" > $OUTFILE  
echo "Job ID: $SLURM_JOB_ID" >> $OUTFILE  
echo "Node: $(hostname)" >> $OUTFILE  
echo "Timestamp: $(date)" >> $OUTFILE  
echo "You ran your first job!" >> $OUTFILE

Click Save, and exit the file.

To submit the job to the compute cluster, use:

sbatch test.sh

You will then get a job ID as an output. It might be good to note this job ID in case you need to kill the job or check its status. We can also use the On-Demand portal to look at our job status. You should receive an email as well with your job status!

If you get a "failed" email, you can check your slurm directory for the slurm output file (looks like slurm-JOB-ID.out to see what went wrong. This is how you troubleshoot failed code from a job. 

---

**Summary**

Let's review what we've done so far!

1. There are many ways to study microbial ecology, one of which is to profile the microbial community composition.
2. We use **16S** (a small section of taxonomically informative target DNA, here we use 16S rRNA gene) to **study microbial composition and diversity**.

**Why this gene?**

1. It’s **ubiquitous**.
2. Contains regions that identical across diverse organisms (**conserved regions**), and regions that are variable across organisms (**variable regions**).
3. We target for a **specific region of the 16S gene**, PCR amplify, DNA sequence, and then use **qiime2 for sequence data processing**. 

 **Today:**

4. We used **Qiime2** to begin **processing sequence data.** 
5. Qiime2 **requires some type of input data** (today we input demultiplexed reads, your homework will input raw data)
    
    - **Decomposition Tutorial:** 
    
    1. We began with pre-demultiplexed paired end data (2x150nt)
    2. We denoised each run separately
    3. Merged quality controlled data together
    

---

#### **Cow Dataset for Homework:**  

Now its time to practice, we will use a new dataset, called the "cow dataset" to practice the commands you have learned: 

- **20 adult dairy cattle** were sampled (via swabbing) to determine the microbial community composition between **5 different body sites**.  
- Samples were sequenced with 2x250 bp chemistry on an Illumina miseq
- You will be given the raw sequencing file, the barcodes file, and the metadata file.
- We want you to determine **if and how the microbial composition changes between sites**
- These data are **paired end fastq files** that you will copy from a public folder on Alpine. 
- For homework 1, you will practice **importing** raw data, **demultiplexing paired ends reads**, and **denoising**.
- You will gain experience with editing code and importing paired end reads, and demultiplexing and denoising those samples.
- What you will submit: 
    - you will push your homework (via an obsidian note) to GitHub with the **commands** used to import, demultiplex, and denoise the samples.
    - you will include the **answers to the questions** in the note as well 

**To start your homework, go to the modules in Canvas, and download the homework_1.md file. this is the template you will follow to submit your homework (due Feb 19th at midnight).** 

**![Screenshot 2026-02-05 at 11.21.37 AM.png](https://colostate.instructure.com/courses/220471/files/38883454/preview)**

[](https://colostate.instructure.com/courses/220471/modules/items/7708206)

[](https://colostate.instructure.com/courses/220471/modules/items/7721665)