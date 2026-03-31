---
marp: true
theme: dsi_certificates_theme
paginate: true
---


# Post-GWAS Analyses I

```
$ echo "Data Sciences Institute"
```
------

# What You Will Learn Today

- **What GWAS teaches us**: polygenicity, effect sizes, and cross-ancestry considerations.
- **Post-GWAS fine-mapping basics**: Regular approaches and Bayesian approaches.
- **Functional annotation primer**: ENCODE, Roadmap, GTEx and molecular QTLs.
-------

# What have we learned so far from GWAS?

<!--
By performing GWAS studies, scientists have successfully identified the
association of hundreds of thousands to millions of SNPs to a single
phenotype. 


Example of GWAS catalog /extracted from lei's slides
-->

- Large-scale GWAS have mapped hundreds of thousands to millions of SNP-phenotype associations.
- Example (2021): The GWAS Catalog listed 4,865 publications and 247,051 associations.
  - These reported SNPs help illuminate molecular mechanisms of common diseases and the biological pathways underlying traits of interest.
- Associations for some SNPs linked to rare diseases have been tested intensively.
- Yet classic GWAS alone have not yielded solid, mechanistic insight into how variants drive phenotypes.

-------

# GWAS SNP-Trait Discovery Timeline



  ![Sales Figure, w:850](./images/gwas_timeline.png)


-------

# Practical Lessons from GWAS

- Complex traits are highly polygenic, with thousands of variants contributing small increments of risk or trait change.

- For common variants, single-variant effect sizes are typically modest; odds ratios frequently fall in the 1.05-1.20 range.

- For height, a well-powered model trait, the effect sizes are on the order of $\sim 1$ millimeter.

- Because effects are small and testing is genome-wide, large cohorts are needed to reach stringent significance (e.g., $\mathrm{p}<5 \times 10^{-8}$ ).


--------

# Practical Lessons from GWAS

- A GWAS peak typically marks a cluster of nearby variants (about 10–100 kb) that move together because of linkage disequilibrium.

- Multiple independent association signals can reside within the same locus.

- The majority of GWAS associations lie outside protein-coding exons; these variants are believed to act mainly by regulating gene expression (e.g., enhancer or promoter activity) rather than by changing amino-acid sequence.

- Allele frequencies, LD structure, and effect sizes at disease loci can vary across ancestries.

- Pleiotropy (the same variant or locus is associated with multiple traits) is ubiquitous.

---------

# Practical Lessons from GWAS

- Many individual GWAS are underpowered to detect the smallest effects; nonetheless, the large number of contributing variants means genuine signals still emerge.

- Even when a variant’s effect on a biomarker is small, the clinical impact can be substantial: the implicated gene may encode a tractable drug target, and modulating it can yield large therapeutic benefits.

- For example, common variants near HMGCR only have a small influence on LDL-cholesterol, but drugs targeting the encoded protein reduce LDL by ~30\%.


--------

# From GWAS discovery to Medicines

- The overarching aim of human genetics is to enable translational medicine-turning genetic insights into better diagnostics, prevention, and therapies.

- Genetically supported targets are more likely to progress successfully through clinical development, including to phase III trials and eventual approval.

- E.g. People with loss-of-function mutations in SLC30A8 (the ZnT-8 transporter) have a lower risk of type 2 diabetes, leading to companies to develop ZnT-8 antagonists for diabetes therapies.

-----

# Success Stories: From GWAS to Clinical Impact


  ![Sales Figure, w:700](./images/gwas_drug.png)


------


# Post-GWAS Analyses


------

# Motivation

- pGWAS (post-GWAS) is a crucial step to move beyond SNP-level associations toward biological mechanism.

- Beyond the basic task of identifying genetic associations, several post-GWAS analyses can be performed:
  1. Fine-mapping: statistical approach to identifying causal variants.
  2. CRISPR experiments: experimental techniques
  3. Polygenic Risk Score (PRS): predicting trait values based on genotype profiles.
  4. Others: colocalization, TWAS, network inference, G×G and G×E analyses, etc.

-----------

# Confounding in GWAS -LD

- One major issue is confounding caused by local correlation among sites (linkage disequilibrium), which makes it difficult to distinguish true signal from variants that are merely correlated.

  ![Sales Figure, w:700](./images/finemap.png)

---------
<!--
# Fine-mapping

- Goal: identify causal variants and estimate the number of putative causal variants per locus.
- A central step in establishing causality is to eliminate confounding arising from linkage disequilibrium (LD).
  ![bg right:50% w:600](./images/finemap_example1.png)
----------
-->

# What is fine-mapping?

- GWAS often identifies a broad locus with many associated SNPs.
- Due to **linkage disequilibrium (LD)**, many SNPs in the locus are correlated
  and show similar p-values.
