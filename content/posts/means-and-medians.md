---
categories:
  - math
date: "2019-05-19T17:04:51Z"
id: 21
tags:
  - mean
  - median
title: Means and medians
url: /2019/05/19/means-and-medians/
---

Understanding random experiments through deterministic invariants is a central goal of the mathematical branch known as probability theory or stochastics. While the *expected value* (or the *mean*) of a random experiment has a very natural interpretation (as its name suggests, it is the value we expect to occur on average when repeating the experiment) and a simple ad hoc definition as well as an easy generalization once foundations of measure theory have been established, the situation for the *median* is not as nice. Even though the median is more robust when dealing with outliers and, thus, might yield a more convenient substitute for the expected value, I have not encountered a general definition and treatment of it until I was concerned with learning the underlying statistical theory of Machine Learning (as for instance in *The Elements of Machine Learning* by Hastie et. al. or in *Understanding Machine Learning: From Theory to Algorithms* by Shalev-Shwartz and Ben-David). This post gives a basic treatment for discrete random variables and should be accessible to readers with some understanding of elementary discrete math and probability.

## Expected values and medians for discrete random variables

Let $X$ be a random variable attaining finitely many real values $\\{x\_1, \\dotsc, x\_k\\} \\subset \\mathbf{R}$. Denote the probability that $X = x\_i$ by $p\_i$. We know that

- $0 \\leq p\_i \\leq 1$ for all $i$ and
- $\\sum\_{i=1}^k p\_i = p\_1 + p\_2 + \\dotsc + p\_k = 1.$

**Definition.** The *expected value* (or the *mean*) of the random variable $X$, denoted by $\\mathbb{E}\[X\]$ is the weighted average of its values, weighted by their respective probabilities, i.e. $\\mathbb{E}\[X\] := \\sum\_{i=1}^k p\_i x\_i.$

Let us quickly discuss a couple of examples and some computational rules before moving on to the median.

- If \\(X\\) is *constant*, i.e. it only attains one value \\(x\_1\\) with probability \\(p\_1 = 1\\), then the expected value agrees with that one value.
- If we assume that $p\_1 = \\dotsc = p\_k = 1/k$, i.e. that $X$ is *distributed uniformly on its values*, then the expected value will be the ordinary average of the values $\\{x\_1, \\dotsc, x\_k\\}$.
- Let $k = 2$ so that $X$ takes on only two values. Let the first value be $x\_1 = 0$ taken on with probability $p\_1 = 0.9999$ and let the second value be $x\_2 = 10000000000 = 10^{10}$ taken on with probability $p\_2 = 1 – p\_1 = 0.0001 = 10^{-4}$. By our definition, the expected value of $X$ is \\[\\begin{align\*}\\mathbb{E}\[X\] &amp;= p\_1 x\_1 + p\_2 x\_2 \\\\&amp;= 0.999 \\cdot 0 + 10^{-4} \\cdot 10^{10} = 10^6.\\end{align\*}\\] Even though we know with 99.99% probability that $X$ will be zero, the expected value is some unexpectedly high number because there is the trace of a chance that $X = 10^{10}$. This example is the heart of what we mean by that the expected value is prone to outliers.

**Computational rules.** The following rules are central to the expected value. They may be proved by (more or less) direct computation from the definition.

- The expected value is *monotone*, i.e. for random variables $X, Y$ such that $X \\geq Y$, we also have $\\mathbb{E}\[X\] \\geq \\mathbb{E}\[Y\]$.
- The expected value is a *linear functional*, i.e. for real numbers $\\alpha, \\beta \\in \\mathbb{R}$ and another random variable $Y$, $\\mathbb{E}\[\\alpha X + \\beta Y\] = \\alpha\\mathbb{E}\[X\] + \\beta\\mathbb{E}\[Y\]$.

