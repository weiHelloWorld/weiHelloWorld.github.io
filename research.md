---
layout: page
title: Research
---  

## [<span style="color: #397249">Enhanced sampling using autoencoders with data augmentation</span>]({{ site.url }}/research_proj/enhanced_sampling_using_autoencoders.md)


## <span style="color: #397249">Hierarchical autoencoders: dimensionality reduction with importance ranking</span>

In a typical dense layer in a neural network, **all nodes are equivalent**, meaning that if we swap weights of any two nodes, the output will stay unchanged.  Therefore, unlike PCA, an autoencoder cannot sort "principal components" (which we call "collective variables" or CVs for autoencoders) by their importance.  

To make sure autoencoder CVs are sorted by important, we need to **break the equivalence and rewire the neural network**.  Based on idea of [NLPCA](http://www.nlpca.org/), we design two following autoencoder architectures:

![]({{ site.url }}/figures/hierarchical_autoencoder.png)

On the left hand side, we have a "collective hierarchical autoencoder".  The idea is for an input `s`, instead of just creating one reconstructed output, we create n outputs (n is dimensionality of low-dimensional space, n = 3 in the diagram above).  Now here is the key: we **break the equivalence of nodes in the CV layer** (which encodes CVs), such that the information of first output (`q1`) only comes from CV1, and information of second output (`q2`) only comes from CV1, CV2, and so on.  The loss function `L` is the average reconstruction loss of these reconstructed outputs `L = ((s - q1)^2 + (s - q2)^2 + (s - q3)^2) / 3`.  Since CV1 participates in all reconstructions, it should contain most important information.  CV2 participates in all reconstructions except output `q1`, therefore it should be second important.  By doing this, we get the low-dimensional CVs ranked by their importance.

On the right hand side, it is a different variant we call "boosted hierarchical autoencoder".  The idea is based on **boosting**, which is also the idea behind **boosted trees** and **[residual neural networks](https://en.wikipedia.org/wiki/Residual_neural_network)**.  As the same with previous variant, first output `q1` is constructed from CV1 only.  What is different is about following nodes: for CV2, instead of directly collaborating with CV1 to construct `q2`, it aims to fit the **residual error** that has not been learned by the CV1.  Let the fitted result be `d2` (as indicated in the diagram), then the second constructed result `q2` is `q2 = q1 + d2`.  This is similar to a boosted regression tree which **adds several tree outputs** to get the final output.  Similarly, CV3 fits residual error that has not been learned by CV1 and CV2 as `d3`, and corresponding constructed output is `q3 = q1 + d2 + d3`.  Again, loss function is still the average reconstruction loss `L = ((s - q1)^2 + (s - q2)^2 + (s - q3)^2) / 3`.  Following similar discussion, we can convince ourselves that CV1, CV2, CV3 are low-dimensional CVs sorted by importance.

Information about hierarchical autoencoders and their application in molecular simulations can be found [here](https://aip.scitation.org/doi/abs/10.1063/1.5023804).

## <span style="color: #397249">Discovering slow modes from time-series data</span>

In quantum mechanics, we have all information about how a system evolves if we know the evolution of each of its "independent orthogonal components" (aka "eigenstates").  This is also true for general dynamic systems: if we find such components (we call "slow modes") and figure out their evolution, we get all information about its kinetics.

How do we find these slow modes?  We use a similar idea to [variational method in quantum mechanics](https://en.wikipedia.org/wiki/Variational_method_(quantum_mechanics)).  Basically we search in a function space for the slow modes (which are functions) with specific objectives and constraints.  But unlike variational methods in QM, where we find eigenstates given the expression of the dynamic operator (Hamiltonian) but no observations, here we are interested in finding slow modes **given time-series observations but no expression for the dynamic operator**.

Let's consider the 1st component: the slowest mode of the system.  We want to find the function in a function space that is slowest (formally, meaning that the function `f(x)` has maximum [autocorrelation](https://en.wikipedia.org/wiki/Autocorrelation) given the time series data).  How do we define a function space?  A simple way is to first define a few **basis functions** and define the function space to be **all possible linear combinations** of these basis functions.  

But would it affect our results if we do not have a good pre-defined basis functions?  Yes, for instance if we use linear functions as basis functions (known as [TICA](http://msmbuilder.org/3.3.0/tica.html)), then it is not possible to capture nonlinear slow modes, which could be a big issue as **many systems are intrinsically nonlinear**.  

Is it possible to **adapt basis functions to data**?  Yes, one way is to use **[kernel methods](https://en.wikipedia.org/wiki/Kernel_method)** where we do not explicitly specify basis functions, but instead specify the similarity measure among data and basis functions are determined accordingly.  The problem is, the similarity measure is universal, which controls the "resolution" of the learned function.  If a slow mode has super fine details in some parts while do not have many details in others, we may want to **have different "resolutions" in different regions** to be efficient, which cannot be done with kernel methods.  

Consider following example, suppose `f(x)` is the slowest mode, which has a crazily oscillating part A together with smooth regions outside A:

![]({{ site.url }}/figures/slow_modes_1.png)

If we use a low "resolution", we do not need many data points to learn it, for instance we pick following data points (smiling faces):

![]({{ site.url }}/figures/slow_modes_2.png)

We are almost for sure that we lose all details of region A.  Can we add more points in A?  That is not going to help, since the "resolution" is so low that all points in A seem roughly the same.

What if we use a much higher "resolution"?  Now we need much more data, like this:

![]({{ site.url }}/figures/slow_modes_3.png)

It is fine and should resolve all details, but here comes another issue: **computation is too expensive**.  Since kernel methods require computing pairwise distance matrix which takes O(N^2) memory (N is number of points used).  Moreover, computation involves solving generalized eigenvalue problem, which takes O(N^3) time.  This means we cannot feed this method with too many data points, which limits its ability for complex systems.

Can we have basis functions adapted to data with flexible "resolutions"?  This is where neural networks come in.  Due to [universal approximation theorem](https://en.wikipedia.org/wiki/Universal_approximation_theorem), neural networks are flexible enough to approximate any continuous functions.  Based on work of [VAMPnets](https://www.nature.com/articles/s41467-017-02388-1), we use neural networks to learn basis functions and simultaneously find the optimal linear combination to fit the slowest mode using data.  We find this method is more accurate in discovering slowest mode.  Moreover, it is **much faster and much more memory efficient** since this neural network based method only takes O(N) memory and O(N) time.

If you are interested in how to discover higher-order slow modes, and theoretical connection with transfer operator theory and other details, please checkout [this](https://aip.scitation.org/doi/abs/10.1063/1.5092521) paper.  For implementation and application, checkout [this](https://github.com/hsidky/srv) GitHub repository.

## <span style="color: #397249">Theoretical bounds of time-lagged autoencoders</span>

We derived and proved the greatest lower bound of the reconstruction loss for time-lagged autoencoders, for finite-state stationary process.  

[test](./research_proj/test.md)

(to be continued)