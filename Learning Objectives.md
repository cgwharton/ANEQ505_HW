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

- Describe the major bacterial and microbial eukaryotic taxa in the mammalian gut

- Describe what would happen if you didn’t demultiplex your sequencing run

- Understand the difference between clustering and denoising

- Understand the difference between ASVs and OTUs

- Describe the file types you have after denoising

- Describe how taxonomy is useful

- Understand how taxonomy is assigned to microbiome sequence data

- Give an example of how taxonomy can be used for quality control of data

- Understand the basics of phylogenetic trees, and why they are useful

Below is a **quiz-oriented walkthrough** of each learning objective. I’ll keep this conceptual (not protocol-heavy), flag common pitfalls, and highlight what instructors typically want you to say.

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

