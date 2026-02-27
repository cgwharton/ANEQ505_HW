# Week 6 Tutorial: Alpha Rarefaction, Core Metrics, Alpha Diversity Plots [PC Commands]

---

Val's follow ups from last week... 

**1. Remember those long amplicons we saw in the COW dataset? (the ones that were ~425bp, seemingly they all pop up at 18S sequences when you BLAST them). This happens from some unintended affinity for our primers for the 18S rRNA gene of eukaryotes... we originally thought these would come up as Unclassified and out be filtered out by using --p-include c__, but.... they are classifies as bacteria in GG2 (not sure why yet). regardless, thats not good! they need to go...**  

Even though we have done this tutorial enough times to know that it wont affect the results drastically, the best practice would be to remove those long amplicons. (the technical reason on why we may have missed this in the past is because our lab used to use [DeblurLinks to an external site.](https://journals.asm.org/doi/10.1128/msystems.00191-16 "(opens in a new window)") to denoise our sequences. Deblur uses forward reads only, and that's the type of sequencing we've had in the past. Deblur will automatically filter out any non-16S looking amplicons because of its specific error correcting algorithm).  

With dada2, we use longer reads (2x250 can give you longer reads because technically we have ~500bp of sequences, so thats why we get those longer amplicons- they merge in a different place than 16s v4 region would, and due to the error correcting algorithm from dada2, these longer amplicons are not filtered out because they are technically real biological signal). 

_Removing ASVs based on length:_ 

#Remove amplicons larger than 300bp

`qiime feature-table filter-seqs \``--i-data cow_seqs_dada2.qza \``--m-metadata-file cow_seqs_dada2.qza \``--p-where 'length(sequence) < 300' \``--o-filtered-data cow_seqs_dada2_filtered300.qza`

`qiime feature-table tabulate-seqs \``--i-data cow_seqs_dada2_filtered300.qza \``--o-visualization cow_seqs_dada2_filtered300.qzv`

`qiime feature-table filter-features \``--i-table cow_table_dada2.qza \``--m-metadata-file cow_seqs_dada2_filtered300.qza \``--o-filtered-table cow_table_dada2_filtered300.qza`

`qiime feature-table summarize \``--i-table cow_table_dada2_filtered300.qza \``--m-sample-metadata-file ../metadata/cow_metadata.txt \``--o-visualization cow_table_dada2_filtered300.qzv`

**2. about using taxonomic classifiers, add info on using the pre-trained v4 silva and where to find that**

For qiime2 versions 2024.10 and later, the qiime2 developers **will not be providing pre-trained naive bayes classifiers for Silva 138 for the 515F/806R region**. See my forum [postLinks to an external site.](https://forum.qiime2.org/t/silva-v4-classifier/32410/2 "(opens in a new window)"). 

- One of the forum moderators does have a link for pre trained naive bayes v4 silva classifiers [https://bwsyncandshare.kit.edu/s/zK5zjAbsTFQRpboLinks to an external site.](https://bwsyncandshare.kit.edu/s/zK5zjAbsTFQRpbo "(opens in a new window)")
- the downside is that who knows how long he will keep providing the classifiers for each new qiime2 version... 
- so if you really want to use silva, you have to [train your own classifierLinks to an external site.](https://docs.qiime2.org/2021.11/tutorials/feature-classifier/ "(opens in a new window)"), or you can use an older version of qiime (one that is compatible with the older classifiers)
    - to do this, you would go to the data resources page: [https://docs.qiime2.org/2024.10/data-resources/Links to an external site.](https://docs.qiime2.org/2024.10/data-resources/ "(opens in a new window)")
    - and then use the drop down menu on the left side to chose the version of qiime2 you want to use, install that version of qiime2, and then go through your analysis that way and that way you know the pre-trained classifiers are compatible with your qiime2 version!

See example for qiime2 version 2023.7: 

[https://docs.qiime2.org/2023.7/data-resources/Links to an external site.](https://docs.qiime2.org/2023.7/data-resources/ "(opens in a new window)") 

![Screenshot 2026-02-25 at 1.27.17 PM.png](https://colostate.instructure.com/courses/220471/files/39175041/preview)

**3. How to know if a taxonomic classifier is incompatible with your version of qiime2?** 

if you are using an incompatible version of a classifer / qiime2 combo, you will get an error like this: 

`Plugin error from feature-classifier: The scikit-learn version (1.4.2) used to generate this artifact does not match the current version of scikit-learn installed (0.24.1). Please retrain your classifier for your current deployment to prevent data-corruption errors.Debug info has been saved to /scratch/alpine/lindsval@colostate.edu/.tmp/qiime2-q2cli-err-n7odp7uq.log`

so this says that there were different versions of scikit-learn that were used to train the classifier, and that it is incompatible with the version of qiime I was using. In this case, you have to use **QIIME2 version 2024.10** **or later** with the **pre-trained 2024.09 naive bayes v4 version of Greengenes2**.

**4. Back to the decomp tutorial! Lets run the taxa bar plot code for the whole dataset as well & explore**

**Alpine On-Demand: [ondemand-rmacc.rc.colorado.edu](http://ondemand-rmacc.rc.colorado.edu/ "(opens in a new window)")**

sinteractive --reservation=aneq505 --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

  
module purge  
module load qiime2/2024.10_amplicon  
  
cd /scratch/alpine/$USER/decompu_tutorial/taxaplots  
  
qiime taxa filter-table \--i-table ../dada2/table.qza \--i-taxonomy ../taxonomy/taxonomy_gg2.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--p-include c__ \--o-filtered-table ../dada2/table_nomitochloro.qza  
  
#visualize:   
qiime taxa barplot \--i-table ../dada2/table_nomitochloro.qza \--i-taxonomy ../taxonomy/taxonomy_gg2.qza \--m-metadata-file ../metadata/metadata.txt \--o-visualization taxa_barplot_all_samples.qzv

---

**Alpha Rarefaction, Core Metrics, Alpha Diversity Plots**

### **Alpha Rarefaction**

Now that we have our tree and we are ready to start looking at the diversity of our samples, we need to normalize first. In our last lecture, you learned about rarefying as an option for normalizing your data. As a reminder, we do this because the number of reads differs between samples and this can be unfair when you are doing any statistical analysis. For example, say you have sample 1 with 100 reads and sample 2 with 10,000 reads. Then, say you want to compare how much E. coli is in sample 1 versus sample 2. It's likely that sample 2 will have more E. coli compared to sample 1. Is this because the environment that sample 2 was collected from has a biological relevant reason for having more E. coli, or is this because sample 2 happened to have more reads sequenced in that sample and having more E. coli is just an artifact of the PCR and sequencing chemistry? The answer is that we really don't know. This makes comparing samples that have different numbers of reads in them unfair and this results in invalid statistical analyses. So, before we can compare our samples to each other, we have to subsample (without replacement) the same number of reads from each sample - this is called rarefying. How do we know how many reads to pull from each sample?

*if you wanted to do absolute abundance youd do PCR or metagenomics (super expensive)*

The command below shows one tool we can use to determine where to rarefy. The plot we are generating subsamples our feature table at different depths and calculates the alpha diversity at each of these. We are looking for the point when the graphs level off, which suggests that at that point we are not gaining any more diversity by adding more sequences. Ideally, we would rarefy at a level where all of the samples have leveled off, though this is not always possible.

We chose 10000 for the max depth because the sample with the most features has 26204 sequences and when we look at the table.qzv we see that we start losing samples around 11000 sequences, and we want to capture all possibilities. By default, 10 rarefied tables are calculated at each sampling depth to provide an error estimate. 

Before running alpha rarefaction and core metrics we want to filter out the controls from our table.

cd ..  
  
qiime feature-table filter-samples \--i-table dada2/table_nomitochloro.qza \--m-metadata-file metadata/metadata.txt \--p-where "NOT [sample_type] IN ('control') " \--o-filtered-table dada2/table_nomitochloro_nocontrol.qza  

Now lets run alpha rarefaction

mkdir alpha_rarefaction  
  
cd alpha_rarefaction  
  
qiime diversity alpha-rarefaction \--i-table ../dada2/table_nomitochloro_nocontrol.qza \--m-metadata-file ../metadata/metadata.txt \--p-max-depth 10000 \--o-visualization alpha_rarefaction_curves.qzv

![Screenshot 2026-02-25 at 9.00.07 PM.png](https://colostate.instructure.com/courses/220471/files/39185419/preview)

The visualization file will display two plots. The upper plot will display the alpha diversity (observed features or shannon) as a function of the sampling depth. This is used to determine **whether the richness or evenness has saturated based on the sampling depth. The rarefaction curve should “level out” as you approach the maximum sampling depth.**

The second plot shows the number of samples in each metadata category group at each sampling depth. This is useful to determine the sampling depth where samples are lost, and whether this may be biased by metadata column group values.

After we have this visualization, we can use the rarefaction curves and the visualization of our DADA2 feature table to decide where to rarefy. Looking at these two visualizations, where would you rarefy? 

### **Alpha Diversity Review**

**_What are we measuring with alpha diversity?_**

![Screenshot 2024-02-13 at 1.47.32 PM.png](https://colostate.instructure.com/courses/177609/files/30566297/preview)

****flower type and color are a metaphor for diversity, we are not talking about the microbes of the flowers. If the left picture were a field of microbes, the alpha diversity would be extremely low because the only two species are yellow tulips, grass and trees. Versus the flower bed on the right has at least 50+ species of flowers so the alpha diversity would be high.** 

Alpha diversity is measuring your **richness** (or presence and absence of organisms), the **evenness** (the abundance of the organisms) and we can also add in the **phylogeny** to "weight" our alpha/within sample diversity based on shared evolutionary history.

Thus, alpha diversity uses the **metadata,** **feature table**, and a **phylogenetic tree,** which describes the evolutionary relationship between your organisms, to determine the within sample diversity.

---

### **Running the Diversity Pipeline (Core-Metrics) to Generate Alpha and Beta Diversity**

One really nice thing about QIIME2 is that we can get nearly all of the diversity metrics (alpha and beta) that we could possibly want with just one command. This command, called **core-metrics**, is what we call a pipeline - it runs a series of analyses with just one command. 

If you look in the [diversity plugin documentationLinks to an external site.](https://docs.qiime2.org/2021.11/plugins/available/diversity/ "(opens in a new window)"), you will see a lot of options! There are a few pipelines, and the one we will be using is the core-metrics pipeline with and without phylogeny (because we just constructed our tree, so we definitely want to incorporate phylogeny into our analyses). You may be interested in the non-phylogenetic pipeline if you don't have a tree. While we recommend using a tree, in some cases you don't have the raw data that you need available to you in order to construct a tree, so QIIME2 still has options for analyses in those cases.

In addition to **pipelines,** there are also **methods.** If you only care about alpha OR beta diversity, you can use the appropriate method to just get those metrics. 

Today we will run the **core-metrics-phylogenetic pipeline** because we are going to be looking at both alpha and beta diversity, and we are going to incorporate phylogeny. 

Note that the output here is a directory. This is because it generates so many files that it would be ridiculous to ask you to name all of them. So, QIIME2 has standard diversity output names, and you just choose the name of the directory where you want them to go.

**Run Core-Metrics**

- uses the filtered table, we don't want stats on mitochondria or chloroplasts!
- the tree allows us to use phylogenetic diversity metrics
- we use the sample depth of 1500 as chosen from the alpha rarefaction plots.
- a new directory is created for you with all of the results, and files are pre-named for you.
- This should take about 5 mins

# cd back into the main decomp_tutorial directory  
cd ../  
  
qiime diversity core-metrics-phylogenetic \--i-table dada2/table_nomitochloro_nocontrol.qza \--i-phylogeny tree/tree_gg2.qza \--m-metadata-file metadata/metadata.txt \--p-sampling-depth 1500 \--output-dir core-metrics-results

After this completes (~2mins), explore your output to see what files QIIME2 generated.

cd core-metrics-results  
  
ls core-metrics-results  
  
cd ../ #move out of the core-metrics directory  
  
pwd # make sure you are back in your decomp_tutorial

As you can see, this command generates many outputs for the most common alpha and beta diversity test. Some are QIIME 2 artifacts and others are both the artifacts (.qza) and the visualizations (.qzv). 

### **Alpha Diversity Files**

**Which core-diversity-metric files are alpha diversity and what do they mean? (ordered here from simplest to more complex):**

**1. Observed features:** this is an alpha diversity metric that counts the number of **_present_ features** in your community.

`core-metrics-results/observed_features_vector.qza` 

**2. Pielou's evenness:** an alpha diversity metric that measures relative **evenness** (a comparison of organism abundance) of species richness

`core-metrics-results/evenness_vector.qza`

**3. Faith's Phylogenetic Diversity (pd):** this is an alpha diversity metric that uses **phylogenetic information plus richness** (presence/absence of an organism) to determine alpha diversity.

`core-metrics-results/faith_pd_vector.qza`

**4. Shannon's Diversity:** this is an alpha diversity metric that uses **richness** (presence/absence of an organism) _and_ **evenness** (organism relative abundance), but does **not** use phylogeny. 

`core-metrics-results/shannon_vector.qza` 

#### **Alpha group significance**[Links to an external site.](https://docs.qiime2.org/jupyterbooks/cancer-microbiome-intervention-tutorial/030-tutorial-downstream/060-alpha-diversity.html#alpha-group-significance "Permalink to this headline (opens in a new window)")

For categorical data, we use the **alpha-group-significance visualizer**. Each sample gets one alpha diversity metric because, unlike beta diversity, the math is independent of other samples- more on that later! For now, just remember that **alpha diversity is within-sample diversity**. So, we don't have to input the group of interest here, the only decision we need to make is which diversity metric we're interested in.

First we’ll look for **general patterns in alpha diversity across samples** by comparing different **categorical groupings** of samples to see if there is some relationship to richness. This uses a **Kruskal-Wallis H test** which is a rank-based nonparametric test that can be used to determine if there are **statistically significant differences between two or more groups**. 

To start with, we’ll examine **‘observed features’:**

qiime diversity alpha-group-significance \--i-alpha-diversity core-metrics-results/observed_features_vector.qza \--m-metadata-file metadata/metadata.txt \--o-visualization core-metrics-results/observed_features_statistics.qzv

**Is there a significant difference in the number of observed features between any of the categorical data?** 

We'll go ahead and try using the alpha-group-significance visualizer with the **Shannon** index (looks at richness and evenness) and the **Faith's Phylogenetic Diversity** index (looks at richness while incorporating phylogeny). 

qiime diversity alpha-group-significance \--i-alpha-diversity core-metrics-results/shannon_vector.qza \--m-metadata-file metadata/metadata.txt \--o-visualization core-metrics-results/shannon_statistics.qzv  
  
qiime diversity alpha-group-significance \--i-alpha-diversity core-metrics-results/faith_pd_vector.qza \--m-metadata-file metadata/metadata.txt \--o-visualization core-metrics-results/faiths_pd_statistics.qzv

**When we consider richness and evenness (Shannon's Diversity), is there a significant difference between sample type? What about the facility?**

**Is there a difference in phylogenetic diversity (so Faith's PD) between sample type? facility?**

For continuous covariates that we think could be associated with alpha diversity, we can use the alpha-correlation visualizer. In this study, a couple of variables we could look at include add_0c and days since placement.

qiime diversity alpha-correlation \--i-alpha-diversity core-metrics-results/faith_pd_vector.qza \--m-metadata-file metadata/metadata.txt \--o-visualization core-metrics-results/faith_pd_correlation_statistics.qzv

Since most of our experiments are slightly more complex than just comparing across one categorical or continuous variable, there is the [QIIME2 longitudinal plugin.    Links to an external site..](https://docs.qiime2.org/2021.11/plugins/available/longitudinal/ "(opens in a new window)") We will explore longitudinal analysis of alpha diversity metrics more in-depth during the longitudinal tutorial.

---

**Homework #2**

Now that we have imported the reads, demultiplexed all the samples, and denoised the reads into ASVs (generating the representative seqs file, the feature table, and the denoising stats), we can assign taxonomy to those ASVs and generate some taxa bar plots. Finally, the homework will end with creating a job script for the SEPP tree. 

**Note**: we are adding in the filtering step (the commands are written for you) so you can remove large/contaminating amplicons from the seqs file and the feature table. 

**To do:** 

- Go to the homework 2 materials module to download your respective obsidian file (pc or mac) to get started.
- due March 6th at midnight 
- push to github to submit

Email us with any questions or problems!

---

#### **Note about VS Code and Obsidian**

Here is a [link](https://colostate.instructure.com/courses/220471/files/39191108?wrap=1 "VS_code_in_obsidian.md (opens in a new window)") [Download link](https://colostate.instructure.com/courses/220471/files/39191108/download?download_frd=1)[Open this document with ReadSpeaker docReader](https://docreader.readspeaker.com/docreader/?cid=11403&lang=en_us&url=https%3A%2F%2Finst-fs-iad-prod.inscloudgate.net%2Ffiles%2Fdb776124-2cb1-4940-b441-44e9183a9efe%2FVS_code_in_obsidian.md%3Ftoken%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE3NzIyMTM2NTcsInVzZXJfaWQiOm51bGwsInJlc291cmNlIjoiL2ZpbGVzL2RiNzc2MTI0LTJjYjEtNDk0MC1iNDQxLTQ0ZTkxODNhOWVmZS9WU19jb2RlX2luX29ic2lkaWFuLm1kIiwiaG9zdCI6bnVsbCwianRpIjoiZmRhM2E3OWYtNzk4OC00ZGI3LTkxNDAtZjc3YTBkOWQ1MWUzIiwiZXhwIjoxNzcyMzAwMDU3fQ.hsXU_D1swwhSiVbMfLMfvgZQeZh50IrLoengyy8ilIl5yWzx-QlZQyHQox90YJ18c_w3kq_d3zC19n0WninfYQ "Open this document with ReadSpeaker docReader (opens in a new window)")to instructions to run VS Code through Obsidian using the VS Code in Obsidian community plugin. You can use this plugin to directly open VS Code in your Obsidian vault and then check your code for hidden spaces. Saving the code in VS Code will automatically save it in Obsidian.