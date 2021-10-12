# Lab_Phylo

For today's lab, we're going to be using IQ-TREE (http://www.iqtree.org/doc/Tutorial), a popular maximum-likelihood program for tree-building. I like IQ-TREE because it's pretty user-friendly; the documentation (particularly on the website, but also in the help commands) is exceptionally clear, and the syntax isn't super confusing (compared to some tree programs that will remain nameless...). It's also more than powerful enough for most analyses.

The dataset we're using is adapted from Lartillot et al. 2007 (https://doi.org/10.1186/1471-2148-7-S1-S4), and consists of 8 Nematodes, 10 Arthropods, 5 vertebrates, 2 sponges, 4 cnidarians, and 7 fungi. Each species has 3 genes that were concatenated together into one amino acid alignment (36 taxa, 658 characters)---the original paper used 37 species, 146 genes, and 35,371 characters. **This is a particularly troublesome dataset!** The taxon sampling is designed to exacerbate long-branch attracion (LBA). Nematodes are (notoriously) weird and on a long branch all by themselves. In this case, since Fungi are so distantly related to all the animals, they are also on a long branch. Thus, with "normal" phylogenetic models, LBA pulls nematodes towards fungi at the base of the tree, and outside of their correct placement sister to the arthropods.

A quick note on rooting: when you infer a tree, it is usually **unrooted**, meaning it doesn't have any direction. You can't tell which direction time flows without external information; this is because most phylogenetic models are time-reversible. This is why we use **outgroups**, which are groups of sequences that we know are sister to all other sequences in our tree. We root the tree between the outgroup and ingroup. In this case, fungi are the outgroup, but they are a very distant outgroup (hence the LBA). A better choice in this case would be choanoflagellates. 

```
conda create -n trees
conda activate tree
conda install -c bioconda iqtree
```


### What kinds of models can you implement in IQ-TREE?

-POISSON is the simplest amino acid substituion model, with equal frequences between all pairs of amino acids.
-JTT, WAG, and LG are all common general substition models. These are based on empirical observations from large numbers of closely-related proteins, and are what most people use most of the time.
-There are other special matrices for mitochondria, viruses, etc.

-F: estimate empirical amino acid frequencies
  By default, amino acid frequencies are given by the substitution model. The -F parameter estimates these frequencies driectly from your data. 
  
-Some sites (in immune proteins...) can evolve very quickly. Other sites might be under strong selection and evolve very slowly.
G4: there are 4 rate categories drawn from a gamma distribution. The number is user-specific, but 4 is common. 
R4: there are 4 free rate categories (not constrained to a distribution; better fit but slower). 
  
-I: there is some proportion of invariant sites that never change
  -these are essentially removed from the analysis, prior to calculating the rate categories.
  
  
  
 -feel free to play around with different parameter combinations!
 
 -What if you try a POISSON matrix and no additional parameters?
 
 -What if you try a pretty well-fitting model like LG+I+G4?
 
 -What is the best-fit model?
 
 -What's wrong with this picure? Did the more complex model change the placement of Nematoda?
 


-So, even though applying model-based approaches are better than parsimony at dealing with LBA... LBA is still a problem sometimes! 
 
 
 
 -support values
 -bootstraps.
 
 We can specify 1000 "ultrafast" bootstraps in IQ-TREE using -bb 1000.
    
-In order to deal with this LBA, we can try using an even more complex model, a "site-heterogeneous mixture model". 
 
Normally, we can have different amino acid frequencies (e.g. -F), but they are assumed to be the same across all sites in the alignment. It turns out that is a big assumption: for example, an amino acid in an inter-membrane domain of a protein might have pretty different biochemical constraints than a site in the extracellular environment. Some amino acids might be much more common at some positions. Mixture models are an attempt to model this so-called **site heterogeneity**, which is why they are called "site-heterogeneous" models. You may see simpler models referred to as "site-homogeneous". These models were originally specific to Bayesian methods (PhyloBayes, http://www.atgc-montpellier.fr/phylobayes/), but IQ-TREE has implemented approximations of the Bayesian model in a maximum-likelihood framework. 

Specifically suited to ameliorate LBA, because they are better at modeling multiple substituions. 

Let's try it out and see if it fixes our Nematode problem! This will take several times as long to run as the previous model. A special thing about mixture models in IQ-TREE is that they want a starting tree in order to estimate model parameters. We can specifiy this with -ft, and let's use our previous best tree.

```iqtree -s LEAN.fasta -nt 8 -m LG+C20 -ft LEAN.fasta.treefile -bb 1000 -pre C20```

*This model implicitly assumes a gamma rate distribution, so you don't need +G*


## Some takeaways

Site-heteregeneous models are sort of fancy and are good at ameliorating LBA. Some people argue that these models are prone to overfitting... nothing goes un-debated in phylogenetics! 

In the real world, **taxon sampling is probably more important than your model**. Try to "break up" long branches by including additional species. Try to sample evenly across your groups of interest. Try to choose a close outgroup. 

Think about support values! If you see a weird relationship, but the support is ambiguous, don't draw strong conclusions. 




 
  


