**Workflow overview**

**Importing the data into a qiime2 artifact**

The first step of the workflow is always to **import your fastq data into a QIIME2 artifact**. If your data is **multiplexed, you don't need a manifest**, we just need to tell QIIME2 the name of the directory where we put the raw reads and the type of data we are importing (which in this class is the **EMP paired end sequences**). 

**Demultiplexing**

**What is the difference between multiplexed and demultiplexed data? and what is the purpose of demultixplexing?**

When the data is multiplexed the samples are still pooled together in single fastq files. We need to demultiplex in order to assign the reads to our samples. This is possible due to the unique barcodes for each sample.

**Denoising with DADA2**

Then we denoise using DADA2 to generate our amplicon sequence variants "ASVs".

**What is the purpose of denoising?**

- **Quality filtering:** Trims off low-quality reads
- **Chimera checking:** Two DNA sequences that joined together which is considered a contaminate
- **Paired- end read joining:** Joining of forward and reverse reads

**Taxonomic classification**

**What's the purpose of using a taxonomy classifier?**

We use a pre-trained classifier that was trained on the V4 hypervariable region (corresponding to the 515F-806R primers) to assign taxonomy to our ASV (greengenes2 version 2024.09)

**Filtering**

after you classify your ASVs, a next step is commonly to filter the table. You can filter out chloroplasts and mitochondria, filter out rare features, or filter out samples (like controls AFTER you check them for quality).

**Generating a phylogenetic tree**

**Why would we want to generate a tree?**

Phylogenetic trees are useful tools in diversity analysis because they let us consider the evolutionary relationships between organisms. The tree is required for diversity analysis that incorporate phylogeny such as:

**Alpha diversity:** Faith's Phylogenetic Diversity

**Beta diversity:** Weighted UniFrac & Unweighted UniFrac

Recall that we use the **sepp method** that is available under the **fragment-insertion** plugin in QIIME2.

**Making a taxonomy barplot**

A taxonomy barplot provides us with a visual interactive barplot of the relative abundance of microbes at a desired taxonomic level.

**Now that we have assigned taxonomy. What is an important step that we should do to our feature table to get rid of a specific source of contamination?**

We should filter out Mitochondria & Chloroplast. If you are working with your own data you might also want to filter out your positive and negative controls for subsequent taxa plots AFTER you assess them for quality and contamination first.

**Alpha Rarefaction**

**Why is it important to rarefy? and how can we know at what level to rarefy?**

Because comparing samples with different numbers of reads can skew statistical analysis. So we subsample the same number of reads from each sample - this is called rarefying. We can determine where to rarefy by generating an alpha rarefaction curve and choosing the sequencing depth that captures the most amount of diversity within all the samples, and also keeps as many reads per sample as possible.

**Diversity metrics**

**Alpha diversity**

**What does alpha diversity tell us?**

In alpha diversity we are assessing the microbial diversity within a sample.

**Beta diversity**

**What does beta diversity tell us?**

In beta diversity, we are comparing the entire community of each sample to each other.

**Recall the beta diversity output files we get from core-metrics:**

- **Unweighted UniFrac:** Unweighted, phylogenetic. Measures the richness of the species and the number of shared branches in the phylogenetic tree. Unweighted UniFrac is **more sensitive to differences in low-abundance features**.
- **Weighted Unifrac:** Weighted, phylogenetic. Measures the **richness** and **evenness** of the species _and_ the **number of shared branches** in the phylogenetic tree. Weighted UniFrac is useful for **examining differences in community structure.**
- **Jaccard:** Unweighted, non-phylogenetic. Measures how many shared microbes there are between the samples. So the fraction of unique features, regardless of abundance.
- **Bray Curtis:** Weighted, non-phylogenetic. Uses richness and evenness to measure how many features are shared and the abundance of those features. Good when you have highly abundant species!

We can explore the PCoA plots for each from the core-metrics folder generated

---

**Lets get started on this weeks tutorial:** 

**1. One more file to export!**

OnDemand [https://ondemand-rmacc.rc.colorado.edu/pun/sys/dashboardLinks to an external site.](https://ondemand-rmacc.rc.colorado.edu/pun/sys/dashboard "(opens in a new window)") 

`ainteractive`

`module purge`  
`module load qiime2/2024.10_amplicon`

`cd /scratch/alpine/[$USER](mailto:lindsval@colostate.edu "(opens in a new window)")/decomp_tutorial/dada2`

`qiime feature-table transpose \--i-table table_nomitochloro.qza \--o-transposed-feature-table table_nomitochloro_transposed.qza`

`qiime metadata tabulate \--m-input-file table_nomitochloro_transposed.qza \--m-input-file seqs.qza \--m-input-file ../taxonomy/taxonomy_gg2.qza \--o-visualization tabulated_results.qzv`

#### **2. Directory Setup for R Analysis**

It is important when setting up a project to use a clear directory structure. Keeping raw data, processed data, scripts, results, and figures in separate folders helps maintain reproducibility and prevents accidental modification of original data. A well-structured project directory also makes analyses easier to understand, share, and rerun in the future. 

To start, create a new directory on your laptop called **`r_decomp_tutorial`**. If you’d like, you can place this directory inside your Obsidian vault. Next, create the following folders inside this directory so that the final file structure looks like this:

r_decomp_tutorial  
├── 01_notes  
├── 02_data  
├── 03_metadata  
├── 04_code  
└── 05_figures

The **`01_notes`** directory is where you can store any notes related to your project.  
The **`02_data`** directory should contain raw data (for example, sequencing data, or qiime2 outputs qzvs).  
The **`03_metadata`** directory will store all metadata associated with the project.  
The **`04_code`** directory is where your R scripts will be kept.   
The **`05_figures`** directory will store figures generated from your R analyses.

**3. Download data for R analysis**

Now that we have our directories set up, we need to download our **alpha and beta diversity metrics and the tabulated_results.tsv file** from Alpine. 

1. Log in to ondemand and navigate into the decomp_tutorial.

2. Navigate into the export directory and select the alpha_div and beta_div directories we made last week. Once they are selected, go ahead and download them. Move the alpha and beta diversity directories to the 04_code directory on your computer. 

3. Now open a terminal or PowerShell window and navigate into the 04_code directory. You'll notice that the alpha_div and beta_div directories are zipped, we need to unzip them. 

4. Unzip the directories

for f in *_div.zip; do  
 unzip "$f" -d "${f%.zip}"  
done

5. In Ondemand, download metadata.txt and move it into the 03_metadata directory. 

6. Open the tabulate_results.qzv in qiime2 view and download the tsv. It will download as metadata.tsv rename it to tabulated_results.tsv. In the 04_code directory, make a new directory named taxonomy. Move the downloaded tabulated_results.tsv into the taxonomy directory.

7. Download the R markdown here and move it into the 04_code directory.  

Now we are ready to work in R.