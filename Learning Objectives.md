- ## Explain basic concepts and general trends in microbiome science
	- Microbiomes = **communities of microorganisms** (bacteria, archaea, fungi, protists, viruses) living in a habitat.
	- Microbiomes affect **host health, nutrient cycling, immunity, metabolism, and disease**
	- Microbial communities are:
		- **Highly diverse**
		- **Context-dependent** (host, diet, environment, geography)
		- **Dynamic** over time
	- Trend: shift from _“who is there?”_ → _“what are they doing?”_ (functional inference, multi-omics).
- ## Describe pros and cons of different microbiome data types

- ## Understand what makes a good amplicon marker

- ## Describe how 16S rRNA gene amplicon data are generated

- ## Describe two reasons why a microbial community may or may not show biogeographic patterns

- ## Describe two forces shaping the mammalian gut microbiome

- ## Describe the major bacterial and microbial eukaryotic taxa in the mammalian gut

- ## Describe what would happen if you didn’t demultiplex your sequencing run

- ## Understand the difference between clustering and denoising

- ## Understand the difference between ASVs and OTUs

- ## Describe the file types you have after denoising

- ## Describe how taxonomy is useful

- ## Understand how taxonomy is assigned to microbiome sequence data

- ## Give an example of how taxonomy can be used for quality control of data

- ## Understand the basics of phylogenetic trees, and why they are useful

- ## Understand common confounders in microbiome studies
	- Batch effects: Different extraction kits, operators, PCR runs, sequencing lanes, or reagent lots create systematic differences unrelated to biology; they can dominate multivariate structure if not blocked or adjusted.
	
	- Sample handling: Variable storage time, freeze–thaw cycles, and temperature during transport alter community composition, especially in low‑biomass or oxygen‑sensitive communities.
	
	- Host/background DNA: High host:microbe DNA ratios (e.g., blood, tissue, placenta) cause mis‑annotation of host reads and stochastic detection of rare taxa.
	
	- Low biomass and stochasticity: In low‑biomass sites, tiny absolute differences in contamination or sampling depth create large relative changes, reducing power and inflating apparent between‑sample differences.
	
	- Bioinformatic choices: Truncation lengths, chimera removal, clustering vs ASVs, and reference database all influence which taxa appear and their relative abundance, affecting both alpha and beta diversity.

- ## Understand common contamination types and sources
	For typical 16S/shotgun workflows, contamination often comes from:
	- Reagents and kits: DNA extraction kits, PCR master mixes, and water carry characteristic “kitome” taxa (e.g., skin and environmental bacteria).
	- Environment and handling: Airborne microbes, dust, lab surfaces, and operator skin introduce low‑level contaminants during extraction and PCR setup.
	- Cross‑well/cross‑sample contamination: Splashing, aerosols, or index hopping cause reads to bleed across wells or libraries, especially with high‑biomass neighbors.
	- Carry‑over from previous runs: Residual amplicons or libraries on instruments or in pipettes can appear in new runs.
	Conceptually, contaminant profiles are often: (1) similar across negative controls; (2) relatively constant in absolute abundance; and thus (3) relatively more prominent in low‑biomass samples than in high‑biomass ones.

- ## Understand the use of negative and positive controls in sequencing and how to utilize them
	- Negative controls
		- Typical negative controls include extraction blanks and PCR no‑template controls.
		- Use them to:
			- Characterize lab‑specific contaminant signatures: Taxa consistently enriched in blanks indicate reagent/environmental contaminants for that batch.
			- Guide contaminant removal: Approaches include removing ASVs/OTUs that are significantly associated with low total DNA, that are enriched in negatives vs true samples, or that track batch but not biology.
			- Assess batch‑specific issues: If some batches show many reads or unexpected taxa in blanks, treat that batch with extra caution or exclude it.
			- Limitations: Negative controls alone miss contaminants that only appear when real DNA is present, or that occur stochastically in a subset of samples. This is why relying solely on public contaminant lists is discouraged relative to using internal controls.
	- Positive controls (mock communities, spike‑ins)
		- Positive controls include defined mock communities and dilution series.
		- Use them to:
			- Check taxonomic accuracy and bias: Compare observed vs expected composition to assess extraction bias (e.g., gram‑positive under‑representation), primer bias, and pipeline performance.
			- Monitor sensitivity and dynamic range: Dilution series indicate where low biomass begins to converge toward contaminant profiles and where diversity metrics become unreliable.
			- Validate contaminant‑removal strategies: After applying contaminant filtering, the mock community should become more similar to its expected profile (improved recovery, fewer spurious taxa).
		- In practice, ideal sequencing runs include: sample‑matched extraction blanks, PCR negatives, at least one mock community, and, if you care about limits of detection, a mock dilution series.

