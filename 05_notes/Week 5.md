# Week 5 Tutorial: Taxonomy, Taxa Barplots, Filtering, Phylogenetic Tree [Mac Commands]

**Announcements:**

**If you have not already added your github link, please do so. this is the only way we can give you credit for your homework!**

[https://docs.google.com/spreadsheets/d/1CfZ8sRFpGaSOT8g9UEwxk2CZW8wkZUyxQj6Mmm84K1I/edit?gid=0#gid=0Links to an external site.](https://docs.google.com/spreadsheets/d/1CfZ8sRFpGaSOT8g9UEwxk2CZW8wkZUyxQj6Mmm84K1I/edit?gid=0#gid=0 "(opens in a new window)") 

**If you need to hear all of this material again, this is the perfect you tube series for you!**

Qiime2 youtube:[https://www.youtube.com/watch?v=M2iXewkYHE0&list=PLbVDKwGpb3XmkQmoBy1wh3QfWlWdn_pTTLinks to an external site.](https://www.youtube.com/watch?v=M2iXewkYHE0&list=PLbVDKwGpb3XmkQmoBy1wh3QfWlWdn_pTT "(opens in a new window)")[![](https://colostate.instructure.com/images/play_overlay.png)](https://www.youtube.com/watch?v=M2iXewkYHE0&list=PLbVDKwGpb3XmkQmoBy1wh3QfWlWdn_pTT)

**Last week on tutorial Fridays:**

- Denoised sequence data from 2 runs, merged denoising outputs
- Interpreted denoising outputs

**This week:**

- Quiz #2
- Homework 1 review
- Assign taxonomy using Greengenes2
- Taxa barplots
- Filtering the feature table
- Phylogenetic tree

**Taxonomy**

![image.png](https://colostate.instructure.com/courses/220471/files/39113904/preview)

**Taxonomic Classification**

Last week we denoised the sequences into ASVs. That representative sequences file contained Feature IDs and the sequences. Now, let's classify our ASVs to assign each one its microbial taxonomy. Recall that there are a **variety of databases** that can be used to classify taxonomy:

- [Greengenes2Links to an external site.](https://www.nature.com/articles/s41587-023-01845-1 "(opens in a new window)") - reference tree that includes integrated 16S rRNA regions, full-length 16S rRNA full genes, and whole genome information
- [SILVALinks to an external site.](https://www.arb-silva.de/ "(opens in a new window)") - high quality ribosomal RNA database
- [Ribosomal Database ProjectLinks to an external site.](https://www.glbrc.org/data-and-tools/glbrc-data-sets/ribosomal-database-project "(opens in a new window)") (RDP) - quality controlled, aligned, and annotated 16S rRNA sequences
- [NCBI Reference Sequence DatabaseLinks to an external site.](https://www.ncbi.nlm.nih.gov/refseq/ "(opens in a new window)") (RefSeq) - a comprehensive, integrated, non-redundant, well-annotated set of reference sequences including genomic, transcript, and protein.
- [GenBankLinks to an external site.](https://www.ncbi.nlm.nih.gov/genbank/ "(opens in a new window)") - the NIH genetic sequence database, an annotated collection of all publicly available DNA sequences
- [Genome Taxonomy DatabaseLinks to an external site.](https://gtdb.ecogenomic.org/about "(opens in a new window)") (GTDB) - a standardised microbial taxonomy based on genome phylogeny
- [Web of LifeLinks to an external site.](https://biocore.github.io/wol/ "(opens in a new window)") (WoL) - reference phylogeny for microbes

After some internal testing in our lab, we find we get the best, and most, classifications by using [Greengenes2Links to an external site.](https://www.nature.com/articles/s41587-023-01845-1 "(opens in a new window)").

**Key takeaways about Greengenes2 (gg2):**

- A reference tree that unifies genomic and 16S rRNA databases into a consistent, integrated source
- Includes nearly 16K bacterial and archaeal whole genomes, 18K full-length 16S rRNA sequences, 1.7 million near-complete 16S rRNA genes, and over 23 million 16S rRNA V4 region sequences, all compiled from a variety of sources
- Yields a reference tree covering over 21 million sequences from 31 different types of environments

For the purpose of this class, we will use the pre-trained classifier version of gg2 that was trained on the V4 hypervariable region (corresponding to the 515F-806R primers). 

**Let's get started!**

1. Log onto Alpine On-Demand: [ondemand-rmacc.rc.colorado.edu](http://ondemand-rmacc.rc.colorado.edu/ "(opens in a new window)")
2. Navigate to your decomp tutorial directory
3. open a terminal
4. Go to the taxonomy folder:  
    

`cd /scratch/alpine/$USER/decomp_tutorial/taxonomy`

 **5. Load the class node:**

sinteractive --reservation=aneq505 --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

 **6. Load your Qiime environment:**

module purge  
  
module load qiime2/2024.10_amplicon

**7.  wget the classifier:**

wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza

the next command, uses the **q2-feature-classifier** plugin to classify the ASVs by its **classify-sklearn method. This** is machine-learning-based classification, in which **classifiers must be** **_trained_**, e.g., to learn which **features best distinguish each taxonomic group**, adding an additional step to the classification process. 

Here is how you can visualize a "classifier": 

![Screenshot 2026-02-17 at 8.43.49 PM.png](https://colostate.instructure.com/courses/220471/files/39082514/preview)

we give the **classifier** a **reference database (**with **sequences** and the **associated taxonomic classification**) (green boxes)

And tell the classifier, if you see a sequence like this, you should be returning a taxonomy label like this one (red boxes)

**8.  use our representative reads to classify taxonomy, which will give us our taxonomy.qza output. This will take ~2 minutes.**

`qiime feature-classifier classify-sklearn \`  
`--i-reads ../dada2/seqs.qza \`  
`--i-classifier 2024.09.backbone.v4.nb.qza \`  # from greeengenes
`--o-classification taxonomy_gg2.qza`


**9. Let's make our taxonomy into a visualization, transfer to our local computers, and look at it in QIIME2 View.**

qiime metadata tabulate \  
--m-input-file taxonomy_gg2.qza \  
--o-visualization taxonomy_gg2.qzv

Let's take a look at our taxonomy file using [view.qiime2.orgLinks to an external site.](https://view.qiime2.org/ "(opens in a new window)")!

- we can see that we get an output file with each Feature ID, its taxonomy, and the confidence level.
- Helpful QIIME2 forum post on how confidence is calculated (hint: it isn’t simple): [https://forum.qiime2.org/t/how-is-the-confidence-calculated-with-taxa-assignments/179/3Links to an external site.](https://forum.qiime2.org/t/how-is-the-confidence-calculated-with-taxa-assignments/179/3 "(opens in a new window)")

**10. Since there are over 400 samples, we are first going to group by type_days - a metadata column that includes both sample type and days since placement. This will make our results a lot more interpretable**

qiime feature-table group \  
--i-table ../dada2/table.qza \  
--m-metadata-file ../metadata/metadata.txt \  
--m-metadata-column type_days \  
--p-mode mean-ceiling \  
--p-axis sample \  
--o-grouped-table ../dada2/table_type_days.qza

**11. Now, using our grouped feature table, we will remove contaminating features like chloroplasts and mitochondria.** 

- Things we often want to remove are plant-derived DNA or eukaryotically-derived DNA, but you can remove other contaminates too here by adding that taxonomy to the --p-exclude line. 
- The reason these show up in our data is due to the endosymbiotic theory, that mitochondria & chloroplasts, likely originating as bacteria, became symbionts in cytoplasm of eukaryotic cells. 
- note that sp004296775 is another chloroplast, it MUST also be removed. see this forum [post.Links to an external site.](https://forum.qiime2.org/t/silva-vs-greengenes2-2024-9-yield-different-taxonomic-classifications-of-chloroplast/31889/7 "(opens in a new window)")
```
qiime taxa filter-table \  
--i-table ../dada2/table_type_days.qza \  
--i-taxonomy taxonomy_gg2.qza \  
--p-exclude mitochondria,chloroplast,sp004296775 \  
--p-include c__ \  
--o-filtered-table ../dada2/table_type_days_nomitochloro_include.qza
```
### **Taxa Barplots**

Now that we have ASVs classified taxonomically, let's generate a taxa barplot to visualize it, transfer it to our local computers, and look at it in QIIME2 View.

**1. Move to the taxa plots directory**

`cd ../taxaplots`

**2. Generate the taxa bar plot** 

qiime taxa barplot \  
--i-table ../dada2/table_type_days_nomitochloro.qza \  
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \  
--o-visualization taxa_barplot_type_days_nomitochloro.qzv

A quick way to search for specific taxa is to open your taxa bar plot and use command-F (or ctrl-F for PC) to search for that taxa. Check that chloroplasts and mitochondria were filtered out.

...

Even though these taxa weren't present, it's good practice for you now because it is standard for these things to be filtered out. It's also a good habit to think about taxa you might need to filter out when doing your own analyses.

**3. Check the controls**

First, you will need to filter down to just the controls

# First filter samples

qiime feature-table filter-samples \  
--i-table ../dada2/table.qza \  
--m-metadata-file ../metadata/metadata.txt \  
--p-where "[sample_type]='control'" \  
--o-filtered-table ../dada2/table_controls.qza

# Create a taxa barplot with our table with just controls

qiime taxa barplot \  
--i-table ../dada2/table_controls.qza \  
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \  
--m-metadata-file ../metadata/metadata.txt \  
--o-visualization taxa_barplot_controls.qzv

Why do we check controls?

- Contamination
- Workflow validation

Why include positive controls?

- Positive controls contain known combinations and quantities of microbiomes. This allows you to confirm whether your DNA extraction and library preparation protocol was successful and is suitable for identifying a variety of different organisms.

![Screenshot 2026-02-18 at 3.02.21 PM.png](https://colostate.instructure.com/courses/220471/files/39092121/preview)

What should you do with taxa found in extraction controls?

- Report them
- Remove them statistically with tools designed for contamination removal
- In most cases, researchers should avoid simplistic removals of taxa , OTUs, or ASVs appearing in negative controls, as many will be microbes from other samples rather than reagent contaminants.

You've just done your first taxonomic analysis, congratulations!

**Questions:**

1. Do you see any differences in taxa between sample type? What about facility? 
2. What taxa did you notice in the extraction controls?

**Pro tips/extra info:**

1. If you ever want to train your own classifier, here is a link to the tutorial on the QIIME2 website; [https://docs.qiime2.org/2021.11/tutorials/feature-classifier/Links to an external site.](https://docs.qiime2.org/2021.11/tutorials/feature-classifier/ "(opens in a new window)")
2. Here is a link to some data resources for picking out pre-trained classifiers and reference databases: [https://docs.qiime2.org/2021.11/data-resources/Links to an external site.](https://docs.qiime2.org/2021.11/data-resources/?highlight=silva "(opens in a new window)")
3. Instead of filtering your feature table, QIIME2 also allows you to filter your taxonomy.qza. Either way will work just fine.
4. To learn more about filtering data, visit the following filtering tutorial on the QIIME2 website; [https://docs.qiime2.org/2023.9/tutorials/filtering/Links to an external site.](https://docs.qiime2.org/2023.9/tutorials/filtering/ "(opens in a new window)")

**Phylogenetic Trees**

You have already learned about phylogenetic trees during the lecture portion of our class, but let's quickly review some of that information. Phylogenetic trees are useful tools in diversity analysis because they let us consider the evolutionary relationships between organisms. There are several ways in which trees can be constructed, and shown below are the common ways for constructing trees using 16S rRNA data.

![image.png](https://colostate.instructure.com/courses/220471/files/38493069/preview)

If you are only interested in a quick and dirty analysis, then you can use a de novo approach to create what is called a "fast tree" using the align-to-tree-mafft-fasttree pipeline in the phylogeny plugin in QIIME2. While we don't recommend this for publishing purposes, it can be useful for quickly exploring your data if needed because the insertion tree method can take a long time.

For this course, we will be using the **sepp method** that is available under the **fragment-insertion** plugin in QIIME2. This method uses a backbone tree from a reference database, in this case from Greengenes2, though this can be changed using the --i-reference-database parameter. It attempts to match the representative sequences in our samples with sequences in the tree, and if it cannot it finds the best branch point and adds the unannotated sequences in. This ensures that all the sequences are included in the tree, and that there is valuable structure. This is the recommended method to construct your phylogenetic trees because it is much more accurate and informative.

Let's navigate to our tree directory and download the reference tree. You can look at your options at [https://docs.qiime2.org/2021.11/data-resources/Links to an external site.](https://docs.qiime2.org/2021.11/data-resources/ "(opens in a new window)") under the SEPP reference databases heading at the bottom. As mentioned above, we are using the Greengenes2 database today, which can be found at [https://ftp.microbio.me/greengenes_release/2022.10/Links to an external site.](https://ftp.microbio.me/greengenes_release/2022.10/ "(opens in a new window)") 

**1. Go to the tree directory**

`cd ../tree`

**2. Get the reference backbone**

wget https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza

For a full description of all the options, plus some extras that we didn't use today, look at the [documentation page. Links to an external site.](https://docs.qiime2.org/2021.11/plugins/available/fragment-insertion/sepp/#sepp-insert-fragment-sequences-using-sepp-into-reference-phylogenies "(opens in a new window)")

Note that the plugin we are using is `fragment-insertion`, and the method is sepp. We use the representative sequences here, not the table, because this is completely independent of your samples; all it needs is a list of the sequences in your dataset and the ASV label associated.

As an output, we get the `tree.qza` that we need for diversity analyses. We also get a `tree_placements.qza` file, which has information about where it places your features before the actual rooted tree is generated. We get this because it is an intermediate file that is generated during the tree placement, but we don't actually need to look at it (unless you get really invested in your tree and phylogenetics in the future).

This job can take a while (especially in real datasets), so we'll submit it as a job. Here, it may take around 20 minutes.

Note: Remember that before submitting a job you need to **exit the interactive node first before submitting**.

cd ../slurm

nano sepp_script.sh

#!/bin/bash  
#SBATCH --job-name=sepp  
#SBATCH --nodes=1  
#SBATCH --ntasks=24  
#SBATCH --partition=amilan  
#SBATCH --time=04:00:00  
#SBATCH --mail-type=ALL  
#SBATCH --mail-user=<YOUR EMAIL HERE>@colostate.edu  
#SBATCH --output=slurm-%j.out  
#SBATCH --qos=normal  
  
  
#Activate qiime  
  
module purge  
module load qiime2/2024.10_amplicon  
  
# go to your decomp directory  
cd /scratch/alpine/$USER/decomp_tutorial  
  
#frament insertion sepp  
qiime fragment-insertion sepp \--i-representative-sequences dada2/seqs.qza \--i-reference-database tree/2022.10.backbone.sepp-reference.qza \--o-tree tree/tree_gg2.qza \--o-placements tree/tree_gg2_placements.qza \--p-threads 4

sbatch sepp_script.sh

Congratulations! Now you have a tree! QIIME2 doesn't have a real way to visualize these, since a tree is basically a tool for future analyses, but if you get super interested in what it looks like you can plug it into the iTol visualizer here: _[https://itol.embl.de/upload.cgiLinks to an external site.](https://itol.embl.de/upload.cgi "(opens in a new window)")[.Links to an external site.](https://itol.embl.de/upload.cgi "(opens in a new window)")_

### **Summary**

Let's review what we've done so far!

![image.png](https://colostate.instructure.com/courses/220471/files/38493077/preview)

Green = File(s), Yellow = Qiime2 actions, Blue = Visualization

[https://docs.qiime2.org/2023.5/tutorials/overview/#conceptual-overview-of-qiime-2](https://docs.qiime2.org/2023.5/tutorials/overview/#conceptual-overview-of-qiime-2 "(opens in a new window)")