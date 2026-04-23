## Launch a session and load qiime
 - do this every time you open terminal
 - qiime2 analysis was done with qiime2 amplicon version 2024.10
```
ainteractive --ntasks=6 --time=02:00:00

module purge
module load qiime2/2024.10_amplicon

cd project
```
### Load data
- do this once
```
cp -r /pl/active/courses/2025_summer/CSU_2025/raw_reads_oxycow .
```
### Create directories for the different analyses:
- do this once
```
mkdir slurm

mkdir taxonomy

mkdir tree

mkdir taxaplots

mkdir dada2

mkdir demux

mkdir core_metrics

mkdir metadata
```
### Load metadata
```
cd metadata

qiime metadata tabulate \--m-input-file metadata.txt \--o-visualization metadata.qzv
```
### Import sequence/reads
```
qiime tools import \
--type EMPPairedEndSequences \
--input-path raw_reads_oxycow \
--output-path oxycow_reads.qza
```
## Demultiplex 
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

cd /scratch/alpine/$USER/project/demux

qiime demux emp-paired \
--m-barcodes-file ../metadata/oxy_barcodes.txt \
--m-barcodes-column Barcode \
--p-rev-comp-mapping-barcodes \
--p-rev-comp-barcodes \
--i-seqs ../oxycow_reads.qza \
--o-per-sample-sequences demux_oxycow.qza \
--o-error-correction-details cow_demux_error.qza

#visualize the read quality

qiime demux summarize \
--i-data demux_oxycow.qza \
--o-visualization demux_oxycow.qzv
```

### Submitting the Job
 ```
 cd slurm
 sbatch oxy.sh
 ```
- 4/17 demux failed
	- figured out it was a problem with sample ID containing "/" 
		- ex: 1180_04/24/2025_12:00 PM

### Troubleshooting
4/23/26 

**Sample IDs in oxy_barcode.txt contain "/" and QIIME doesnt like it**

```
# checking to see if the SampleIDs contain "/"

head oxy_barcodes.txt

# code to make a cleaned copy

