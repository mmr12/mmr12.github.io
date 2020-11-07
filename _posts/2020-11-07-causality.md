# An introduction to causality

This is the first chapter of a series of two, aiming to discuss tools to leverage invariances in a dataset to build robust predictors. 

In this first chapter we introduce the language of causality, linking the universe of 'cause and effect' to probability theory. We define the probabilistic role of causal parents and formalise the concepts of causal invariance and interventional environments.
Here's the table of contents:

1. TOC
{:toc}

## Introduction
### The Generalisation quest

One of the key considerations of any predictive method is how well it performs on unseen data. Usually, to estimate the generalisation error one pools together all data collected, shuffles and separates it into a training and test set, trains the model to learn correlations between the input and the output on the former and reports the error metric on the latter. Although this approach is already indicative on whether a model is overfitting to the training data, it can lead the algorithm to absorb data collection biases, learning spurious correlations and hindering its ability to generalise to unseen situations. 

### Student GPA example
As an example, say we would like to predict student's Grade Point Average (GPA). We obtain data from two universities A and B about the amount of time the student spent studying and their first salary after leaving university. We shuffle and split the data, train an algorithm and obtain a good accuracy on the test data. We then apply the same algorithm to a third dataset collected from university C, and find that our accuracy drops. Looking into the data, we notice that students with similar GPA graduating from university A earn a first salary 10\% higher than students from university B, and 25\% higher than students from university C. This phenomenon could be explained by unoberseved variables affecting the student first salary such as location and university reputation. The result is that the correlation between the student GPA and its first salary is dependent on the environment, or, as we define it, *spurious*. Hence a model using a student first salary as a predictor is likely to fail in unseen scenarios. 

To the contrary, as a thought experiment, if students were magically forced to study less for their exams, then we would see their grades drop accordingly. Therefore the relationship between the amount of time spent studying and the student GPA is unmoved by changing environments. We say that the relationship between the study hours and the student's GPA is invariant. Treating the datasets from university A and B separately would enable us to discern spurious correlations from invariant ones and %integrate only the latter in a predictive model.
increase the robustness of our predictive model.


How can we identify and leverage invariant relationships to create more robust predictors?

### Cause and effect
In order to answer the question, consider what makes the relationship between the number of hours studied and a student's GPA. In universal wisdom, if a student studies for their course then they will achieve high grades. From a *cause and effect* optic, we intuitively know that the amount of time worked (H) is the cause of the student grade (G). If we were to doubt our intuition, a simple timeline argument supports the claim. As the studying happens before the exam, the exam cannot be the cause of the studying. We say that H is the cause of G. In addition, students with higher GPA are more likely to get higher paying jobs (S). Hence we claim that G causes S. The relationship between the three variables is depicted in Figure 1, where arrows represent causal relationships. A more in depth explanation of such causal graphs takes place in Section *Structural Equation Model and causal graphs*. 

![Student GPA causal graph](../images/2020/11/07/ISO_figures-Page-1-intro-fig.jpg)
__*Figure 1: Student GPA causal graph*__

What are the defining factors of the causal relationship between H and G? We will answer this question in Section *Causal parents*. In Section *Interventions* we explain how environments may change, whereas in the next Section *Invariant predictors* we discuss two methods to utilise the invariance of causal relationships to build robust predictive models across environments.

## Causality
### Causal graph and nomenclature
#### Causal parents
Following the example in the introduction, the GPA of a student is caused by the number of hours they spent studying. If the new variable 'weather' ($$W$$) is introduced, we can safely state that overall the weather has no impact on a student GPA, that is the same number of hours of study will result in the same GPA. Translating this statement in mathematical terms with variables $H$, $G$ and $W$ we obtain:

\begin{equation}
    P(G|H, W) = P(G|H) 
\end{equation}

Thinking more broadly about our observational setting, the time spent studying is not the only factor affecting a student GPA. For example the student stress and difficulty of the exam are other causes of their GPA. Let us collect all variables that have a direct effect on $G$. These are all the causal parents of $G$, denoted $\text{par}(G)$. In our example, assume $\text{par}(G)=\{$ "weekly hours studied", "stress", "exam difficulty"$\}$. Similarly we can define the children of a variable as all the variables that are directly affected by it, and the ancestors and descendants of a variable as all the indirect causes and indirect effects of a variable. We can claim

