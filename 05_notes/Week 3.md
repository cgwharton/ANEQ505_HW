# Week 3 Tutorial: Demultiplexing, Denoising, & Intro to homework [Mac Commands]

**Last week on tutorial Fridays:**

- Logged onto an interactive node
- Loaded Qiime2
- Downloaded decomp tutorial study metadata and visualized it

**This week:**

- Obsidian tip and tricks
- Import sequences
- Review and practice demultiplex sequences
- Review and practice denoising sequences
- Submitting jobs
- Discuss homework data and formatting

---

**Obsidian tips and tricks**

Example Obsidian note here [Example_Obsidian_Note.md](https://colostate.instructure.com/courses/220471/files/38895443?wrap=1 "Example_Obsidian_Note.md (opens in a new window)")[Download Example_Obsidian_Note.md](https://colostate.instructure.com/courses/220471/files/38895443/download?download_frd=1)[Open this document with ReadSpeaker docReader](https://docreader.readspeaker.com/docreader/?cid=11403&lang=en_us&url=https%3A%2F%2Finst-fs-iad-prod.inscloudgate.net%2Ffiles%2F0611686e-2ff8-45b8-82de-37242f637ba5%2FExample_Obsidian_Note.md%3Ftoken%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE3NzAzNzExNjEsInVzZXJfaWQiOm51bGwsInJlc291cmNlIjoiL2ZpbGVzLzA2MTE2ODZlLTJmZjgtNDViOC04MmRlLTM3MjQyZjYzN2JhNS9FeGFtcGxlX09ic2lkaWFuX05vdGUubWQiLCJob3N0IjpudWxsLCJqdGkiOiIyYTFkZmFhNC00MzkyLTRiNzQtYjNmNC1iNjEyZTUzZDliZjQiLCJleHAiOjE3NzA0NTc1NjF9.1Dn_4j7DAAXZDiy1xdwD2-XnDV8gClppdCq0ThuMlpB5n2nntRptYEcDkxq0bH-g8u1orsU_pgHqVe69LaOv4g "Open this document with ReadSpeaker docReader (opens in a new window)")

Example Obsidian workflow template here [example_worflow.md](https://colostate.instructure.com/courses/220471/files/38895442?wrap=1 "example_worflow.md (opens in a new window)")[Download example_worflow.md](https://colostate.instructure.com/courses/220471/files/38895442/download?download_frd=1)[Open this document with ReadSpeaker docReader](https://docreader.readspeaker.com/docreader/?cid=11403&lang=en_us&url=https%3A%2F%2Finst-fs-iad-prod.inscloudgate.net%2Ffiles%2F2bee0549-4385-4c7a-982c-c9c69c6ce1e3%2Fexample_worflow.md%3Ftoken%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE3NzAzODM1MTQsInVzZXJfaWQiOm51bGwsInJlc291cmNlIjoiL2ZpbGVzLzJiZWUwNTQ5LTQzODUtNGM3YS05ODJjLWM5YzY5YzZjZTFlMy9leGFtcGxlX3dvcmZsb3cubWQiLCJob3N0IjpudWxsLCJqdGkiOiJmNjgwMGIwMy01MWVjLTQwZmUtYTI4Mi00MDYxNzQzMGI2MjYiLCJleHAiOjE3NzA0Njk5MTR9.LCJS9cCQwgnDc3-y88H31cq69PVPmaGgKhzwxoIOb28a9OKGLnaHo9rIFhtb5r6MxMqTbZW9tIcN0al62-Jezw "Open this document with ReadSpeaker docReader (opens in a new window)")

**Obsidian cheat sheet:**

- To start a code chunk type ``` This will automatically put in the other set  ``` to signal the end of the code chunk
- To create a link to a webpage: [text for the link]([https://example.com/)Links to an external site.](https://example.com/\) "(opens in a new window)") 
- If you don't want the website link attached to a word, you can just copy and paste the link into your note
- To link to a page in your vault: [[page_you_want_to_link]]

**Mark down cheat sheet:**

|Element|Markdown Syntax|
|---|---|
|[HeadingLinks to an external site.](https://www.markdownguide.org/basic-syntax/#headings "(opens in a new window)")|`# H1   ## H2   ### H3`|
|[BoldLinks to an external site.](https://www.markdownguide.org/basic-syntax/#bold "(opens in a new window)")|`**bold text**`|
|[ItalicLinks to an external site.](https://www.markdownguide.org/basic-syntax/#italic "(opens in a new window)")|`*italicized text*`|
|[Ordered ListLinks to an external site.](https://www.markdownguide.org/basic-syntax/#ordered-lists "(opens in a new window)")|`1. First item   2. Second item   3. Third item`|
|[Unordered ListLinks to an external site.](https://www.markdownguide.org/basic-syntax/#unordered-lists "(opens in a new window)")|`- First item   - Second item   - Third item`|

More Obsidian syntax can be found [hereLinks to an external site.](https://help.obsidian.md/syntax "(opens in a new window)"). 

---

**Importing Data**

Before we can analyze sequencing data in QIIME2, we first need to import it. This step tells QIIME2 where your data lives and how they are formatted.

There are two common ways to import data into QIIME 2, you will get practice with both:

1. **Using a manifest file** – This method uses a text file that lists each sample and the location of its corresponding sequencing files. This is the approach you will use for the decomposition dataset because the sequences have already been demultiplexed.
    
2. **Using compressed raw sequencing files** – This method is typically used when you receive raw data directly from a sequencing facility. This is the approach we will use for the homework.
    

---

**Demultiplexing**

Let's review demultiplexing, which we talked about in lecture. Remember that after extracting the DNA, we can use a barcoded PCR technique to assign a unique 12 base-pair barcode to all of the sequences within a single sample. After pooling and sequencing everything in one sequencing run (multiplexing; makes sequencing cheaper!), we can bioinformatically demultiplex to separate the sequences based on the sample they came from using the 12 base pair (bp) barcode. 

**_Simply put, demultiplexing means we take the sequences and assign to them to the appropriate samples_**

How does this work? When we incorporate the barcode during PCR, we know exactly what 12 bp barcode corresponds to each sample. We can record this in our metadata (YAY more metadata!). From the sequencer, we will get a **sequences.fastq** file (if it also has a .gz extension, that just means it is a zipped file), and a **barcodes.fastq** file.

**Multiplexed data has all the sequences from all the samples in one file (the sequences.fastq file)!** 

- **barcodes.fastq.gz will have the barcodes for each sample**
- **your metadata file will contain the sample names and barcodes**
- **These 3 files together will enable you to demultiplex the sequences.** 
- **your output will be a fastq file per sample**

**Input:** 

**![Screenshot 2023-02-02 at 3.26.17 PM.png](https://colostate.instructure.com/courses/220471/files/38493065/preview)**

**output:**![Screen Shot 2022-02-10 at 12.26.20 PM-2.png](https://colostate.instructure.com/courses/220471/files/38493067/preview)

When we tell QIIME2 to demultiplex, we will give it our sequences.fastq and barcodes.fastq files, and it will use these files along with the barcode that we give it in our metadata file to separate the sequences by sample name. This is the whole workflow from the bench (adding the barcodes to each sample) so the output files we get from the sequencer:

![Screen Shot 2022-01-27 at 12.43.18 PM.png](https://colostate.instructure.com/courses/220471/files/38493053/preview)

Recall that the output sequences.fastq file is a specific file type that comes from the sequencer that contains 4 line types, where each set of 4 lines corresponds to one sequencing read. The first line is a sequencer identifier. This line you will likely never use, but it has some information about the sequencing run.

![Screen Shot 2022-01-27 at 1.02.51 PM.png](https://colostate.instructure.com/courses/220471/files/38493051/preview)

The second line is either the sequence (150 or 250 bp long depending on your chemistry for an Illumina Miseq run), or it is your barcode. 

![Screen Shot 2022-01-27 at 1.12.15 PM.png](https://colostate.instructure.com/courses/220471/files/38493056/preview)

The "+" in the third line is just a placeholder. The purpose of this is to signal the end of the sequence and the start of the next line. 

The fourth and final line looks like a bunch of gibberish, but it has information about the quality of your sequence. This line will always be the same length as the sequence (or barcode), as each character corresponds to the quality of a single base pair.

![Screen Shot 2022-01-27 at 1.15.48 PM.png](https://colostate.instructure.com/courses/220471/files/38493057/preview)

The quality scores can then be converted into **phred quality scores** using an ASCII coding system. A phred score is a single number that corresponds to a single base pair within a sequence, and that number tells us the probability that the base at that position is correct. See the below chart on the left for phred scores and what they mean for how accurate a base might be.

![Screen Shot 2022-02-10 at 12.26.20 PM-2.png](https://colostate.instructure.com/courses/220471/files/38493068/preview)

**What Phred Score should you aim to have?** 

While there is no "correct" phred score, it's good to shoot for one of **at least 30**. It is okay if you need to keep some bases with a phred score of lower than this, but it is always dependent on the dataset itself. This is something we will explore later in our tutorials.  

**Why review this?** 

For the decomp dataset we will skip demultiplexing and instead import by a manifest file, but for the homework you will practice demultiplexing from raw data files. For both datasets you will practice reviewing the demultiplexed data for its quality which will inform your denoising choices. 

**Importing Sequences Using Manifest File for the Decomp Tutorial**

The decomp dataset sequences have already been demultiplexed (some sequencing facilities will demultiplex for you, some wont so its important to know how to import for both ways), so we will import them using the **fastq manifest format**. This is when we have a single fastq file for each sample, and when we import, we need to give the path for each fastq sample file in a .tsv or .txt file. This tells QIIME2 where to look for the sequences for each sample and it will combine them into one .qza file that we can use for downstream analyses. We will import the sequences as `SampleData[PairedEndSequencesWithQuality]`, because we have paired end data that contains Phred scores in the sequences file. 

![Screen Shot 2022-01-27 at 1.34.57 PM.png](https://colostate.instructure.com/courses/220471/files/38493050/preview)

**Let's get started!**

**Alpine On-Demand:** [ondemand-rmacc.rc.colorado.edu](http://ondemand-rmacc.rc.colorado.edu/ "(opens in a new window)")

Load the class node:

sinteractive --reservation=aneq505 --time=01:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

Load your Qiime environment:

module purge  
  
module load qiime2/2024.10_amplicon

Let's navigate to the correct place. Use `cd` to move into your **decomp****_tutorial** folder. Use `pwd` to make sure you're in the correct place. 

`cd /scratch/alpine/$USER/decomp_tutorial`  
  
pwd

**Create directories for the different analyses so you can keep your files organized:**

`mkdir slurm`

`mkdir taxonomy`

`mkdir tree`

`mkdir taxaplots`

`mkdir dada2`

`mkdir demux`

`mkdir core_metrics`

**Create a directory for the manifest and import the manifest files (we made them for you):** 

In your study you would need to construct the manifest file yourself, 

mkdir manifest  
cd manifest  

`cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/manifest_run2.txt .`

`cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/manifest_run3.txt .`

Move out of the manifest directory:

`cd ../`

**Copy the demultiplex reads into your analysis directory:**

`cp -r /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/reads_run2 .`

`cp -r /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/reads_run3 .`

Let's take a look at the data we just imported, we can do this by moving into the reads folder, and typing ls

`cd reads_run2`

`ls`

these are the individual sample fastq files!

`cd ../`

**Import data into QIIME2:**

# run the import command to generate a qiime2 readable artifact for the reads, save the demux results in the demux folder.  
  
#import run 2  
qiime tools import \  
--type "SampleData[PairedEndSequencesWithQuality]" \  
--input-format PairedEndFastqManifestPhred33V2 \  
--input-path manifest/manifest_run2.txt \  
--output-path demux/demux_run2.qza  
  
#import run 3  
qiime tools import \  
--type "SampleData[PairedEndSequencesWithQuality]" \  
--input-format PairedEndFastqManifestPhred33V2 \  
--input-path manifest/manifest_run3.txt \  
--output-path demux/demux_run3.qza  

**Check Data Quality:** 

The first important step of data analysis is to check the data quality. Now that we've gotten our data into QIIME2 it's simple to get a visualization that we can use to evaluate our sample quality.  

# generate a demux visualization file  
  
cd demux  
  
#run2  
qiime demux summarize \  
--i-data demux_run2.qza \  
--o-visualization demux_run2.qzv  
  
#run3  
qiime demux summarize \  
--i-data demux_run3.qza \  
--o-visualization demux_run3.qzv  

Just like above, transfer this output file to your local computer and visualize it in [view.qiime2.orgLinks to an external site.](http://view.qiime2.org/ "(opens in a new window)"). 

**Questions to Answer with this Output:**

- What is the per-sample sequencing count?
- Which sample had the fewest sequences per sample? The most?
- How many samples were in this run?
- How does the quality look?
- Does your data look the exact same as your neighbors?
- How long are the reads? 
- **Where would you trim and/or truncate these sequences to maintain the highest quality data?**

**Binned Quality Scores**

Some sequencing platforms (e.g. newer Illumina instruments like the MiSeq i100) do not report **individual Phred quality scores** for each base. Instead, they use **binned quality scores**, where ranges of quality values are grouped and reported as a single representative score. This reduces file size and improves data compression, but it changes how quality information is encoded in your FASTQ files.

Rather than seeing the full range of Phred scores (e.g., 0–41), bases are assigned to one of **four quality bins**, which are reported as:

**2** → very low quality bases

**9** → low quality bases

**23** → moderate quality bases

**38** → high quality bases

Each of these values represents a _range_ of underlying Phred scores rather than an exact estimate of sequencing accuracy. For example, a base reported with a quality score of 38 may represent any base whose true quality falls within the highest bin.

![Screenshot 2026-02-04 at 12.54.14 PM.png](https://colostate.instructure.com/courses/220471/files/38870007/preview) 

This means that your demultiplexed sequences from your demux.qzv might look like this: VT ADD DEMUX HERE

Run the denoising commands and then take a Break  

---

### **Denoising sequences to ASVs using DADA2**

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
6. - **Decomposition Tutorial:** 
    
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

**To start your homework, go to the modules in Canvas, and download the homework_1.md file. this is the template you will follow to submit your homework (due Feb 12 at midnight).**