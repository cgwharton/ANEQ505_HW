
Briefly **describe** the key information from each denoising output file:
1. Representative Sequences
	 The sequences file contains the unique denoised sequences (ASVs) after error correction and filtering. These are later used for visualization and further analysis. 
		 
2. Denoising Stats
	 The stats file provides us with the initial number of reads per sample and then summarizes how many reads were kept or removed at each step of denoising. This is useful for checking data quality and troubleshooting read loss. 
		 
3. Denoised Table
	The table file provides summary statistics on ASVs in the dataset- total feature frequency, along with frequency per sample and frequency per feature. You can also look at sampling depth and see how many samples would be included/excluded at a certain depth. 
		

**Answer the following questions:**  
1. What is the mean reads per sample?
	Pre-denoising: Mean reads per sample was 15,163.394558.
	Post-denoising: 11,115.7
	
	==*demux was just for quality check*==

2. How long are the reads?
	 251
	 
	 ==*Found in seqs*==
		 ==*250 - 427*==

3. What is the maximum length of all your sequences?
	 The maximum length was 427.
	 
	 *CORRECT*
	 *note: the 427 was an error*

4. Which sample (not including extraction controls starting with EC) lost the highest % of reads?
	The sample that lost the most reads was 2019.3.14.cow.oral.20, it only had 8.76% of the original input. 
	
	hhh


5. Why did you choose to trim or truncate where you did?
	I chose to trim off the 251st base and keep 0-250 because the reverse read median quality score for 251 was 13, falling under the benchmark of 30.

**To submit your homework from this document:**
write all of your commands here, then use command+P (for mac) or control+P (for windows) and search Git: commit. click it. then search for Git: Push and click it. go to your github online to check that it pushed correctly. we will check your github for homework credit. 

## Catie's Command Log ##

```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```
 
 Launch a session and load qiime
```
ainteractive --ntasks=6 --time=02:00:00
module purge
module load qiime2/2024.10_amplicon
```

Import sequence/reads
```
qiime tools import \
--type EMPPairedEndSequences \
--input-path raw_reads \
--output-path cow_reads.qza
```

Demultiplex code in the demux.sh file to submit a job

```
#!/bin/bash
#SBATCH --job-name=demux
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=cwharton@colostate.edu

module purge
module load qiime2/2024.10_amplicon

#change the following line if your file path looks different
cd /scratch/alpine/$USER/cow/demux

#Below is the command you will run to demultiplex the samples.

qiime demux emp-paired \
--m-barcodes-file ../metadata/cow_barcodes.txt \
--m-barcodes-column barcode \
--p-rev-comp-mapping-barcodes \
--p-rev-comp-barcodes \
--i-seqs ../cow_reads.qza \
--o-per-sample-sequences demux_cow.qza \
--o-error-correction-details cow_demux_error.qza

#visualize the read quality
qiime demux summarize \
--i-data demux_cow.qza \
--o-visualization demux_cow.qzv
```

Submitting the Job
 ```
 cd slurm
 sbatch demux.sh
 ```

Denoising

```
cd ../dada2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_cow.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 6 \
--o-representative-sequences cow_seqs_dada2.qza \
--o-denoising-stats cow_dada2_stats.qza \
--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file cow_dada2_stats.qza \
--o-visualization cow_dada2_stats.qzv

qiime feature-table summarize \
--i-table cow_table_dada2.qza \
--m-sample-metadata-file ../metadata/cow_metadata.txt \
--o-visualization cow_table.qzv

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2.qza \
--o-visualization cow_seqs.qzv
```