- ## Describe the difference between alpha and beta diversity
	- Alpha diversity: Within‑sample complexity—how many taxa (richness) and how evenly they are distributed (evenness) in a single sample.
	
		- Examples: Observed richness, Chao1 (richness estimate), Shannon index (richness + evenness), Simpson index (dominance/evenness).
	
	- Beta diversity: Between‑sample differences—how similar or dissimilar community composition is across samples or groups.
	
	- Applied example: Alpha diversity can assess whether treatment reduces within‑host richness, while beta diversity tests whether the overall community composition shifts between treatment and control.

- ## Understand the different beta diversity metrics and how they are useful
	- Beta diversity is computed as a distance or dissimilarity matrix; different metrics emphasize different features.
	
	- Common metrics:
		- Jaccard (presence/absence): Compares which taxa are present, ignoring abundances; useful when absolute quantitation is unreliable or when rare taxa are of primary interest.
		
		- Bray–Curtis (abundance‑weighted): Uses counts or relative abundances; sensitive to dominant taxa and compositional differences in abundant members.
		
		- UniFrac (phylogenetic): Measures the fraction of branch length unique to each sample on a phylogenetic tree.
		
		- Unweighted UniFrac: Presence/absence along the tree, sensitive to rare, phylogenetically distinct taxa.
		
		- Weighted UniFrac: Abundance‑weighted, dominated by common lineages; can show low dissimilarity if communities share abundant, closely related taxa even with many rare differences.
		
		- Aitchison distance: Euclidean distance in centered log‑ratio (CLR) space, designed for compositional data; often preferred for methods that directly model relative abundances.​
	
	- How to choose:
		- Presence/absence focus or heterogeneous sequencing depth → Jaccard or unweighted UniFrac.
		
		- Abundance changes in dominant taxa (e.g., probiotics, dysbiosis) → Bray–Curtis or weighted UniFrac.
		
		- Strong phylogenetic signal or interest in evolutionary relationships → UniFrac variants.
		
		- Compositional modeling and integration with multivariate methods → Aitchison (CLR)‑based distances.

- ## Understand different beta diversity statistical test options
	- Tests operate on the distance matrix to assess group differences or covariate effects.
	
	- PERMANOVA (e.g., vegan::adonis)
		- Non‑parametric ANOVA‑like test on distances; evaluates whether group centroids in multivariate space differ.
		
		- Supports complex designs (multiple covariates, interactions, stratified permutations).
		
		- Common for testing treatment, time, or exposure effects on overall microbiome composition.​
		
		- Sensitive to differences in dispersion: if group variances differ, you can get significant results driven by dispersion rather than location, so pair with dispersion tests (e.g., PERMDISP).
	
	- ANOSIM
		- Ranks distances and compares within‑ vs between‑group similarity, outputting an R statistic (0 = no separation, 1 = complete separation).
		
		- Conceptually simpler but often less robust and more sensitive to dispersion and unbalanced designs than PERMANOVA.
	
	- Additional multivariate options
		- Distance‑based redundancy analysis (db‑RDA) and constrained ordination: Partition variance in beta diversity explained by environmental/clinical covariates.
		
		- Methods like FFMANOVA and ASCA: Treat the full abundance matrix in a multivariate ANOVA framework, sometimes offering better power in certain “few taxa vs many taxa” scenarios.​
		
		- In practice, a typical workflow is: compute one or more ecologically appropriate beta‑diversity metrics; visualize with ordination (PCoA, NMDS); test group effects with PERMANOVA (with stratified permutations if blocking); check homogeneity of dispersion; and interpret in light of controls and potential confounders.

---

## 1. Basic concepts and general trends in microbiome science

**Core ideas**

- Microbiomes = **communities of microorganisms** (bacteria, archaea, fungi, protists, viruses) living in a habitat.
    
- Microbiomes affect **host health, nutrient cycling, immunity, metabolism, and disease**.
    
- Microbial communities are:
    
    - **Highly diverse**
        
    - **Context-dependent** (host, diet, environment, geography)
        
    - **Dynamic** over time
        
- Trend: shift from _“who is there?”_ → _“what are they doing?”_ (functional inference, multi-omics).
    

---

## 2. Pros and cons of different microbiome data types

You don’t need exhaustive lists—know the **tradeoffs**.

### 16S rRNA gene amplicon sequencing

**Pros**

- Cheap, scalable
    
- Good for **community composition**
    
- Well-established pipelines
    

**Cons**

- Limited taxonomic resolution (often genus level)
    
- No direct functional information
    
- PCR bias
    

### Shotgun metagenomics

**Pros**

- Species/strain-level resolution
    
- Functional genes detected
    
- No primer bias
    

**Cons**

- Expensive
    