\begin{equation}
    P(G|\,\text{par}(G), Z) = P(G|\,\text{par}(G))\quad\forall Z\notin \text{par}(G)  
\end{equation}


Assume a student transfers from university A to university B, where university A and B have standardised examinations, changing the observational setting. Denote by $e_1$ the first environment, and $e_2$ the second. Regardless of the location, the same student will achieve the same GPA. More generally, parents of a variable have the same effect on the variable. 

\begin{equation}
    P(G^{e_1}|\,\text{par}^{e_1}(G)) = P(G^{e_2}|\,\text{par}^{e_2}(G))
\label{eq:parent-inv}
\end{equation}

This property is not limited to observational data. Suppose we forcefully intervened on the difficulty of the exam by increasing considerably its complexity. All other factors remain constant except the exam difficulty. The relationship between the exam difficulty and the GPA would not be broken, rather we would observe a different part of its distribution. Hence equality (\ref{eq:parent-inv}) would still hold. This is in contrast with the first salary example discussed in the introduction. In the latter, the distribution $$P(G\|S)$$ would vary from one university to the other.

Although equality (\ref{eq:parent-inv}) holds in most scenarios, a specific type of intervention invalidates it. Suppose we intervene on the exam, this time by increasing the score of all students by 30\%. Then the same student having studied the same amount of time and being presented with the same exam will achieve a different GPA.

Or, as Peters et al. [2016] note [^1]:
>If we consider all `direct causes' of a target variable of interest, then the conditional distribution of the target given the direct causes will not change when we interfere experimentally with all other variables in the model except the target itself.[Peters et al., 2016, p. 948][^1]

#### Structural Equation Model and causal graphs
To describe the knowledge of all causal relationships present in a given universe of variables $\{X_1,...,X_d\}$, we introduce the concept of Structural Equation Model (SEM).

 __*Definition:*__
*An SEM $\mathcal{C}:= (S,N)$ governing the random vector $X:=(X_1,...,X_d)$ is a set of structural equations*
\begin{equation}
    S_i: \,\,X_i \leftarrow f_i(\text{par}(X_i), N_i)
\end{equation}
*where $\text{par}(X_i)\subseteq{X_1,...,X_d}\setminus\{X_i\}$\} are called parents of $X_i$, and $N_i$ are independent noise random variables. We say that "$X_i$ causes $X_j$" if $X_i\in\text{par}(X_j)$. We call causal graph of $X$ the graph obtained by drawing a node for each $X_i$ and an edge from $X_i$ to $X_j$ if $X_i\in\text{par}(X_j)$ [Arjovsky et al., 2019][^2].* 

For linear Gaussian noise models such as the ones described by Peters et al. [2016][^1], we can formulate SEMs specifically as follows.

 __*Definition:*__
*A Gaussian SEM $\mathcal{C}:= (S,N)$ governing the random vector $X:=(X_1,...,X_d)$ is a set of structural equations* 
\begin{align}
    S_i:\,\, &X_i \leftarrow \sum_{k\neq i}\beta_{i,k}X_k + \epsilon_i& &i=1,..,p\\
    &\epsilon_i\sim\mathcal{N}(0, \sigma^2_i)& &i=1,..,p
\end{align}
*where $\epsilon_i$ are independent noise random variables.
The parents of $X_i$ are given by*
\begin{equation}
    \text{par}(X_i) = \{k\in\{1,..,p\}\setminus \{i\}: \beta_{i,k}\neq 0\}
\end{equation} 

Note here that $\beta_{i,k}$ and $\beta_{k,i}$ are distinct.

The link between SEM and probability follows from the former's definition and is given by the Causal Markov Condition [Scheines, 1997][^3]

 __*Definition:*__
*A variable $X_i$ is independent of its non-descendants given its parents par$(X_i)$.* 

#### SEM and Bayesian Networks
Although causal graphs can look similar to Bayesian networks, probabilistic networks and causal networks are not interchangeable. Probabilistic networks relate to the joint distribution of variables present in the system. Edges from variable nodes have the single use to indicate the probabilistic independences extracted from the data. However, the orientation of the arrows bare no significance and cannot be interpreted in a causal manner. As an example inspired from  Dawid [2010][^4], Figure 2 shows three Markov equivalent graphs, that graphs they can all be interpreted similarly in a probabilistic context. By d-separation
, 'first salary' is independent from 'weekly hours studying' given `GPA average'. However only one set of arrows defines the natural relation between the three variables, that is their causal relationship shown in Figure 1.