**Definition.** A *median* of the random variable $X$ denoted by $\\mathbb{M}\[X\]$ or $\\mathbf{Median}\[X\]$ is a real number such that both the probabilities that $X \\leq \\mathbb{M}\[X\]$ as well as $X \\geq \\mathbb{M}\[X\]$ are bigger than $1/2$. In formulae: $P(X \\leq \\mathbb{M}\[X\]) \\geq 1/2$ and $P(X \\geq \\mathbb{M}\[X\]) \\geq 1/2$.

The problem is immediately apparent: The definition does not give a formula for the median and there is no reason for the median to be unique (note that I only defined what *a* median is). Let us take a look at the same examples we exercised for the expected value.

- If \\(X\\) is constant then the median agrees with the value that \\(X\\) attains with probability \\(1\\). Even this trivial example is not as immediate as in the case of the expected value, so let’s give some details: Let’s say the sole value that \\(X\\) attains is \\(x\\). Then we certainly have \\(P(X \\geq x) = 1 \\geq 1/2\\) and \\(P(X \\leq x) = 1 \\geq 1/2\\).
- If $X$ is distributed uniformly on its values, we may compute the median by ordering its values. Let $x\_1 &lt; \\dotsb &lt; x\_k$ be such an ordering of its values. We claim that every real number in the closed interval $I = \[x\_{\\lfloor(k+1)/2\\rfloor}, x\_{\\lceil(k+1)/2\\rceil}\]$ is a median of $X$. Here, $\\lfloor(k+1)/2\\rfloor$ denotes the biggest integer smaller than $(k+1)/2$ and $\\lceil(k+1)/2\\rceil$ denotes the smallest integer bigger than $(k+1)/2$ (in German, these brackets are known as the *lower resp. upper Gauß brackets*). If $k$ is odd, $(k+1)/2$ already is an integer. In this case, $I$ consists of one single point. Now, let $x \\in I$. Then $x \\geq x\_{\\lfloor(k+1)/2\\rfloor}$ and, thus, \\[\\begin{align\*} P(X \\leq x) &amp;\\geq P(X \\leq x\_{\\lfloor(k+1)/2\\rfloor})\\\\ &amp;= p\_1 + \\dotsb + p\_{\\lfloor(k+1)/2\\rfloor} \\\\ &amp;= 1/k (\\lfloor(k+1)/2\\rfloor) \\\\ &amp;\\geq 1/k ( (k+1)/2 – 1/2) = 1/2. \\end{align\*}\\] The other defining inequality follows similarly. Often, the median is agreed to be the middle point of the interval $I$ so that one can talk about *the* median in this case.
- Let \\(X\\) be the random variable taking on the value $0$ with probability $0.9999$ and $10^{10}$ with probability $10^{-4}$. Then $\\mathbb{M}\[X\] = 0$ is the only median of $X$. Indeed, $P(X \\geq 0) = 1 \\geq 1/2$ and $P(X \\leq 0) = P(X = 0) = 0.9999 \\geq 1/2$ so that $0$ is a median of $X$. If $x &gt; 0$, then $P(X \\geq x) = 10^{-4} &lt; 1/2$ and if $x &lt; 0$, then $P(X \\leq x) = 0 &lt; 1/2$ so that in both cases $x$ cannot be a median of $X$.

Comparing the outcome of the last example for the expected value and the median, we see that the median yields a more typical value of the random variable if its distribution is *skewed*. What if the distribution is *symmetric*, though?

**Definition.** Let $X$ be a random variable. We say that $X$ is *distributed symmetrically onto its values* if there is a *symmetry point,* i.e., a real value $s$ such that $P(X \\geq s + x) = P(X \\leq s – x)$ for all real values $x$.

**Observation.** If $X$ is distributed symmetrically onto its values, then the expected value is a median, i.e. $\\mathbb{E}\[X\] = \\mathbb{M}\[X\]$.

