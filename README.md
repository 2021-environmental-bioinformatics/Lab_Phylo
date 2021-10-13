### Lab_Phylo
For today's lab, we're going to be using IQ-TREE (http://www.iqtree.org/doc/Tutorial), a popular maximum-likelihood program for tree-building. I like IQ-TREE because it's pretty user-friendly; the documentation (particularly on the website, but also in the help commands) is exceptionally clear, and the syntax isn't super confusing (compared to some tree programs that will remain nameless...). It's also more than powerful enough for most analyses. Let's install: 

```
conda create -n trees
conda activate tree
conda install -c bioconda iqtree
```


The dataset we're using is adapted and subsampled from Lartillot et al. 2007 (https://doi.org/10.1186/1471-2148-7-S1-S4), and consists of 10 Nematodes, 13 Arthropods, 6 deuterostomes, and 10 fungi. Each species has 3 genes that were concatenated together into one amino acid alignment (39 taxa, 611 characters)---the original paper used 37 species, 146 genes, and 35,371 characters. **This is a particularly troublesome dataset!** The taxon sampling is designed to exacerbate long-branch attracion (LBA). Nematodes are (notoriously) weird and on a long branch all by themselves. In this case, since fungi are so distantly related to all the animals, they are also on a long branch. Thus, LBA pulls nematodes towards fungi at the base of the tree, and outside of their correct placement sister to the arthropods.

A quick note on rooting: when you infer a tree, it is usually **unrooted**, meaning it doesn't have any direction. You can't tell which direction time flows without external information; this is because most phylogenetic models are time-reversible. This is why we use **outgroups**, which are groups of sequences that we know are sister to all other sequences in our tree. We root the tree between the outgroup and ingroup. In this case, Fungi is the outgroup, but it's a very distant outgroup (hence the LBA). A better choice in this case would be choanoflagellates. 

### What goes into a phylogenetic model?

## Substitution matrices
-POISSON is the simplest amino acid substituion model, with equal frequences between all pairs of amino acids.
-JTT, WAG, and LG are all common general substition models. These are based on empirical observations from large numbers of closely-related proteins, and are what most people use most of the time.
-There are other special matrices for mitochondria, viruses, etc.

# Base (equilibrium) frequencies
Some of the 20 amino acids are more common than others. For example, tryptophan only shows up about 1% of the time.

-F: estimate empirical amino acid frequencies
  By default, amino acid frequencies are given by the substitution model. The -F parameter estimates these frequencies driectly from your data. 
  
# Rate heterogeneity
-Some sites (e.g. in immune proteins) can evolve very quickly. Other sites might be under strong selection and evolve very slowly. Fast-evolving sites convey less information, because they accrue more hidden substitutions! This is often modeled by grouping sites into "rate categories." 

-G4: there are 4 rate categories drawn from a gamma distribution. The number is user-specific, but 4 is common. 
-R4: there are 4 free rate categories (not constrained to a distribution; better fit but slower). 
  
-I: there is some proportion of invariant sites that never change
  -these are removed from the analysis prior to calculating the rate categories.
  
 So, you could specify a model by typing -m JTT+F+R6, for example. 
  
# Support values
We would like to know how confident we should be in the relationships shown in our tree: the data might support some splits very well, and others not so well. In maximum-likelihood analyses, this is usually done with **bootstrapping**. To bootstrap, you randomly subset your data and calculate a new tree using the subsample. For each split in your original best tree, the proportion of bootstraps that recover that relationship is called the "bootrap support". Normally, you want a support of >90% to be really confident. 

You can specify 1000 "ultra-fast" boostraps in IQ-TREE using -bb 1000.
  
### Tree-building exercises
To visualize trees, we're going to use ITOL: https://itol.embl.de/upload.cgi. You can copy and paste your tree file into the text box, or upload the file directly; you probably have to download the file to your local computer first in either case. 

First, start an interactive (or scavenger) session with 8 cores and some memory:

```srun -p interactive -N 1 -n 8 --mem=10gb --pty bash```
and load your conda environment.

1. Run IQ-TREE with a Poisson substitution matrix and no additional parameters.

```iqtree -s LEAN.fasta -nt 8 -m POISSON -bb 1000```

There are a bunch of output files, including "LEAN.fasta.parstree". This is a parsimony tree inferred by IQ-TREE as a starting point, so let's take a look at it. 

Where are the nematodes in this tree? 

Now let's look at the ML tree we computed: "LEAN.fasta.treefile". Now where are the nematodes? Does anything else look weird about this tree? Which parts of the tree are well-supported, and which are poorly-supported?

2. Let's try some more complex models! Try one or more different combinations on your own, using any of the above flags. 
  Note: you will either need to specify -redo (which will overwrite your previous files), or -pre something, which will save new files with that prefix.
  
How does your new, more complex tree(s) compare to the simple Poisson model? Does the topology change, or the support values? How do we know what model we should use?

3. Luckily, IQ-TREE comes with a way to find the model that best-fits your data. If you run
 ```iqtree -s LEAN.fasta -nt 8 -redo -m TESTONLY```
 IQ-TREE will report the best model, without doing the subsequent tree analysis. With -m TEST, it will infer the best model and automatically construct a tree. 
 What is the best model in this case? 
 
 
## Some bonus details
You'll notice that the parsimony tree definitely suffered from LBA, and that this was partly rectified even with a very simple model. For the most part, adding complexity to the models didn't change this topology, although the support values tended to get lower when we used better-fitting models. This is because the "best" tree is still wrong! We only know this from external information. The ML tree correctly tells us that nematodes are more closely related to arthropods than deuterostomes, but it hasn't resolved the relationship between Arthropoda and Nematoda---the low support values tell us that we need to investigate those relationships in more detail. Remember that this dataset was made by humans to be problematic. 

There are more complex models that are not included in the default ModelFinder becuase of runtime. To include the free-rate models, you can run -m MF instead of -m TEST. There are also some new, fancy models called "site-heterogeneous mixture models", that are specifically suited for ameliorating LBA becuase they are better at modeling multiple substitutions. 

Normally, we can have different amino acid frequencies (e.g. -F), but they are assumed to be the same across all sites in the alignment. It turns out that is a big assumption: for example, an amino acid in an inter-membrane domain of a protein might have pretty different biochemical constraints than a site in the extracellular environment. Some amino acids might be much more common at some positions. Mixture models are an attempt to model this so-called **site heterogeneity**, which is why they are called "site-heterogeneous" models. These models were originally specific to Bayesian methods (PhyloBayes, http://www.atgc-montpellier.fr/phylobayes/), but IQ-TREE has implemented approximations of the Bayesian model in a maximum-likelihood framework. 

If you have time, you can run one of these models like this: 

```iqtree -s LEAN.fasta -nt 8 -m LG+C20 -ft LEAN.fasta.treefile -bb 1000 -pre C20```

*This model implicitly assumes a gamma rate distribution, so you don't need +G*
A special thing about mixture models is that they want a starting tree in order to estimate model parameters. We specifiy this with -ft, and let's use our previous best tree.

This model is a much better fit to the data, but still doesn't change the topology. 
However, 


### Some takeaways and tip
In the real world, **taxon sampling is probably more important than your model**. Try to "break up" long branches by including additional species. Try to sample evenly across your groups of interest. Try to choose a close outgroup. In this example, the _only_ way to really resolve these relationships would be to sample more taxa in an intelligent way, and perhaps to sequence more genes.

-


Site-heteregeneous models are sort of fancy and are good at ameliorating LBA. Some people argue that these models are prone to overfitting... nothing goes un-debated in phylogenetics! 


Think about support values! If you see a weird relationship, but the support is ambiguous, don't draw strong conclusions. 




 
  


