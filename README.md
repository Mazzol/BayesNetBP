# BayesNetBP

This package implements belief propagation methods in Bayesian Networks to propagate evidence through the network. This package supports reasoning or inference in discrete, continuous and hybrid networks under the framework of Conditional Gaussian Bayesian networks. To cite this package, please use

>Han Yu, Moharil Janhavi, Rachael Hageman Blair. "BayesNetBP: An R package for probabilistic reasoning in Bayesian Networks". Submitted.

Andrew Yan from Cornell University has also made significant contribution to this package in improving its computational efficiency and expanding its functionality.

The belief propagation methods is implemented through interfacing the work by [Cowell, 2005](http://www.jmlr.org/papers/volume6/cowell05a/cowell05a.pdf) and the sum-product algorithm as described by Daphne Koller and Nir Friedman. Probabilistic graphical models: principles and techniques. MIT press, 2009.

This package is also available on [CRAN](https://cran.r-project.org/package=BayesNetBP), but will be more frequently updated on GitHub. To install the package from GitHub, please use

```{r, eval=FALSE}
library("devtools")
install_github("hyu-ub/BayesNetBP/BayesNetBP")
```
## Getting started

### Initialization
The belief propagation starts with initialization of a ClusterTree object by Initializer function. The ClusterTree object is the computational object of this algorithm. The Initializer function needs the input of a Directed Acyclic Graph (DAG) in the format of a graphNEL object, a data frame for estimating the local distributions and a named vector specifying node types. 

In the following example, the ClusterTree is initialized based on a dataset 'liver'. This data contains a graphNEL object liver$dag, a data frame liver$data, and a named vector liver$node.class indicates which nodes are discrete (TRUE) and continuous (FALSE).

```{r, eval=FALSE}
data("liver")
tree.init.p <- Initializer(dag=liver$dag, data=liver$data, node.class=liver$node.class, propagate = TRUE)
```

### Evidence absorption

After initialization, evidence can be absorbed into the model. The function AbsorbEvidence embeds hard or soft evidence about variables into the model. In the following example, the continuous node Nr1i3 is observed with value 1. For the discrete variable, chr1@42.65, we set the state to 1 (hard evidence). Likelihood evidence (soft evidence)is entered for Spgl1 (High: 0.9, Low: 0.2). This example demonstrates the use of different types of evidence that can be entered in for multiple variables in the network, simultaneously.

```{r, eval=FALSE}
tree.post <- AbsorbEvidence(tree.init.p, 
c("Nr1i3", "chr1_42.65", "Spgl1"),  # observed variables
list(1, "1", c(High = 0.9, Low = 0.2)) # observed values or likelihood
)
```

### Query posterior distributions

The marginal distributions of both continuous and discrete variables can be queried with the function Marginals. For continuous variables, the marginal is a mixture of Gaussian distributions. The output is a data frame with three columns: sub-population probabilities, means and variances. For a discrete variable, the marginal is a vector of state probabilities. The function SummaryMarginals computes means and standard deviations for continuous variables.

```{r, eval=FALSE}
marg <- Marginals(tree.post, c("HDL", "Ppap2a", "Neu1", "chr1_71.35"))
marg$marginals$HDL
SummaryMarginals(marg)
```
FactorQuery is a function that can provide the joint distribution of any combination of factors, as well as conditional distributions of all discrete variables. In the following example, the joint distrbution of HDL and Cyp2b10, and the conditional distribution of HDL, are computed.

```{r, eval=FALSE}
FactorQuery(tree.post, c("HDL", "Cyp2b10"), mode = "joint")
FactorQuery(tree.post, c("HDL"), mode = "conditional")
```

Observations following the joint distribution of the BN can be generated by Sampler function. The sampling results can also be used to approximately estimate the covariance and joint distributions of continuous variables.

```{r, eval=FALSE}
x <- Sampler(tree.post, n = 100)
cov(x[c("Ppap2a", "Neu1", "Degs1")])
```

This package also provides visualization tools and a Shiny app. Please check the package manual or vignette for details.