- Fine-mapping asks:
  - Which SNP(s) in this region are most likely to be **causal**?
  - How many **independent signals** are there?
- Goal: identify specific variants that are the best causal candidates.
- This is a key step before functional follow-up and experimental validation.

----------

# Heuristic fine-mapping: LD-based candidate selection

- Idea: use the **LD pattern around the lead SNP** to pick nearby SNPs
  that are likely to be causal.

- **LD thresholding:**
  - Compute pairwise LD ($r^2$) between the lead SNP and other SNPs.
  - Keep SNPs with LD above a threshold (e.g. $r^2 > 0.6$) as **candidate causal SNPs**.

<!--
- **LD clustering :**
  - Hierarchical clustering of all SNPs in a region based on their pairwise $r^2$ to create clusters.
--> 
---------  
  
# Heuristic Fine-mapping: LD-based Candidate Selection

- Visualization tools such as **[LocusZoom](http://locuszoom.org/)** or **[Haploview](https://www.broadinstitute.org/haploview/haploview)**:
    - Combining the GWAS lead SNP with SNPs in the same LD block to select potential causal SNPs. 

  ![bg right:50% w:500](./images/locuszoom.png)

---

# Heuristic Fine-mapping: LD-based Candidate Selection

- Heuristic LD-based methods are useful for **initial candidate selection**, but not sufficient on their own to define causal variants.
- **Limitations:**
  - Relies on **arbitrary thresholds** for LD and window size.
  - Does **not** model the **joint effects** of multiple SNPs on the trait.
  - Does **not** provide an objective measure (e.g. probability) that a SNP
    is causal—interpretation is partly **subjective**.
  - More rigorous approaches (penalized regression, Bayesian fine-mapping)
    explicitly model multiple SNPs together and quantify uncertainty.

---------

# Heuristic Fine-mapping: Conditional Analysis

- Start from the **lead SNP** in a locus (smallest p-value).
- Use **conditional analysis / forward stepwise regression**:
  - Fit a regression model (linear or logistic) with the current set of SNPs
    in the locus (initially just the lead SNP).
  - For each remaining SNP, test its effect **conditional on** the SNPs
    already in the model (add one candidate SNP at a time).
  - Add the SNP with the **smallest conditional p-value** if it is below a
    pre-specified threshold (e.g. $5 \times 10^{-8}$ or a locus-specific threshold).
  - Repeat these steps until **no remaining SNP** has a significant
    conditional p-value → the SNPs in the final model are treated as
    **independent association signals** in the locus.

------

# Heuristic Fine-mapping: Conditional Analysis

- Implemented in **PLINK2** :
   - e.g. `--condition`, `--condition-list`  
  ![bg right:50% w:600](./images/conditional_analysis.png)
  
  
--------

 # *Bayesian Models for Fine-mapping
 
- Bayesian fine-mapping provides an alternative framework to identify causal variants at loci discovered in GWAS.
- These methods are typically applied locally, analyzing one genomic locus at a time.
- Causal variants are prioritized using the **Posterior Inclusion Probability (PIP)**:
  $$
  P(~\text{SNP i is causal}~∣Z)
  $$
- PIP quantifies the probability that a SNP is truly causal given the observed data.
- SNPs with higher PIP are stronger candidates for causality.
----

# *Bayesian Fine-mapping Schematic


   ![Sales Figure, w:800](./images/finemap_workflow.png)
   
-----
# *Bayesian Fine-mapping

- Input: **GWAS summary statistics (Z-scores) and LD information** at GWAS locus.

- Smart search strategies are used to efficiently find likely causal SNPs
  - Shotgun Stochastic Search (SSS) or Iterative Bayesian Stepwise Selection (IBSS)
  
- Output: **Posterior inclusion probabilities (PIPs)** for each SNP.
- Output also includes **credible sets**: groups of genetic variants that are likely to contain the true causal variant(s) with a specified probability.

- A level $\rho$ credible set is defined to be the smallest subset of correlated variants (with correlation > $r$ ) that has probability $\rho$ or greater of containing at least one causal SNP variant.

------

# *Fine-mapping Example 1


   ![Sales Figure, w:800](./images/CARMA1.png)


------

# *Fine-mapping Example 2

   ![Sales Figure, w:1000](./images/CARMA2.png)

------

# *Practical Tools

- Widely used Bayesian fine-mapping methods: CAVIAR (2014), FINEMAP (2016), SuSiE (2019), CARMA (2023).
- Tutorial (SuSiE in R): https://stephenslab.github.io/susieR/articles/finemapping_summary_statistics.html 


-------

<!--

# Fine-mapping: Penalized Regression Models

- **Jointly model many SNPs** in a region using regression.
- Let $Y$ be the phenotype, $X$ the genotype matrix, and $\beta$ the SNP effects.
- Penalized regression estimates $\beta$ while shrinking small effects towards zero:
  - Examples: **lasso**, **elastic net**, other sparse penalties.
- Objective (elastic net form):
  $$
  \min_{\beta}\; \frac{1}{2n}\|Y - X\beta\|^2
  + \lambda\big(\alpha\|\beta\|_1 + (1-\alpha)\|\beta\|_2^2\big).
  $$
- Result: a sparse model where only a few SNPs have non-zero effects → candidate causal SNPs.

---

# Fine-mapping: Penalized Regression Models

- Works best with **individual-level data** and many correlated SNPs.
- **Tuning parameter** $\lambda$ (and $\alpha$) chosen by cross-validation to minimize prediction error.
- Advantages over forward selection:
  - More stable when SNPs are highly correlated.
  - Simultaneously estimates effect sizes and performs variable selection.
- Limitations:
  - Aims to choose a good model for $Y$, not to quantify causal probabilities.
  - This motivates Bayesian variable selection / Bayesian fine-mapping.

---


# Ingredients of Bayesian inference

- We have an unknown quantity $\theta$:
  - e.g. effect size, or an indicator “SNP $j$ is causal”.
- **Prior** $p(\theta)$:
  - Our belief about $\theta$ *before* seeing the data.
- **Likelihood** $p(\text{data} \mid \theta)$:
  - How likely the observed data are, if $\theta$ had a given value.
- **Posterior** $p(\theta \mid \text{data})$:
  - Our updated belief about $\theta$ *after* seeing the data.
- Bayes’ rule:
  $$
  p(\theta \mid \text{data}) =
  \frac{p(\text{data} \mid \theta)\, p(\theta)}{p(\text{data})}
  \;\propto\; p(\text{data} \mid \theta)\, p(\theta).
  $$

-----

# Bayesian Fine-mapping

- Our goal: decide **which SNPs have non-zero effects**.

- For $m$ SNPs, define an indicator vector $C = (c_1,\dots,c_m)$:
  - $c_j = 1$ if SNP $j$ is causal, $c_j = 0$ otherwise.
  - There are $2^m$ possible $c$ vectors → $2^m$ possible causal models.

- We specify a **prior** over which SNPs are causal (e.g. all equally likely, or a fixed expected number per region).
  
- Using Bayes' formula, for a specified model $C = c$ :

$$
P(C=c \mid D)=\frac{P(D \mid C=c) \cdot P(C=c)}{P(D)}
$$

- $P(D \mid C=c): (Z_1,Z_2,...Z_m) \sim N(0,f(\Sigma,C))$. 

------

# Posterior Inclusion Probability (PIP)

- The posterior probabilities for different models can be used to determine the posterior probability of including SNP $j$ as causal.

- PIP for SNP $i$: sum of posteriors over all models that include SNP $i$ as causal.

$$P I P_i=\sum_c I(c_i =1 ) P(C=c \mid D)$$

- Rank SNPs by PIP to prioritize likely causal variants
  - Higher PIP $\rightarrow$ stronger evidence of causality


--------
  
# Bayesian Fine-mapping

- Input: **GWAS summary statistics (Z-scores) and LD information** at GWAS locus.

- Efficient search algorithms are used to explore the space of causal configurations
  - Shotgun Stochastic Search (SSS) or Iterative Bayesian Stepwise Selection (IBSS)
  
- Output: **Posterior inclusion probabilities (PIPs)** for each SNP.
- Output also includes **credible sets**: groups of genetic variants that are likely to contain the true causal variant(s) with a specified probability.

- A level $\rho$ credible set is defined to be the smallest subset of correlated variants (with correlation > $r$ ) that has probability $\rho$ or greater of containing at least one causal SNP variant.

------  

# Practical Workflow \& Tools

- Steps:
  - Define regions (around lead SNPs; PLINK --clump for sentinel signals).
  - Run Bayesian fine-mapping to obtain PIPs \& credible sets.
- Tools: CAVIAR, FINEMAP, SuSiE, CARMA
- Caution in high-LD regions: probability spreads across correlated SNPs.
- Posterior expected number of causal SNPs $\approx \Sigma PIP_{i}$ over the region.

--------

# Discussion

- Answer: Not necessarily. 
- Ties or near-ties in PIPs can yield multiple valid sets; software typically reports one (often the smallest) based on its ranking rules.
  
  ![Sales Figure, w:750](./images/susie.png)

-------

# Functional Annotation in Fine-Mapping

- Bayesian models can incorporate additional knowledge (e.g. functional annotation) in terms of prior to help disentangle highly correlated variables.

  ![Sales Figure, w:800](./images/finemap_example2.png)


-----

# Functional Annotation in Fine-Mapping

  ![Sales Figure, w:800](./images/carma.png)
  
------


# Hypothetical Examples

- The purple bars represent additional variant-level statistics produced by fine-mapping.
   - β-values for penalized regression
   - PIPs for Bayesian methods 

- The light grey boxes represent the regions selected by fine-mapping.

![bg right:35% w:300](./images/fine_map_all.png)

-------

# More Complex Issues

- Many GWAS results come from meta-analyses without individual-level data.

- Sample size varies by SNP in the meta-analysis, leading to inconsistencies.

- LD information is often taken from external reference panels.

- Mismatches between external LD and GWAS summary stats can invalidate fine-mapping.

- Leverage high-dimensional functional annotations to improve inference.

---------

- ##### We’ll dive deeper into fine-mapping strategies for these scenarios in the Advanced Computational Genomics course!

---------

# Fine-mapping methods: summary

- **Heuristic LD-based methods**
  - Use LD thresholds and visual inspection to pick SNPs near lead SNPs.
  - Fast and intuitive, but arbitrary and non-probabilistic.

- **Penalized regression**
  - Jointly models many SNPs, encourages sparse solutions.
  - Better than simple forward selection in high-LD regions.

- **Bayesian fine-mapping**
  - Models uncertainty over many possible causal configurations.
  - Produces PIPs and credible sets → probabilistic interpretation.

--------
-->

# Functional Follow-up

- Fine-mapping prioritizes putative causal variants at GWAS loci that can be subjected to functional studies.
- Massively parallel CRISPR perturbation of GWAS loci:

  ![Sales Figure, w:700](./images/finemap_crisper.png)

-------

# Functional Annotation

-------


# Functional Annotation of variants

- GWAS pinpoints where associations occur, but not how the implicated variants exert their effects.
- Functional annotation adds biological context to a variant to infer its potential impact on genes, regulatory programs, and molecular traits.
- Use diverse resources to answer specific questions:
  - Does the variant alter amino-acid sequence or function? (e.g. PolyPhen-2)
  - Does the variant fall within an enhancer, promoter, or other regulatory element? (ENCODE, Roadmap Epigenomics)
  - Is it in a conserved region? (evolutionary constraint)
  - Does it affect a molecular trait like gene expression or protein level? (eQTL / pQTL)
  
<!--
Notes:
Before we get into the specific datasets, I want to make sure we’re all on the same page about what functional annotation means. When we do GWAS or when we sequence people, we get a list of variants. That tells us where the variation is in the genome, but it doesn’t tell us what that variant actually does.

Functional annotation is the step where we add biological information back onto each variant to guess or infer its impact. In other words, we’re enriching a bare variant call with evidence from biology.

So what kinds of questions do we ask? First, does this variant change the protein? For coding variants we can use tools like PolyPhen-2 that tell us whether an amino-acid change is likely damaging.
Second, does the variant fall in a regulatory element? For noncoding variants we look at ENCODE or Roadmap epigenomics to see whether that position is an enhancer or promoter in some cell type.
Third, is the position evolutionarily conserved? If that base is preserved across many species, that’s a hint that changing it might matter.
And finally, does the variant actually change a molecular trait in people? That’s where things like eQTLs or pQTLs come in — they tell us that different genotypes at this site lead to different expression or protein levels.

The overall goal of doing all this is not just to decorate the variant — it’s to prioritize which variants are worth following up in experiments later.
-->

--------

# ENCODE

The Encyclopedia of DNA Elements (ENCODE) is a public research project that aims to build a comprehensive parts list of functional elements in the human genome.
    ![bg right:60% w:700](./images/encode.png)


--------

# Roadmap Epigenomics

- "The NIH Roadmap Epigenomics Mapping Consortium was launched with the goal of producing a public resource of human epigenomic data to catalyze basic biology and disease-oriented research."
- Coverage: 127 tissues/cell types (Roadmap and ENCODE) with coordinated measurements of histone marks, DNA methylation, open chromatin, and TF binding.

-------

# Roadmap Epigenomics

   ![Sales Figure, w:950](./images/roadmap.png)

--------

# Molecular QTLs 


- Using molecular QTLs together lets us build a causal chain from variant → molecular effect → phenotype.

  ![Sales Figure, w:700](./images/QTLs.png)

-------

# GTEx

- Genotype-Tissue Expression project (GTEx) links genotype to expression across tissues.
- Collected DNA and RNA from many human donors, multiple tissues per person.
- For each variant: test whether different genotypes show different gene-expression levels in a tissue.
  
  ![bg right:40% w:400](./images/gtex.png)

---------

# What's Next


- How to integrate GWAS with eQTL and other functional data (colocalization frameworks).
- Web tools to visualize GWAS + eQTL signals and perform practical colocalization analysis.

  
### What questions do you have about anything from today?