I would like to prove this observation; actually, I would like to do a little more and give you some insight into the thought process that goes into writing a proof for this. In order to separate my thought process from the sentences that actually end up being written up in the proof, let us agree on this: My thought process will be written in normal weight, while *sentences in the proof will be written in cursive.* This should come as no surprise, as more often than not proofs tend to be written in dense, short language without any decoration, but let me give you a spoiler here: Much more will be spent on elaborating on my thought process than actually writing down the proof.

**Proof.** We start with one thing that mathematicians often do to make the proof more readable. We write *“without loss of generality we may assume…”*, followed by a replacement of the original statement in question with something that, at first, might seem like a special case.

Let me give you more details for the statement at hand. I would like to get rid of the nuisance that is the existence of the symmetry point in the definition of a symmetric distribution. What if I were to replace the random variable $X$ by $Y = X – s$? Well, this shifts the values: $Y$ takes on the values $y\_1 &lt; \\dotsb &lt; y\_k$ where $y\_i = x\_i – s$. And this makes $0$ a symmetry point for its values; indeed, \\[\\begin{align\*}P(Y \\geq 0 + x) &amp;= P(X – s \\geq x) = P(X \\geq s + x) \\\\ &amp;= P(X \\leq s-x) = P(Y \\leq 0-x).\\end{align\*}\\]
Okay, this means that we have replaced $X$, the random variable that has some unknown symmetry point $s$, by some special random variable $Y$ that has a symmetry point $0$ that looks like it could be easier to handle. But what about our task at hand? Let us assume that we have proven the statement for the random variable $Y$, i.e., that the expected value $\\mathbb{E}\[Y\]$ is a median of $Y$. Does this tell us something about $X$? Yes: since $Y = X – s$ and because the expected value is linear, we have $\\mathbb{M}\[X-s\] = \\mathbb{M}\[Y\] = \\mathbb{E}\[Y\] = \\mathbb{E}\[X-s\] = \\mathbb{E}\[X\] – s$. We are close. If we knew that $\\mathbb{M}\[X-s\] = \\mathbb{M}\[X\] – s$, it would immediately follow that the expected value of $X$ is a median. And, in fact, this holds: Let $m$ be a median of $X$. Then \\(P(X-s \\geq m-s) = P(X \\geq m) \\geq 1/2\\) and \\(P(X-s \\leq m-s) = P(X \\leq m) \\geq 1/2.\\)

This all means that it is sufficient to prove the statement for the special random variable $Y$ because, as we have shown above, this implies that the statement is also true for $X$. Because mathematicians do not like to waste letters (or, rather, we like the letter $X$ quite a lot), we tend to rename $Y$ and call it $X$ again. Still with me? We are ready to write the first sentence of our proof:

*Without loss of generality, we may assume that $X$ has $0$ as its symmetry point.*

More often than not, there is no justification given for why no generality has been lost, because after all that internal monologue that led to this sentence it should be obvious (at least to the author). Because we might feel pity on any future reader stumbling upon this first sentence, let us at least leave a small hint for why this replacement is sane:

*Without loss of generality, we may assume that $X$ has $0$ as its symmetry point* *(otherwise, replace $X$ by $X – s$ where $s$ is the original symmetry point).*

One fact should be emphasized here: A written mathematical proof should not be expected to spell out every small detail to the reader. Rather, a proof should be a blueprint or a guideline that assists the reader in gaining understanding of why the statement is true. *How much* detail is spelled out depends on the mathematical level of the text the proof is embedded in. It is very much possible that the observation we are proving right now is left as an exercise to the reader if we are reading a text book on advanced probability theory or statistics so that the reader is expected to have some background on those topics.