awk 'BEGIN{FS=OFS="\t"} 
NR==1 {print; next} 
{
  gsub(/\//, "_", $1)
  gsub(/:/, "_", $1)
  gsub(/ /, "", $1)
  print
}' oxy_barcodes.txt > oxy_barcodes_clean.txt

# check the copy

cut -f1 oxy_barcodes_clean.txt | head
cut -f1 oxy_barcodes_clean.txt | grep '/'
cut -f1 oxy_barcodes_clean.txt | grep ':'
cut -f1 oxy_barcodes_clean.txt | grep ' '

# Those last three should return nothing.
```

### Demultiplex code try #2
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

cd /scratch/alpine/$USER/project/demux

qiime demux emp-paired \
--m-barcodes-file ../metadata/oxy_barcodes_clean.txt \
--m-barcodes-column Barcode \
--p-rev-comp-mapping-barcodes \
--p-rev-comp-barcodes \
--i-seqs ../oxycow_reads.qza \
--o-per-sample-sequences demux_oxycow.qza \
--o-error-correction-details cow_demux_error.qza

#visualize the read quality

qiime demux summarize \
--i-data demux_oxycow.qza \
--o-visualization demux_oxycow.qzv
```

#### Submitting the Job
 ```
 cd ../slurm
 sbatch oxy.sh
 ```
##### Output email: 
Job ID: 26005166  
Cluster: alpine  
User/Group: cwharton@colostate.edu/cwhartonpgrp@colostate.edu  
State: COMPLETED (exit code 0)  
Nodes: 1  
Cores per node: 12  
  
-------- CPU Metrics --------  
CPU Utilized: 00:43:21  
CPU Efficiency: 6.08% of 11:53:00 core-walltime  
Job Wall-clock time: 00:59:25  
Memory Utilized: 3.73 GiB  
Memory Efficiency: 8.28% of 45.00 GiB (3.75 GiB/core)


### Open demux.qzv and check quality score
	Forward reads median quality score never dropped below 30
	Reverse reads median quality score for 251 was 13
		Trim at 0 bp
		Truncate at 251 bp
		

## Denoising
```
cd ../dada2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_oxycow.qza \
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
--m-sample-metadata-file ../metadata/metadata.txt \
--o-visualization cow_table.qzv

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2.qza \
--o-visualization cow_seqs.qzv
```
- Plugin error from feature-table
	- DADA2 table now has cleaned sample IDs like:
		- 1010_04_24_2025_12_00AM
	- but metadata file still has the old/unclean IDs, like:
		- 1010_04/24/2025_12:00 AM
	- QIIME requires the sample IDs in the feature table and metadata file to match **exactly**.

### Troubleshooting
 
```
cd ../metadata

awk 'BEGIN{FS=OFS="\t"}
NR==1 {print; next}
{
  split($1,a,"_")
  $1 = a[1]"_"a[2]"_"a[3]"_2025_"a[4]"_00"a[5]
  print
}' metadata.txt > metadata_fixed.txt

head metadata_fixed.txt
```

- barcode.txt has 1010_04_24_2025_12_00AM  
- metadata.txt has 1010_04_24_2025_12AM_00

```
sed -E 's/([0-9]+_[0-9]+_[0-9]+_2025_)([0-9]+)(AM|PM)_00/\1\2_00\3/' metadata_fixed.txt > metadata_fixed2.txt

cut -f1 metadata_fixed.txt | head -20
```

Then I had to go in and edit the metadata_fixed2.text file to 
### Try feature table again
```
cd ../dada2

qiime feature-table summarize \
  --i-table cow_table_dada2.qza \
  --m-sample-metadata-file ../metadata/metadata_fixed2.txt \
  --o-visualization cow_table.qzv
```
## Classify taxonomy using GreenGenes2

### First get the Greengenes2 database:
```
cd /scratch/alpine/$USER/project/taxonomy

wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```

### Classify taxonomy using GreenGenes2 classify the ASVs 
```
qiime feature-classifier classify-sklearn \
--i-reads ../dada2/cow_seqs_dada2.qza \
--i-classifier 2024.09.backbone.v4.nb.qza \
--o-classification taxonomy_gg2.qza
```

### Visualize the taxonomy of your ASVs: 
```
qiime metadata tabulate \
--m-input-file taxonomy_gg2.qza \
--o-visualization taxonomy_gg2.qzv
```

### Filter out mitochondria/chloroplasts/sp004296775
```
qiime taxa filter-table \
--i-table ../dada2/cow_table_dada2.qza \
--i-taxonomy taxonomy_gg2.qza \
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
--o-filtered-table ../dada2/table_nomitochloro_gg2.qza
```

### Visualize the taxa bar plot
```
qiime taxa barplot \
--i-table ../dada2/table_nomitochloro_gg2.qza \
--i-taxonomy taxonomy_gg2.qza \
--m-metadata-file ../metadata/metadata.txt \
--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2.qzv
```

## Phylogenetic tree 

Create a job script to run the phylogenetic tree building. Remember you must start a new terminal session, navigate to your slurm directory, and then submit the job. You do NOT need to start any other interactive sessions.This job will take about an hour. 

Go to OnDemand and create a new text file for your job script
```
nano <tree.sh>
```

```
#!/bin/bash
#SBATCH --job-name=tree
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=20:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=cwharton@colostate.edu
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal

#Activate qiime

module purge
module load qiime2/2024.10_amplicon


#Get reference

wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza


#Command

qiime fragment-insertion sepp \
--i-representative-sequences ../dada2/cow_seqs_dada2_filtered300.qza \
--i-reference-database ../tree/2022.10.backbone.sepp-reference.qza \
--o-tree ../tree/tree_gg2.qza \
--o-placements ../tree/tree_placements_gg2.qza
```

- submit the job from the terminal
```
#submit the job
sbatch tree.sh