- Computationally intensive
    
- Host DNA contamination
    

### Metatranscriptomics / metabolomics (high level)

- Show **activity**, not just presence
    
- Harder to interpret, expensive, sensitive to degradation
    

---

## 3. What makes a good amplicon marker

A good marker gene:

- Is **present in all target organisms**
    
- Has **conserved regions** (for primer binding)
    
- Has **variable regions** (to distinguish taxa)
    
- Evolves at an appropriate rate
    
- Has a **good reference database**
    

👉 16S rRNA works well for bacteria because it meets all of these.

---

## 4. How 16S rRNA gene amplicon data are generated

High-level workflow:

1. Extract DNA
    
2. PCR amplify a **specific 16S region** (e.g., V4)
    
3. Add **barcodes/indexes**
    
4. Sequence (e.g., Illumina)
    
5. Bioinformatics processing (quality filtering, denoising)
    

Key point: this is **targeted sequencing**, not whole genomes.

---

## 5. Why a microbial community may or may not show biogeographic patterns

Two major reasons:

1. **Dispersal limitation**
    
    - Microbes can’t reach all locations equally
        
2. **Environmental selection**
    
    - Different conditions select for different taxa
        

Why patterns _might not_ appear:

- High dispersal ability
    
- Strong host filtering overriding geography
    

---

## 6. Two forces shaping the mammalian gut microbiome

Common correct answers:

- **Diet** (one of the strongest drivers)
    
- **Host genetics / physiology**
    
- Immune system
    
- Antibiotic exposure
    
- Age and development
    

If unsure, say **diet + host selection**.

---

## 7. Major bacterial and microbial eukaryotic taxa in the mammalian gut

### Bacteria (know these)

- **Firmicutes**
    
- **Bacteroidota (Bacteroidetes)**
    
- **Actinobacteriota**
    
- **Proteobacteria**
    

### Microbial eukaryotes

- **Fungi** (e.g., _Candida_, _Saccharomyces_)
    
- **Protists** (varies by host/environment)
    
- **Helminths** (sometimes discussed as part of gut ecosystem)
    

---

## 8. What happens if you didn’t demultiplex your sequencing run

Demultiplexing = assigning reads to samples using barcodes.

If you don’t:

- All reads are **mixed together**
    
- You **lose sample identity**
    
- Data become **biologically meaningless**
    
- Cannot compare samples
    

This is usually framed as a **fatal error**.

---

## 9. Difference between clustering and denoising

### Clustering

- Groups sequences by similarity (e.g., 97%)
    
- Produces **OTUs**
    
- Inflates diversity
    
- Sensitive to errors
    

### Denoising

- Models sequencing error
    
- Produces exact sequences
    
- Higher resolution
    
- Produces **ASVs**
    

---

## 10. Difference between ASVs and OTUs

- **OTUs** = similarity-based clusters
    
- **ASVs** = exact biological sequences inferred after error correction
    

Key phrase:

> _ASVs are reproducible across studies; OTUs are not._

---

## 11. File types after denoising

Typically:

- **Feature table** (samples × ASVs)
    
- **Representative sequences** (FASTA)
    
- **Metadata file** (separate, but essential)
    
- Sometimes a **phylogenetic tree**
    

Know: abundance table + sequences.

---

## 12. How taxonomy is useful

Taxonomy allows you to:

- Interpret biological meaning
    
- Compare studies
    
- Identify contaminants
    
- Link taxa to known functions or hosts
    

---

## 13. How taxonomy is assigned

Common methods:

- **Naive Bayes classifiers**
    
- Alignment to reference databases (SILVA, Greengenes, RDP)
    

Conceptually:

> Compare your sequences to **known reference sequences** and assign the closest match.

---

## 14. Taxonomy for quality control (example)

Classic examples:

- Detecting **chloroplast or mitochondrial sequences** in 16S data
    
- Unexpected taxa (e.g., marine bacteria in gut samples)
    
- Human skin microbes dominating soil samples
    

This signals **contamination or primer issues**.

---

## 15. Basics of phylogenetic trees and why they’re useful

**Phylogenetic trees**

- Show **evolutionary relationships**
    
- Branch length ≈ evolutionary distance
    

Why useful:

- Used in **phylogenetic diversity metrics**
    
- Improve community comparisons (e.g., UniFrac)
    
- Capture relationships beyond taxonomy labels
    

Key idea:

> They incorporate **relatedness**, not just presence/absence.





- **Focus:** Microbiome studies focus on the genetic potential and interactions of microbial communities within a specific habitat (e.g., gut microbiome). Microbiology covers a broad range of topics, including virology, bacteriology, and immunology, covering both beneficial and harmful microorganisms.

16s rrna mitochondria is considered contamination