Okay, let us turn to some computations now. Because the condition on the probabilities in the definition of a symmetric distribution and in the definition of the median look quite similar, let’s try and compute the median first. Since we have assumed that $0$ is a symmetry point and the condition $P(X \\geq x) = P(X \\leq -x)$ holds for all real values $x$, it has to hold for $0$, which means $P(X \\geq 0) = P(X \\leq 0)$. This looks like it could be useful if we were to prove that $0$ is a median. Let’s see: Because $X$ is a real valued random variable, the probability that $X$ attains *any* real value is $1$ (otherwise, something would be very off). We could rephrase that by saying that $X$ either is a negative real number, or a non-negative real number. Let us write this down in formulas: $1 = P(X \\geq 0 \\text{ or } X &lt; 0) = P(X \\geq 0) + P(X &lt; 0)$. Here, we have used that the events “$X$ is negative” and “$X$ is non-negative” are *disjoint* so that their probabilites simply add up to the union. Since the probability that $X$ is negative is smaller than or equal to the probability that $X$ is non-positive (because potentially $0$ could be a value $X$ attains), we see that $P(X &lt; 0) \\leq P(X \\leq 0)$. Putting this together with the assumption that $P(X \\leq 0) = P(X \\geq 0)$, we obtain $1 \\leq P(X \\geq 0) + P(X \\leq 0) = 2P(X\\geq 0)$, i.e. that $P(X \\leq 0) = P(X \\geq 0) \\geq 1/2)$ so that $0$ is a median. This lays down a proof strategy: We may show the assertion somewhat indirectly, namely by showing that both the median and the mean are $0$. Let us write this down to prepare our reader for the following.

*We will show that both the median and the expected value are $0$.*

Since we have already worked out that the median is $0$, we may pin that down.

*We will start by showing that the median is $0$.*

Now, to write computations down more concisely, mathematicians tend to rewrite their whole thought process in reverse order in a single string of equations leading to the desired result. Let me show you how this could be done in this case.

*Since $0$ is a symmetry point, we have \\[\\begin{align\*}P(X \\leq 0) &amp;= P(X \\geq 0) \\\\ &amp;= 1/2 \\bigl(P(X \\geq 0) + P(X \\leq 0)\\bigr) \\\\ &amp;\\geq 1/2 \\bigl( P(X \\geq 0) + P(X &lt; 0)\\bigr) = 1/2\\end{align\*}\\] which shows that $0$ is a median.*

A reader well-versed in basic probability theory can immediately follow these computations so that we leave out any justifications.

Turning to the expected value, we could use its definition and the assumption on the random value but I’m not exactly sure how. Also, my guts tell me that this will be a mess of indices and intransparent computation. Maybe a more abstract argument will work here. Let’s rephrase the fact that \\(0\\) is a symmetry point slightly: For each real value \\(x\\) we have \\(P(X \\leq x) = P(X \\geq -x) = P(-X \\leq x).\\) This means that the functions \\(x \\mapsto F\_X(x) := P(X \\leq x)\\) and \\(x \\mapsto F\_{-X}(x) := P(-X \\leq x)\\) agree everywhere. These functions are known as the *distribution function of \\(X\\) resp. \\(-X\\)* and, as their name suggests, encode information about the way a random variable is distributed. This suggests that \\(X\\) and \\(-X\\) are identically distributed, i.e. they attain the same values with the same probabilities and this of course means that they share the same expected value. And this is precisely what we need: Because the expected value is linear, \\\[\\mathbb{E}\[X\] = \\mathbb{E}\[-X\] = -\\mathbb{E}\[X\].\\\] But this is only possible if \\(\\mathbb{E}\[X\] = 0\\). If you have never heard of distribution functions before, let me give some hints for how to show that \\(X\\) and \\(-X\\) share the same values attained with the same probabilities more directly. Order the values \\(x\_1 &lt; \\dotsb &lt; x\_k\\) of \\(X\\). Then for \\(x &lt; x\_1\\), \\\[P(-X \\leq x) = P(X \\leq x) = 0.\\\] This means that \\(x\_1\\) is the smallest value that both \\(X\\) and \\(-X\\) attain. And they attain it with equal probability since \\\[P(-X \\leq x\_1) = P(X \\leq x\_1) = p\_1.\\\] Now we can work our way up through the values of \\(X\\) inductively. If we have already shown that both \\(X\\) and \\(-X\\) attain the values \\(x\_1, \\dotsc, x\_i\\) with the same probability, then \\\[P(-X \\leq x) = P(X \\leq x) = p\_1 + \\dotsb + p\_i\\\] for all \\(x\_i \\leq x &lt; x\_{i+1}\\) which means that there is no value in between \\(x\_i\\) and \\(x\_{i+1}\\) that \\(-X\\) attains. And \\(-X\\) attains \\(x\_{i+1}\\) with probability \\(p\_{i+1}\\) since \\\[P(-X \\leq x\_{i+1}) = P(X \\leq x\_{i+1}) = p\_1 + \\dotsb + p\_{i+1}.\\\] Okay, now let’s see what ends up being written in the proof:

