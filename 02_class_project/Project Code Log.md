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
### Create directories:
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

<mark style="background: #FFF3A3A6;">Then I had to go in and edit the metadata_fixed2.txt file to change the controls from "EC7_1_SR69_2025__00" to "EC7_1_SR69" so that it matched the oxy_barcodes.txt</mark>

### Try feature table again
```
cd ../dada2

qiime feature-table summarize \
  --i-table cow_table_dada2.qza \
  --m-sample-metadata-file ../metadata/metadata_fixed2.txt \
  --o-visualization cow_table.qzv
```
FINALLY, it worked!

Mean reads per sample
	Pre-denoising: 33,354.3
	Post-denoising: 20,495.1

How long are the reads: 251 - 415 bp 
- Must filter
## Remove long (300+ base pair) amplicons from the representative sequences file and the feature table
- 4/27/26
```
# filter out any large amplicons from the seqs and table (because they are contaminates)

cd dada2

qiime feature-table filter-seqs \
--i-data cow_seqs_dada2.qza \
--m-metadata-file cow_seqs_dada2.qza \
--p-where 'length(sequence) < 300' \
--o-filtered-data oxy_seqs_dada2_filtered300.qza

qiime feature-table tabulate-seqs \
--i-data oxy_seqs_dada2_filtered300.qza \
--o-visualization oxy_seqs_dada2_filtered300.qzv

qiime feature-table filter-features \
--i-table cow_table_dada2.qza \
--m-metadata-file oxy_seqs_dada2_filtered300.qza \
--o-filtered-table oxy_table_dada2_filtered300.qza
  
qiime feature-table summarize \
--i-table oxy_table_dada2_filtered300.qza \
--m-sample-metadata-file ../metadata/metadata_fixed2.txt \
--o-visualization oxy_table_dada2_filtered300.qzv

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
--i-reads ../dada2/oxy_seqs_dada2_filtered300.qza \
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
--i-table ../dada2/oxy_table_dada2_filtered300.qza \
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
--m-metadata-file ../metadata/metadata_fixed2.txt \
--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2.qzv
```

## Phylogenetic tree 

in tree.sh
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

module purge
module load qiime2/2024.10_amplicon


wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza


qiime fragment-insertion sepp \
--i-representative-sequences ../dada2/oxy_seqs_dada2_filtered300.qza \
--i-reference-database ../tree/2022.10.backbone.sepp-reference.qza \
--o-tree ../tree/tree_gg2.qza \
--o-placements ../tree/tree_placements_gg2.qza
```

New session
```
cd slurm
sbatch tree.sh
```
## Alpha Rarefaction Plot 
4/28/26
Filter out controls
```
qiime feature-table filter-samples \
--i-table dada2/table_nomitochloro_gg2.qza \
--m-metadata-file metadata/metadata_fixed2.txt \
--p-where "NOT [Treatment] IN ('ext_control') " \
--o-filtered-table dada2/table_nomitochloro_nocontrol.qza
```

```
qiime diversity alpha-rarefaction \
--i-table dada2/table_nomitochloro_nocontrol.qza \
--m-metadata-file metadata/metadata_fixed2.txt \
--o-visualization alpha_rarefaction_curve.qzv \
--p-min-depth 10 \
--p-max-depth 30000
```
- Alpha rarefaction curve
	- Seems to plateau around 15000 but I want a closer look
	- 15,000 fine for the class project but go down to 10,000 for publication

```
qiime diversity alpha-rarefaction \
--i-table dada2/table_nomitochloro_nocontrol.qza \
--m-metadata-file metadata/metadata_v3.txt \
--o-visualization alpha_rarefaction_curve_10000.qzv \
--p-min-depth 10 \
--p-max-depth 10000
```
## Run Core Metrics 

```
qiime diversity core-metrics-phylogenetic \
--i-table dada2/table_nomitochloro_nocontrol.qza \
--i-phylogeny tree/tree_gg2.qza \
--m-metadata-file metadata/metadata_v3.txt \
--p-sampling-depth 10000 \
--output-dir core_metrics_results_10000
```

## Visualize alpha diversity plots

**Observed features:** this is an alpha diversity metric that counts the number of **_present_ features** in your community.
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results_15000/observed_features_vector.qza \
--m-metadata-file metadata/metadata_fixed2.txt \
--o-visualization core_metrics_results_15000/observed_features_statistics.qzv
```
- not significant (NS)

**Faith's Phylogenetic Diversity (pd):** this is an alpha diversity metric that uses **phylogenetic information plus richness** (presence/absence of an organism) to determine alpha diversity.
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results_15000/faith_pd_vector.qza \
--m-metadata-file metadata/metadata_fixed2.txt \
--o-visualization core_metrics_results_15000/faiths_pd_statistics.qzv
```
- NS

For continuous covariates that we think could be associated with alpha diversity, we can use the alpha-correlation visualizer.
```
qiime diversity alpha-correlation \
--i-alpha-diversity core_metrics_results_15000/faith_pd_vector.qza \
--m-metadata-file metadata/metadata_fixed2.txt \
--o-visualization core_metrics_results_15000/faith_pd_correlation_statistics.qzv
```
- Spearman 
- VID NS
- Hour NS
- ORP NS
- pH NS
- DO p=0.0027
  
**Shannon's Diversity:** this is an alpha diversity metric that uses **richness** (presence/absence of an organism) _and_ **evenness** (organism relative abundance), but does **not** use phylogeny.
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results_15000/shannon_vector.qza \
--m-metadata-file metadata/metadata_fixed2.txt \
--o-visualization core_metrics_results_15000/shannon_statistics.qzv
```
- Treatment p=0.0648
- Date p=0.00648
- Period p=0.00648
- Time p=0.687

