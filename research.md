---
layout: page
title: Research
---

I am interested in method development and analysis integrating machine learning, computational physics and molecular simulation.  My research focuses on developing machine learning based methods to extract conformational and kinetic information from molecular trajectories, and utilizing that information to improve sampling efficiency.  

## <span style="color: #397249">Enhanced sampling using autoencoders with data augmentation</span>

Understanding of physical systems depends crucially on sufficient sampling of system conformations.  However, the presence of energy barrier generally impedes efficient sampling.  One way to solve this issue is to add biasing forces to "pull" the system along important directions (which are encoded in "collective variables").  Finding such directions is essentially a dimensionality reduction problem, where we want to reduce high-dimensional conformational representations (e.g. encoded as Cartesian coordinates of atoms) to low-dimensional embeddings.  For sampling problem, we have some additional requirements for dimensionality reduction: 

1. we need to have an explicit function mapping from high-dimensional space to low-dimensional space

2. time and space complexity should be O(N) to be efficient

3. low-dimensional embeddings should be **invariant under arbitrary translations and rotations** of molecules, as we are not interested in rigid movement of molecules

To satisfy 1 and 2, we use autoencoders.  To satisfy 3, we borrow idea of data augmentation from image recognition.  The idea is, when an image of cat is shifted/rotated/scaled, it should still be classified as a cat, not anything else.  Therefore, the same molecular conformation with different center-of-mass positions or orientations should still be mapped to the same outputs.  

How do we encode this idea in an autoencoder?  We require that all input conformations are mapped to the same **"aligned" conformation**.  To illustrate the idea, we use cat images as input and show how they are projected onto low-dimensional space.  

Firstly let's see what happens if we use traditional autoencoders.  Recall that traditional autoencoders faithfully reconstruct whatever we put in as inputs.  Therefore in this case, the same "cat conformation" (sitting quietly, smiling with its tail pointing upwards) with different orientations will be mapped to different outputs.  Hence in order to be mapped to different outputs,low-dimensional embeddings **must be different**.


![]({{ site.url }}/figures/data_augment_1.png)

Now consider autoencoders with data augmentation.  We require all these cats be mapped to the same "aligned" outputs:

![]({{ site.url }}/figures/data_augment_2.png)

Here we use an alignment function ("little human") to generate expected outputs for inputs, such that same "cat conformation" with different orientations are mapped to the same output.  Therefore low-dimensional embeddings must be the same as well.

To implement this, we simply need to use following error function for neural network training:

![]({{ site.url }}/figures/data_augment_3.png)

where A(.) is autoencoder output, "little human" is the alignment function, "cat" is the input conformation, R is the regularization term.   What are those sticks and snoopy dogs in the autoencoder output?  They represent reconstruction error, as the autoencoder cannot perfectly reconstruct every detail and there is **information loss** when doing dimensionality reduction.

We use this idea together with enhanced sampling techniques to improve sampling efficiency, and have built an open-source package available [here](https://github.com/weiHelloWorld/accelerated_sampling_with_autoencoder).

For more information, checkout [this](https://onlinelibrary.wiley.com/doi/full/10.1002/jcc.25520) paper.


## <span style="color: #397249">Hierarchical autoencoders: dimensionality reduction with importance ranking</span>

In a typical dense layer in a neural network, **all nodes are equivalent**, meaning that if we swap weights of any two nodes, the output will stay unchanged.  Therefore, unlike PCA, an autoencoder cannot sort "principal components" (which we call "collective variables" or CVs for autoencoders) by their importance.  

To make sure autoencoder CVs are sorted by important, we need to **break the equivalence and rewire the neural network**.  Based on idea of [NLPCA](http://www.nlpca.org/), we design two following autoencoder architectures:

![]({{ site.url }}/figures/hierarchical_autoencoder.png)

On the left hand side, we have a "collective hierarchical autoencoder".  The idea is for an input `s`, instead of just creating one reconstructed output, we create n outputs (n is dimensionality of low-dimensional space, n = 3 in the diagram above).  Now here is the key: we **break the equivalence of nodes in the CV layer** (which encodes CVs), such that the information of first output (`q1`) only comes from CV1, and information of second output (`q2`) only comes from CV1, CV2, and so on.  The loss function `L` is the average reconstruction loss of these reconstructed outputs `L = ((s - q1)^2 + (s - q2)^2 + (s - q3)^2) / 3`.  Since CV1 participates in all reconstructions, it should contain most important information.  CV2 participates in all reconstructions except output `q1`, therefore it should be second important.  By doing this, we get the low-dimensional CVs ranked by their importance.

On the right hand side, it is a different variant we call "boosted hierarchical autoencoder".  The idea is based on **boosting**, which is also the idea behind **boosted trees** and **residual networks**.  As the same with previous variant, first output `q1` is constructed from CV1 only.  What is different is about following nodes: for CV2, instead of directly collaborating with CV1 to construct `q2`, it aims to fit the **residual error** that has not been learned by the CV1.  Let the fitted result be `d2` (as indicated in the diagram), then the second constructed result `q2` is `q2 = q1 + d2`.  This is similar to a boosted regression tree which **adds several tree outputs** to get the final output.  Similarly, CV3 fits residual error that has not been learned by CV1 and CV2 as `d3`, and corresponding constructed output is `q3 = q1 + d2 + d3`.  Again, loss function is still the average reconstruction loss `L = ((s - q1)^2 + (s - q2)^2 + (s - q3)^2) / 3`.  Following similar discussion, we can convince ourselves that CV1, CV2, CV3 are low-dimensional CVs sorted by importance.

## <span style="color: #397249">Discovering slow modes from time-series data</span>

TODO


## <span style="color: #397249">Theoretical bounds of time-lagged autoencoders</span>

We derived and proved the greatest lower bound of the reconstruction loss for time-lagged autoencoders, for finite-state stationary process.  We generalized autocorrelation to multidimensional cases