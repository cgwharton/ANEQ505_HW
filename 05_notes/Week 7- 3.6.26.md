# Week 7 Tutorial: Beta Diversity, Longitudinal Analysis, Exporting Qiime2 Data [Mac]

**Last week on tutorial Fridays:**

- How to deal with 18S contamination 
- Alpha rarefaction
- Running core metrics
- Alpha diversity visualizations

**This week:**

- Terminology updated!
- Homework 2 review
- Beta diversity, beta diversity stats (part 1) on location
- Beta diversity stats (part 2) on repeated measures, longitudinal data, Linear Mixed Effects (LME) models
- exporting qiime2 data for R
- get started on homework 3

---

### **Beta diversity metrics, PCoA plots, and significance testing**

**_What are we measuring with beta diversity?_** 

_![Screenshot 2024-02-13 at 1.54.01 PM.png](https://colostate.instructure.com/courses/177609/files/30566470/preview)_

****Here we are comparing the entire community of each sample (flower color/type) to each other. Based on this, there aren't many shared flower types between these two pictures, suggesting the beta diversity would be large (closer to 1) and the communities are therefore dissimilar.** 

**Distance Matrices:**

You may recall from **alpha diversity that each sample has its own alpha diversity measure**. If you want, you can add alpha diversity as a column to your metadata because no matter what comparison you're making the alpha diversity for a sample is the same as any other value in the metadata (e.g., day you took the sample). In **beta diversity**, however, it is very dependent on which two samples we’re comparing. So we end up with a matrix like this because **we are comparing the samples to one another**:

 **![image.png](https://colostate.instructure.com/courses/220471/files/39277608/preview)**

Notice the sample names at the top are repeated in each row. 

So what we're looking at is the **distance those two samples are from each other _and_ how different they are**. These are usually **calculated on a scale from 0 to 1, with 0 being exactly the same, and 1 being more "distant" or more "dissimilar" and 0 being "closer" or "more similar".** So, these two samples in this top left hand corner are exactly the same (so you'll see that the distance is 0 because they're the same sample- so there's no difference between them).

But! If the samples are somewhat different (**Highlighted red box)**, we see a distance of ~0.6. So again what we're looking at is a **direct comparison** between every single sample in your entire data set to determine how similar or different the microbial communities are. 

**Because of the complexity/size,** we don't usually look at the distance matrices themselves but rather we visualize them with ordination plots like Principal Coordinates Analysis (PCoA)/umaps/NMDS/etc. 

### **Beta Diversity Files**

**Which core-diversity-metric files are beta diversity and what do they mean? Plus, let's check them out in Emperor**

- **Weighted =** considers ASV abundance
- **Unweighted =** does not consider ASV abundance, but rather presence or absence.
- **Phylogenetic** **=** measures the degree to which species are phylogenetically related
- **Non-phylogenetic =** does not account for phylogenetic relatedness.

**Unweighted UniFrac:** Unweighted, phylogenetic. Measures the richness of the species and the number of shared branches in the phylogenetic tree. Unweighted UniFrac is **more sensitive to differences in low-abundance features**. May overrepresent rare features because everything that is present is counted as 1.

- `core-metrics-results/unweighted_unifrac_emperor.qzv` 
    

**Weighted Unifrac:** Weighted, phylogenetic. Measures the **richness** and **evenness** of the species _and_ the **number of shared branches** in the phylogenetic tree. Weighted UniFrac is useful for **examining differences in community structure.** You may see less separation in this visualization compared to the unweighted, that is because the most prominent features are present similarly in both groups. So taxa abundance is accounted for.

- `core-metrics-results/weighted_unifrac_emperor.qzv` 
    

**Jaccard:** Unweighted, non-phylogenetic. Measures how many shared microbes there are between the samples. So the fraction of unique features, regardless of abundance. 

- `core-metrics-results/jaccard_emperor.qzv` 
    

**Bray Curtis:** Weighted, non-phylogenetic. Uses richness and evenness to measure how many features are shared and the abundance of those features. Good when you have highly abundant species!

- `core-metrics-results/bray_curtis_emperor.qzv` 
    

Let's explore the following files: 

- `core-metrics-results/unweighted_unifrac_emperor.qzv` 
- `core-metrics-results/bray_curtis_emperor.qzv` 

**As you click through different metadata colorings, are you seeing any separation between the data points? What metadata category best describes this separation?** 

**Are there any visual differences between unweighted unifrac and bray curtis?**

Note that these visualizations are just highlighting the qualitative differences (things we can see that looks like real differences), we need stats to tell us about the actual quantitative differences (what is statistically different). 

---

### **Testing for** **Significance**

**how do we know what is significant or meaningful to our analysis? We test using Beta Group Significance commands in a variety of ways.**

Beta Group Significance will determine whether groups of samples are significantly different from one another using a permutation-based (PERMANOVA) statistical test.

We are testing the hypothesis that samples within a group are more similar to each other than they are in samples to another group. In other words, it tests whether the within-group distances from each group are different from the between-group distance. Samples that are similar to each other will have smaller distances from each other. Read [this paperLinks to an external site.](https://onlinelibrary.wiley.com/doi/full/10.1111/j.1442-9993.2001.01070.pp.x "(opens in a new window)") for more about PERMANOVA. 

Use **PERMANOVA** when:

- You want to test whether **overall community composition differs between groups**
- Your samples are **independent**
- You are testing **categorical predictors** (e.g., treatment, sample type, facility)
- You are working directly with a **distance matrix** (Bray–Curtis, UniFrac, Jaccard, etc.)

**PERMANOVA** is ideal for:

- Cross-sectional studies
- One-time-point comparisons
- Simple group comparisons

You cannot use **PERMANOVA** when:

- You have **repeated measures** (longitudinal data), this is because samples from the same subject are **not independent**
- You want to model **continuous predictors** (e.g., time)
- You want to include **multiple covariates with complex structure**

PERMANOVA assumes independence of samples. In longitudinal microbiome data, that assumption is violated because samples from the same subject over time are correlated.

***PERMANOVA is not appropriate for this study; however, we want to demonstrate how to run PERMANOVA so you can use it for your homework assignment, where it is appropriate.**

### **Beta diversity stats (part 1)**

**Alpine On-Demand: [ondemand-rmacc.rc.colorado.edu](http://ondemand-rmacc.rc.colorado.edu/ "(opens in a new window)")**
```
sinteractive --reservation=aneq505 --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

module purge
module load qiime2/2024.10_amplicon  
```


**Running a PERMANOVA via the beta group significance test command in qiime2**

Here, we are specifying **"body_site"** to determine whether it is associated with significant differences in unweighted UniFrac distance & bray curtis. 

# unweighted unifrac significance  
  ```
qiime diversity beta-group-significance \
--i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \
--m-metadata-file metadata/metadata.txt \
--m-metadata-column body_site \
--o-visualization core-metrics-results/unweighted_unifrac_distance_matrix.qzv 
  ```

# bray curtis significance  
  

```
qiime diversity beta-group-significance \  
--i-distance-matrix core-metrics-results/bray_curtis_distance_matrix.qza \  
--m-metadata-file metadata/metadata.txt \  
--m-metadata-column body_site \  
--o-visualization core-metrics-results/bray_curtis_distance_matrix.qzv```
```


Let's explore these plots in QIIME2 View.

We are looking at what's called a "distance plot", and they can be kind of confusing. If you remember how beta diversity works, we are measuring a distance between two points (remember all of those metrics we made you calculate? That's where these numbers come from). When we make beta diversity box plots, we are plotting the distribution of distances between two things,  and the n is the number of distances in the box plot (not the number of samples you have for that category). 

Now that we understand these distance plots, let's try to interpret them.

**Does body site contribute to a significant difference in beta diversity?**
	 Yes
**After reviewing the PCoA plots again, which other metadata variable would you test with beta group significance (PERMANOVA)?**
	Scavenging activity
	Facility

---

### **Longitudinal Analysis with Qiime2**

There's a lot you can do:


![Screenshot 2024-02-29 at 6.51.08 PM.png](https://colostate.instructure.com/courses/220471/files/38493072/preview)

==VOLATILITY NOT A STAT PLOT==

Here, we will analyze the time-series data associated with this experiment. To use the **longitudinal** plugin, you need a repeated measures sample in which the same site or individual is sampled over time. In the decomp tutorial, **samples from each donor were collected over accumulated degree days**, so we can use this tool to **understand how the microbiome changed during these periods.** Much of these analyses can be done with alpha or beta diversity. For the sake of time, we will focus on just beta diversity. 

Let's open up our **unweighted UniFrac emperor plot** again to help visualize the analysis that we are about to do. **Color the samples by host_subject_id.** Click on the animations tab and select days add_0c for the gradient and **host_subject_id** as the trajectory. Then, click the play button and observe. You can adjust the speed to observe changes in beta diversity at a slower pace.

We can see visual changes in beta diversity over time, but this wouldn't be very useful for reporting in a manuscript. **Let's look at longitudinal differences with beta diversity so we can start associating numbers and p-values with what we see.**

We want to look at **how samples from an individual donor change along principal coordinates**. So, we will construct a **volatility plot** which will let us **track the beta diversity changes over time within an individual.** Volatility refers to the temporal stability of a metric over time and between subjects. 

This command is a little different than those we've used before because the parameters we are naming can actually be changed in the visualization. That's why they are called the "defaults". Also, note that we are using the output of the core metrics for this analysis.

# make a longitudinal directory  
  
mkdir longitudinal  
  
cd longitudinal  
  
# construct a volatility plot  
  
qiime longitudinal volatility \  
--m-metadata-file ../metadata/metadata.txt \  
--m-metadata-file ../core-metrics-results/weighted_unifrac_pcoa_results.qza \  
--p-state-column add_0c \  
--p-individual-id-column host_subject_id \  
--p-default-group-column 'sample_type' \  
--p-default-metric 'Axis 2' \  
--o-visualization pc_vol_sample_type.qzv

Let's explore this output in q2view. Using the control bars to the right, look at variation in sample_type and facility along PCs 1, 2, and 3. **What kind of patterns do you see with time along each axis?**

***Once you have an overview of each axis, you can do linear mixed effects models with the axis of interest as the response variable in R. I will usually test the first 3 axes in R for my variables of interest.** 

Another method is to do a **distance-based analysis using the first-distances method**. We'll use this to test the **hypothesis that sample type affects the magnitude of change in the distance from the first sample collected.** This baseline parameter is used to specify a static time point against which all other time points are compared - if we were to remove this, we would instead look at the rate of change for each individual between each time point. 

Here, the state column designates the time component in the metadata in this case, add_0c, and the individual id column is used to designate the host subject id sample type. 

Another method is to do a **distance-based analysis using the first-distances method**. In this analysis, we test the hypothesis that **microbial community composition changes over accumulated degree days (ADD)**, measured as the distance from each sample type’s baseline time point (ADD = 0). By specifying a baseline (`--p-baseline 0`), all subsequent time points are compared back to that fixed reference, allowing us to quantify how much each community diverges from its starting composition over time.

The `state` column (`add_0c`) represents accumulated degree days (time), and the `individual-id-column` (`host_subject_id_sample_type`) links repeated measures from the same sample type (soil or skin, facility) across time. If the baseline parameter were omitted, the analysis would instead evaluate change between successive time points (i.e., rate of change rather than deviation from the initial state).

# evaluate using first distances  
  
qiime longitudinal first-distances \  
--i-distance-matrix ../core-metrics-results/weighted_unifrac_distance_matrix.qza \  
--m-metadata-file ../metadata/metadata.txt \  
--p-state-column add_0c \  
--p-individual-id-column host_subject_id_sample_type \  
--p-baseline 0 \  
--o-first-distances from_first_wunifrac.qza

We can again use a volatility analysis to visualize the change in beta diversity based on distance.

# visualize volatility  
  
qiime longitudinal volatility \  
--m-metadata-file ../metadata/metadata.txt \  
--m-metadata-file from_first_wunifrac.qza \  
--p-state-column add_0c \  
--p-individual-id-column host_subject_id \  
--p-default-metric Distance \  
--p-default-group-column 'sample_type' \  
--o-visualization from_first_wunifrac_vol.qzv

**Looking at this plot, does one soil type change more over time than the other? What about by facility?**

**Note:** this analysis can also be performed on alpha diversity metrics like Shannon and observed features. From the command above, replace from_first_unifrac.qza file with an alpha diversity file (shannon.qza) and replace the "Distance" metric with "shannon".

Next, we can do a statistical test with a Linear Mixed Effects model. This will let us know if there is a relationship between a dependent variable (diversity) and one or more independent variables, such as accumulated degree days, sample type, facility, etc within the repeated measures. 

Since we are interested in the change in distance from the initial time point, we use Distance for --p-metric. 

# run LME   
  
qiime longitudinal linear-mixed-effects \  
--m-metadata-file ../metadata/metadata.txt \  
--m-metadata-file from_first_wunifrac.qza \  
--p-state-column add_0c \  
--p-individual-id-column host_subject_id \  
--p-formula "Distance ~ add_0c + facility + sample_type" \  
--o-visualization from_first_wunifrac_lme_formula.qzv

There is a new line here, --p-group-columns. While our main question centers around whether accumulated degree day and facility affects the longitudinal change in the microbial community, we also know that sample type plays a large role in shaping the microbial community as well. By including this line, we can account for **all** of these things "**to test whether microbial communities diverge from baseline over accumulated degree days, while controlling for variation due to facility and sample type."**

Let's look at this output in q2view now. **Here, is there a significant association between the sample type and temporal change? What about facility?**  If you need a review from a statistics class to understand what the model outputs mean, the table below may help, but [this videoLinks to an external site.](https://www.youtube.com/watch?v=NIGbYdGErLw&list=PLbVDKwGpb3XmvnTrU40zHRT7NZWWVNUpt&index=23 "(opens in a new window)") may also help (play from about 14 min to 20 min):

![Screen Shot 2022-03-08 at 2.37.10 PM.png](https://colostate.instructure.com/courses/220471/files/38493055/preview)

**When is it time to move your longitudinal analysis outside of Qiime2?**

Here are some R packages that are useful for downstream longitudinal analysis. 

- **Linear Mixed Effects Models (lme4)**
    - Fit linear and generalized linear mixed-effects models.
    - Can model interactions
    - More flexible than the Qiime2 plugin
- **Estimating Marginal Means (emmeans)**
    - Compare treatment groups at specific timepoints
        
    - Adjust for covariates like age
        
    - Interpret interactions (e.g., treatment × time)
        
    - Generate adjusted pairwise comparisons with multiple-testing correction
        
- **ANCOM-BC2**
    - Adjust for covariates (like age)
- **MaAsLin2**
    - Enables multivariable association testing with fixed and random effects

---

### **Exporting Qiime2 data** 

For downstream analysis outside of qiime2, it is helpful to export qiime2 data so it can be used with other data analysis tools such as R or Python. 

We export QIIME 2 data by unzipping the `.qza` file because a `.qza` artifact is essentially a compressed (zipped) directory. Inside this archive is a structured collection of files, including the actual analysis data (e.g., diversity metrics), as well as supporting files such as provenance information, metadata, version information, and checksums.

The provenance folder records the full analysis history of the artifact, allowing reproducibility and transparency of all upstream steps. The checksums file ensures data integrity by verifying that the contents have not been altered. The `data/` directory contains the primary output of interest (e.g., alpha diversity values or ordination results).

By unzipping the `.qza` file, we can directly access the underlying data files for use in downstream analyses outside of QIIME 2.

**To start, move out of the longitudinal directory.**

cd ../

We will export these alpha diversity metrics: **shannon_vector.qza, observed_features_vector.qza, faith_pd_vector.qza, and eveness_vector.qza.** 

**Create a new directory for the data we want to export**

mkdir export

**We will start with unzipping the shannon_vector.qza**

unzip core-metrics-results/shannon_vector.qza -d export/shannon

 After unzipping, you’ll notice a UUID-named folder inside `export/shannon/`. QIIME 2 stores each artifact inside a uniquely identified directory for provenance tracking. Within that folder, the `data/` directory contains the actual Shannon diversity values (`alpha-diversity.tsv`), which is the file we will use for downstream analysis.

**Let's explore the file structure of the unzipped directory**

**Navigate into the export directory and take a look around.**

cd export   
  
ls 

What do you see?

**Now navigate into the shannon directory and take a look at what is in the directory.**

cd shannon  
  
ls

What do you see? You should see a directory with a bunch of random characters; this is the UUID-named folder.

**Take a look at the data directory**

cd */data  
  
ls

**Now unzip the qza files for the other three alpha diversity metrics.**

Move back into the decomp_tutorial directory

cd ../../../../

Run the unzip commands for each diversity metric

# Observed Features  
unzip core-metrics-results/observed_features_vector.qza -d export/observed_features  
  
# Faith's PD  
unzip core-metrics-results/faith_pd_vector.qza -d export/faith_pd  
  
# Pielou's evenness  
unzip core-metrics-results/evenness_vector.qza -d export/evenness  

**Let's repeat what we did with the alpha diversity metrics with our beta diversity metrics.** 

# Bray Curtis  
unzip core-metrics-results/bray_curtis_pcoa_results.qza -d export/bray_curtis  
  
# Jaccard  
unzip core-metrics-results/jaccard_pcoa_results.qza -d export/jaccard  
  
# Unweighted Unifrac  
unzip core-metrics-results/unweighted_unifrac_pcoa_results.qza -d export/unweighted_unifrac  
  
# Weighted Unifrac  
unzip core-metrics-results/weighted_unifrac_pcoa_results.qza -d export/weighted_unifrac  

- Why do you think for beta diversity we used the pcoa results instead of the distance matrix?

**Let's clean this up a little bit. There are a lot of files, and you only need the .tsv for R.**

Navigate into the export directory

cd export

Make an alpha diversity directory

mkdir alpha_div

Copy the tsv files into the alpha_div directory

# define alpha metrics  
metrics=("shannon" "evenness" "faith_pd" "observed_features")  
  
# copy their tsv files into alpha_div/  
for metric in "${metrics[@]}"; do  
 cp $metric/*/data/alpha-diversity.tsv alpha_div/${metric}.tsv  
done

- Organizing the files this way puts all alpha diversity metrics in one clean directory with consistent filenames, making them easier to import into R for downstream statistical analysis and visualization.

**Now we will repeat the same steps with beta diversity.**

Make a beta diversity directory

mkdir beta_div

Copy the tsv files into the beta_div directory

# define beta metrics  
metrics=("bray_curtis" "jaccard" "unweighted_unifrac" "weighted_unifrac")  
  
# copy their txt files into beta_div/  
for metric in "${metrics[@]}"; do  
 cp $metric/*/data/ordination.txt beta_div/${metric}.txt  
done

Now, if you want to download all of these metrics at once to your local computer, go to OnDemand and select the `export` directory. From there, you can download all of the unzipped files at once. Alternatively, you can navigate into the `export` directory and download only the `alpha_div` or `beta_div` folders, which contain just the TSV files.

We will use these files next week in R. 

---

### Homework #3

- see Homework 3 materials in Modules
- due Thursday at midnight

[](https://colostate.instructure.com/courses/220471/modules/items/7738507)

[](https://colostate.instructure.com/courses/220471/modules/items/7742653)