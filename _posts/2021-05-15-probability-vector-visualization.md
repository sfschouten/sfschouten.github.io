---
title: Plotting on the Simplex - Visualizing (rank) correlations and losses.
tags:
  - master-program
categories:
  - study
excerpt_separator: <!--more-->
---

Mirror of [this post on Medium](https://sfschouten.medium.com/plotting-on-the-simplex-visualizing-rank-correlations-and-losses-182dd781f531).

For the past year or so I have been involved in a project systematically comparing feature-additive eXplainable AI (XAI) methods. This class of XAI methods attempt to explain a model's output in terms of its inputs. Specifically, they assign an importance weight to each of the model's input features.
<!--more-->
In the project, we compare the importance weights produced by different XAI methods using a correlation coefficient. 'Kendall's Rank Correlation Coefficient' better known as Kendall-$\tau$ has been our correlation coefficient of choice. 
However we had realized for some time that it is hard making an informed decision about which correlation coefficient is most suitable. There is no real way of making clear what the difference is between them.

This week I started thinking about ways to visualize (rank) correlation coefficients. By doing this I was hoping to find an intuitive way to get a sense for how the coefficients differ from one another.

It turns out that the key lies in the normalization of the importance weights such that they sum to one. Vectors whose components are all positive and sum to one can be interpreted as probability-vectors. And more importantly, they can be thought of as lying on a simplex, which is what we can use for our visualization.

To be a bit more specific, an (n+1)-dimensional probability-vector lies on the standard n-simplex. Thus, for a 2-dimensional visualization, we can use the 2-simplex (the triangle), which will allow us to visualize functions of 3-dimensional probability vectors.

For our first visualization let's start with a function of probability distributions, Entropy.

![Shannon Entropy](/assets/images/2021-05-15-visualisation/shannon_entropy.png)

The corners represent the extremes, where one of the vector's components is one and the rest are zero, so naturally the entropy is lowest for these points.
The middle (the simplex's barycenter) represents the vector where all components have the value of $\frac{1}{n}$, this is the point of highest entropy.

# Ranking and correlation metrics

Next let's look at our first correlation coefficient, Kendall-$\tau$.

![Kendall's Tau](/assets/images/2021-05-15-visualisation/kendall_tau.png)

On this plot we see some red and green dots in addition to the density. The red dot marks the position of the probability-vector with respect to the Kendall-$\tau$ values are calculated, after all a correlation coefficient is always between two probability-vectors. Let's call this vector the *reference* vector.
The green dots represent the reference's permutations. Because a rank correlation coefficient like Kendall-$\tau$ only uses ranking information, we see a correspondence between the permutations and the value of the function.
For kendall-$\tau$ we see that the triangle is subdivided in six areas, it turns out this subdivision is the [barycentric subdivision](https://en.wikipedia.org/wiki/Barycentric_subdivision). 

Below are two more similar rank-based metrics, the Spearman-$\rho$ and DCG (Discounted Cumulative Gain).
![](/assets/images/2021-05-15-visualisation/spearman_rho.png){:width="300px"}
![](/assets/images/2021-05-15-visualisation/dcg.png){:width="300px"}

Next up, the Pearson Correlation!

![Pearson](/assets/images/2021-05-15-visualisation/pearson.png)

Since the Pearson correlation is not a rank correlation, we now see a smooth function whose value decreases as we get further away from reference vector. 
Note that like with Kendall-$\tau$, rotation about the barycentre seems to represent the most meaningful direction of change.

# Listwise rank-based loss functions
Whilst enjoying my newfound ability to visualize these functions, I realized that there is another class of functions I can look at in this way: loss functions! Specifically, those used for Learning To Rank. Doing so should allow us to get a sense of how well one (loss) function can serve as a proxy for another (discontinuous rank metric).

For example, ListNet is a Learning To Rank technique that projects labels and scores onto the probability simplex and minimizes the Cross-Entropy between them.

So what does Cross Entropy look like?

![](/assets/images/2021-05-15-visualisation/cross_entropy.png)

This looks quite different from any of the ranking metrics, it certainly does not have the 'rotational' quality!
I think this shows why Cross Entropy might not be a great loss function to use for a learning to rank problem.
It does not recognize at all that moving across the barycentre flips the ranking to its opposite.

So can we use our new insight to construct a better loss function? 
Well, we might calculate the cosine similarity between the probability-vectors when transformed to have their origin at the barycenter. This directly quantifies the 'rotational quality' as observed in the ranking methods.

![](/assets/images/2021-05-15-visualisation/barycentric_cos.png)

Wait, this looks a lot like the Pearson Correlation? Well turns out, it is! (technically it is ‘1 - pearson’). So actually we have just found a new way of coming up with an existing metric. This is a decent first step, and others have used the Pearson correlation directly as a loss function. But I think there are a few other things to think about.

First, even though the ranking metrics don't care about how close we are to the barycentre (how high the entropy is), it might be better for optimization reasons to still encourage more confident rankings. Below I have added the euclidean distance to the loss function to capture this.

![](/assets/images/2021-05-15-visualisation/barycentric_proposed.png)

Second, it might make sense to simplify our labels, such that we are always measuring the distance from the middle of a barycentric subdivision. This captures the fact that the ranking metrics only look at the ranking information, not the different probability vectors that produce those rankings.

![](/assets/images/2021-05-15-visualisation/barycentric_proposed_2.png)

# Conclusion
I think this is a good additional way of analyzing these types of functions. It seems to me like it complements other analyses (e.g. showing a loss function is an upper bound of a metric) very well.

It also seems to suggest a number of new loss functions and/or new variants of existing ones. In the near-ish future, I hope to implement and test a few of these and see how they hold up compared to existing losses.

# Extras
![](/assets/images/2021-05-15-visualisation/4_ranking_kendall_tau.png){:width="300px"}

(1) I also plotted some of the ranking functions using 4-rankings, which when normalized lie on the standard 3-simplex (tetrahedron). Here you can see a spherical slice of the Kendall-τ function plotted on the tetrahedron. This was created by centring a sphere on the barycenter of the tetrahedron, and evaluating the function on the points of the sphere.

(2) I created all these plots in Mathematica, the notebook can be downloaded here.

(3) I already did some (very) limited testing of new loss functions on the task I am currently working on for my master’s thesis. The task is link prediction using Knowledge Graph Embeddings, which comes down to learning a ranking of the entities most likely to form a link with another entity by a given relation. For this task, the KL-Divergence is a common loss function, and it seems from my limited testing that replacing it with the losses given above is beneficial in some cases. And that’s while using hyperparameters found for use with the KL-Divergence. I hope the increase in MRR will be even greater when I do a hyperparameter search specifically for the new loss.


{% include mathjax.html %}