*Turning to the expected value, a rephrasing of \\(0\\) begin a symmetry point is that \\(P(X \\leq x) = P(X \\geq -x) = P(-X \\leq x)\\) which means that \\(X\\) and \\(-X\\) share the same distribution function. Since this implies that they both attain the same values with the same probability, we have \\(\\mathbb{E}\[X\] = \\mathbb{E}\[-X\] = -\\mathbb{E}\[X\]\\) so that \\(\\mathbb{E}\[X\] = 0\\) follows.*

Up to now, one could come to the conclusion that a random variable is far from being symmetrically distributed if the expected value and the median stray far from each other (especially when recalling the example with the huge outlier). However, this is not the case. Let us look at a modification of the example with the huge outlier.

**Example.** Let \\(X\\) be a random variable attaining the values \\(\\{0, 10^{10}\\}\\). Assume that \\(0\\) is attained with probability \\(1/2 + \\varepsilon\\) where \\(\\varepsilon &gt; 0\\) is some arbitrarily small positive number. Then \\(\\mathbb{M}\[X\] = 0\\) while \\\[\\mathbb{E}\[X\] = (1/2 – \\varepsilon) 10^{10} \\geq 10^9\\\] at least for \\(\\varepsilon \\leq 2/5\\).

The last example shows that the strength of the median is also its weakness: The median only factors in the majority of values a random variable attains even if the majority is only ever-so-slightly bigger than \\(1/2\\). One fix could be to not only consider the median but also the *first and the third quartile*: These are real values that split up the values a random variable attains differently; namely, the first quartile \\(q\_1\\) is a real value such that \\(P(X \\leq q\_1) \\geq 0.25\\) and \\(P(X \\geq q\_1) \\geq 0.75\\) while the third quartile satisfies \\(P(X \\leq q\_3) \\geq 0.75\\) and \\(P(X \\geq q\_3) \\geq 0.25\\). In this context, the median is also known as the *second quartile*. Even more generally, we could extract the follow invariant.

**Definition.** Let \\(0 \\leq \\delta \\leq 1\\) and \\(X\\) be a random variable. A *\\(\\delta\\)-quantile for the random variable \\(X\\)* is a real number \\(q\_\\delta\\) such that \\(P(X \\leq q\_\\delta) \\geq \\delta\\) and \\(P(X \\geq q\_\\delta) \\geq 1 – \\delta.\\)

Thus, the median may also be called a \\(1/2\\)-quantile. A warning: The notation here might not be standard; for instance, the wikipedia article about quantiles (<https://en.wikipedia.org/wiki/Quantile>) talks about *\\(k\\)-th \\(q\\)-quantiles* which we would simply refer to as \\(k/q\\)-quantiles.

**Conclusion.** The combination of both invariants gives us a first impression of how well-behaved the distribution is: On the one hand, both the expected value and the median being close to each other might suggest that the distribution is close to being symmetric; on the other hand, both straying far from each other hints at the random variable suffering from outliers. To be sure about this, one should also consider other quantiles.