![Three Markov equivalent graphs](../images/2020/11/07/Fig2.png)
__*Figure 2: Three Markov equivalent graphs*__

|In Bayesian networks theory, two variables are independent if they are d-separated. Conditions for d-separation are:
    - $X\leftarrow Z\leftarrow Y$ and $Z$ is observed
    - $X \rightarrow Z \rightarrow Y$ and $Z$ is observed
    - $X\leftarrow Z \rightarrow Y$ and $Z$ is observed
    - $X\rightarrow Z \leftarrow Y$ and all of $Z$ and $Z$'s descendants are unobserved|

If we assume the causal graph will be one of the Markov equivalent graphs, we cannot know which equivalent graph that is. On the other hand, two variables might be independent in probability but linked in causality. As an example, take the relationship between weight and exercise. Exercise consumes calories which decreases weight, but also increases appetite which increases weight. If in a dataset collected the causal effect of exercise on weight and appetite cancel each other then one would observe that weight and exercise are probabilistically independent.

#### Assumptions
So far we described how causal parents are defined, and how causal graphs and Bayesian graphs relate (or rather how they do not).

In order to reach more meaningful conclusions, a few assumptions can be to restrict the universe of possibilities. 

**Acyclicity**: A common assumption, which we will make ourselves throughout the manuscript is that there are no feedback cycles in the causal graph. Hence causal graphs are acyclical.

**Faithfulness**: The faithfulness assumption says that there are no two paths that cancel each other out. Formally, if any independence relation in a dataset is not a consequence of the Causal Markov Condition, then the dataset in unfaithful Eberhardt
and Scheines [2007][^5]. Going back to the weight and exercise example, the dataset where the two variables are independent is unfaithful and would commonly be discarded.
This assumption allows for a stronger link between Bayesian Networks and causal graphs. Given every independence found in a dataset is the result of the Causal Markov Condition, then one of hte Markov equivalent Bayesian graphs represents the  causal relationships of the system.

A criticism of assuming faithfulness is that up to our knowledge, there is no method to test for faithfulness without knowledge of the underlying causal graph.


Having defined the causal graph, we now describe how different environments are created.
## Footnotes

[^1]: [Peters, J., Bü̈hlmann, P. and Meinshausen, N. [2016], ‘Causal inference by using invariant prediction: identification and confidence intervals’, Journal of the Royal Statistical Society: Series B (Statistical Methodology) 78(5), 947–1012.](https://rss.onlinelibrary.wiley.com/doi/abs/10.1111/rssb.12167)

[^2]: [Arjovsky, M., Bottou, L., Gulrajani, I. and Lopez-Paz, D. [2019], ‘Invariant risk mini- mization’, arXiv preprint arXiv:1907.02893 . Version v3.](https://arxiv.org/abs/1907.02893)

[^3]: [Scheines, R. [1997], ‘An introduction to causal inference’.](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.118.3002)

[^4]: [Dawid, A. P. [2010], Beware of the dag!, in I. Guyon, D. Janzing and B. Scho ̈lkopf, eds, ‘Proceedings of Workshop on Causality: Objectives and Assessment at NIPS 2008’, Vol. 6 of Proceedings of Machine Learning Research, PMLR, Whistler, Canada, pp. 59– 86.](http://proceedings.mlr.press/v6/dawid10a.html)

[^5]: [Eberhardt, F. and Scheines, R. [2007], ‘Interventions and causal inference’, Philosophy of Science 74(5), 981–995.](http://www.jstor.org/stable/10.1086/525638)