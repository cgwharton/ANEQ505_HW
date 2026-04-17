**Load data
```
cp -r /pl/active/courses/2025_summer/CSU_2025/raw_reads_oxycow .
```

 **Launch a session and load qiime**
```
ainteractive --ntasks=6 --time=02:00:00
module purge
module load qiime2/2024.10_amplicon
```

**Create directories for the different analyses:**
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

**Load metadata**
```
cd metadata

qiime metadata tabulate \--m-input-file metadata.txt \--o-visualization metadata.qzv
```

**Import sequence/reads**
```
qiime tools import \
--type EMPPairedEndSequences \
--input-path raw_reads_oxycow \
--output-path oxycow_reads.qza
```

**Demultiplex code in the demux.sh file to submit a job**
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
cd /scratch/alpine/$USER/project/demux

#Below is the command you will run to demultiplex the samples.

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

**Submitting the Job**
 ```
 cd slurm
 sbatch oxy.sh
 ```

**Denoising**
```
cd ../dada2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_oxycow.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 6 \
--o-representative-sequences oxy_seqs_dada2.qza \
--o-denoising-stats oxy_dada2_stats.qza \
--o-table oxy_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file oxy_dada2_stats.qza \
--o-visualization oxy_dada2_stats.qzv

qiime feature-table summarize \
--i-table oxy_table_dada2.qza \
--m-sample-metadata-file ../metadata/metadata.txt \
--o-visualization cow_table.qzv

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2.qza \
--o-visualization cow_seqs.qzv
```