## Longitudinal

```
# make a longitudinal directory  
  
mkdir longitudinal  
  
cd longitudinal  
  
# construct a volatility plot  
  
qiime longitudinal volatility \
--m-metadata-file ../metadata/metadata_fixed2.txt \
--m-metadata-file ../core_metrics_results_15000/weighted_unifrac_pcoa_results.qza \
--p-state-column Hour \
--p-individual-id-column VID \
--p-default-group-column 'Treatment' \
--p-default-metric 'Axis 2' \
--o-visualization pc_vol_sample_type.qzv
```

```
# evaluate using first distances  
  
qiime longitudinal first-distances \
--i-distance-matrix ../core-metrics-results/weighted_unifrac_distance_matrix.qza \
--m-metadata-file ../metadata/metadata.txt \
--p-state-column add_0c \
--p-individual-id-column host_subject_id_sample_type \
--p-baseline 0 \
--o-first-distances from_first_wunifrac.qza
```

```
# visualize volatility  
  
qiime longitudinal volatility \
--m-metadata-file ../metadata/metadata.txt \
--m-metadata-file from_first_wunifrac.qza \
--p-state-column add_0c \
--p-individual-id-column host_subject_id \
--p-default-metric Distance \
--p-default-group-column 'sample_type' \
--o-visualization from_first_wunifrac_vol.qzv
```

```
# run LME   
  
qiime longitudinal linear-mixed-effects \
--m-metadata-file ../metadata/metadata.txt \
--m-metadata-file from_first_wunifrac.qza \
--p-state-column add_0c \
--p-individual-id-column host_subject_id \
--p-formula "Distance ~ add_0c + facility + sample_type" \
--o-visualization from_first_wunifrac_lme_formula.qzv
```

## Exporting Qiime2 data
```
# move to project directory

cd ../

mkdir export 
```
### Shannon
```
unzip core_metrics_results_10000/shannon_vector.qza -d export/shannon
```

move back to project directory if not there
### Observed Features  
```
unzip core_metrics_results_10000/observed_features_vector.qza -d export/observed_features  
```
### Faith's PD  
```
unzip core_metrics_results_10000/faith_pd_vector.qza -d export/faith_pd 
``` 
### Pielou's evenness  
```
unzip core_metrics_results_10000/evenness_vector.qza -d export/evenness
```
### Bray Curtis  
```
unzip core_metrics_results_10000/bray_curtis_pcoa_results.qza -d export/bray_curtis  
```
### Jaccard  
```
unzip core_metrics_results_10000/jaccard_pcoa_results.qza -d export/jaccard  
```
### Unweighted Unifrac  
```
unzip core_metrics_results_10000/unweighted_unifrac_pcoa_results.qza -d export/unweighted_unifrac  
```
### Weighted Unifrac  
```
unzip core_metrics_results_10000/weighted_unifrac_pcoa_results.qza -d export/weighted_unifrac
```

### Export
 
```
cd export 

mkdir alpha_div

# define alpha metrics  
metrics=("shannon" "evenness" "faith_pd" "observed_features")  
  
# copy their tsv files into alpha_div/  
for metric in "${metrics[@]}"; do  
 cp $metric/*/data/alpha-diversity.tsv alpha_div/${metric}.tsv  
done
```

```
mkdir beta_div

# define beta metrics  
metrics=("bray_curtis" "jaccard" "unweighted_unifrac" "weighted_unifrac")  
  
# copy their txt files into beta_div/  
for metric in "${metrics[@]}"; do  
 cp $metric/*/data/ordination.txt beta_div/${metric}.txt  
done
```

on your computer use terminal and navigate to 04_code 

```
for f in *_div.zip; do  
 unzip "$f" -d "${f%.zip}"  
done
```

```
cd /scratch/alpine/$USER/project/dada2

qiime feature-table transpose \--i-table table_nomitochloro_gg2.qza \--o-transposed-feature-table table_nomitochloro_transposed.qza

qiime metadata tabulate \--m-input-file table_nomitochloro_transposed.qza \--m-input-file oxy_seqs_dada2_filtered300.qza \--m-input-file ../taxonomy/taxonomy_gg2.qza \--o-visualization tabulated_results.qzv

```

## Move to R

### Shannon Model 
#### Fixed: Tx
- Residuals evenly distributed 
- Treatment p = 0.02167
#### Fixed: Hour
- residuals even 
- p = 0.5119
#### Fixed: Sequence
- Residuals evenly distributed 
- Seq p = 0.6835
#### Fixed: Period
- Residuals evenly distributed 
- Period p=0.1131

### Faiths Model