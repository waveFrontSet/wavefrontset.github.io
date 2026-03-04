---
title: "The Markov Chains of Elina Harper"
date: 2024-03-29T00:00:00+01:00
draft: false
slug: the-markov-chains-of-elina-harper
categories:
  - math
tags:
  - probability
---

*Spoiler Warning*: This blog post contains mild mechanical spoilers for the
second scenario of the *Innsmouth Conspiracy* expansion of *Arkham Horror LCG*.

In *The Vanishing of Elina Harper*, the players need to find out who abducted
agent Elina Harper and at what location she's taken hostage. In total, there are
6 suspects and 6 locations. During setup, the players need to randomly (and
secretly) choose one suspect and one location and shuffle the remaining
locations and suspects together as the *Leads deck*. During actual gameplay, the
players may then trade a certain amount of clues to look at the top (up to 3)
cards of this deck, choose to draw one of the revealed cards and shuffle the
remaining cards together with a replacement card from the encounter deck.

This means that at all times there are 10 cards in the Leads deck and the
players know a certain amount of cards in that deck. The goal of this scenario
is to see as many cards of the Leads deck as possible. Because the players have
13 rounds to do so but may only look at the top cards once per round, the
question for the optimal strategy arises: Should they look at the top cards 
each round or should they wait until they have acquired enough clues to look at
the maximum amount of (3) cards?

Related to this is the question what the expected number of draws is when
looking at the top 1, 2 or 3 cards respectively. Interestingly, while the
expected number of draws when looking at the top card may be computed in a
brute-force manner, we desperately need to organize the computation for the
other two cases. This is where Markov chains will come into play.

## TLDR: The results

Because the following is math-heavy, let's first state the results.

- If you decide to only ever look at the top card of the Leads deck, you will
  need over **29 rounds** on average until you have seen every card.
- If you decide to look at the top 2 cards of the Leads deck, you will need over
  **14 rounds** on average until you have seen every card.
- Finally, if you decide to look at the top 3 cards of the Leads deck, you will
  need over **9 rounds** on average until you have seen every card.

Of course, this assumes that you are able to gather the required amount of clues
each round. Here's a table showing the probabilities of having seen each card
after each draw for the three strategies.

| Draw | Only drawing 1 | Only drawing 2 | Only drawing 3 |
|-----:|---------------:|---------------:|---------------:|
|    4 |              0 |              0 |         0.0149 |
|    5 |              0 |         0.0006 |          0.088 |
|    6 |              0 |         0.0095 |         0.2188 |
|    7 |              0 |         0.0388 |         0.3726 |
|    8 |              0 |         0.0939 |         0.5186 |
|    9 |              0 |         0.1713 |         0.6419 |
|   10 |         0.0004 |         0.2624 |          0.739 |
|   11 |          0.002 |         0.3583 |         0.8124 |
|   12 |         0.0062 |         0.4519 |         0.8663 |
|   13 |         0.0143 |         0.5387 |         0.9053 |

A few comments:

- To have a decent chance at success, your group should be able to gather at
  least 2 clues per player per round.
- If your group is able to gather *exactly* 2 clues per player each round, it
  might be better to spend those two clues each round (you will only have 8
  chances of looking at cards otherwise, totalling a probability of success at
  51.86% compared to 53.87% of spending 2 clues per player each round for 13
  rounds).
- This doesn't take into account that you might want to fish for victory points
  on locations in the Leads deck.

## Brute-forcing the case of only looking at the top card

Let $d$ be the number of cards we look at from the top of the Leads deck, $1
\leq d \leq 3$.
Let $k$ be the number of cards we have already seen. The [binomial
coefficient](https://en.wikipedia.org/wiki/Binomial_coefficient) $\binom{10}{d}$
is the number of possible outcomes to draw $d$ cards from a deck of 10 cards
(disregarding order of the drawn cards). Likewise, there are $\binom{k}{d}$ ways
to draw $d$ cards from the $k$ cards we have already seen. Consequently, the
probability to only draw cards we have already seen when having seen $k$ cards
and drawing $d$ cards at once is
\\[
q_{d,k} = \frac{\binom{k}{d}}{\binom{10}{d}} = \frac{k!(10-d)!}{(k-d)!10!}.
\\]
In the case $d=1$ this simplifies to $q_{1,k} = k/10$ so that the probability to
see a new card when already having seen $k$ cards is $p_{1,k} = 1-q_{1,k} = (10-k)/10$.

Let's compute the expected number of rounds we have to play when only drawing
one card from the Leads deck each round. Let $X_{k}$ be the random variable of
rounds we have to play until we see a new card when having seen $k$ cards from
the Leads deck. Then we may compute the expected value of $X_k$ as follows.
$$
\mathrm{E}[X_k] = 1 \cdot P(X_k = 1) + 2 \cdot P(X_k = 2) + 3 \cdot P(X_k = 3) +
\dotsb = \sum_{i=1}^\infty i \cdot P(X_k = i).
$$

The probability that we have to play exactly $i \geq 1$ rounds is 
$$
P(X_k = i) = q_{1,k}^{i-1} p_{1,k} = \frac{k^{i-1} (10 - k)}{10^i}
$$
because we don't have to make any progress in the first $i-1$ rounds (which has
probability $q_{1,k}^{i-1}$) and then finally see a new card in the $i$th round
(which has probability $p_{1,k}$). Plugging this into our computation for the
expected value, we end up with

$$
E[X_k] = \sum_{i=1}^\infty i \cdot \frac{k^{i-1} (10 - k)}{10^i} =
\frac{10-k}{10} \sum_{i=1}^\infty i \cdot \left(\frac{k}{10}\right)^{i-1}.
$$

Why have I rewritten the result in this form? Because we see now that we've
ended up with the first derivative of the [geometric
series](https://en.wikipedia.org/wiki/Geometric_series) $\sum_{i=0}^\infty
(k/10)^i$. Because the geometric series has the closed form
$$
\sum_{i=0}^\infty r^i = \frac{1}{1-r}
$$
which has the derivative $1/(1-r)^2$, we deduce
$$
\sum_{i=1}^\infty i \cdot \left(\frac{k}{10}\right)^{i-1} = \frac{1}{(1-k/10)^2}
= \frac{10^2}{(10 - k)^2}.
$$

Finally, we arrive at
$$
E[X_k] = \frac{10}{10 - k}.
$$

Note that this formula also holds for $k=0$ and yields $E[X_0] = 1$ (if we
haven't seen any card yet, we are guaranteed to see a new card in the first
round). Summing these numbers up yields the desired average number of rounds
when only drawing one card:

$$
\sum_{k=0}^9 E[X_k] = 10 \sum_{k=0}^9 \frac{1}{10-k} = 10 \cdot \sum_{i=1}^{10}
\frac{1}{i} = 10 \cdot H_{10}.
$$

where $H_{10}$ denotes the 10th [harmonic number](https://en.wikipedia.org/wiki/Harmonic_number).

Likewise, we can compute the probabilities that we only take, say, 10 rounds to
see all cards when only drawing one card from the Leads deck each round. In this
case, we need to have $X_k = 1$ for all $0 \leq k \leq 9$ and the probability
for that is
$$
\prod_{k=0}^9 p_{1,k} = \frac{10!}{10^{10}} = \frac{567}{1562500} = 0.00036288.
$$


## Modelling drawing from the Leads deck with Markov chains

Doing similar computations for the case where we draw more than 1 card each
round is possible but becomes unwieldy quickly. To orchestrate computations, we
use [Markov chains](https://en.wikipedia.org/wiki/Markov_chain) in the form of
transition matrices.

First, let us think about the case where we draw 2 cards each round:
- In the first round, when we haven't seen any cards yet, we are guaranteed to
  see two new cards.
- After the first round, we have three possibilities:
  - We could only draw the two cards we have already seen,
  - We could draw one card we have already seen and one new card,
  - We could draw two cards we haven't seen yet.
- At some point, we have either seen all cards or we end up with 9 cards seen --
  in the latter case, we only have two possibilities (see the last card or not).

A *state diagram* helps to visualize these possibilities. In our case, the state
is the number of cards we have already seen: The initial state is $0$ (we
haven't seen any cards) and the terminal (or *absorbing*) state is $10$ (we have seen everything
in the Leads deck). We draw an edge between states if it is possible to advance
to that state. Here's how it could look like:

{{<mermaid>}}
flowchart LR
    0 --> 2
    2 --> 2 & 3 & 4
    3 --> 3 & 4 & 5
    4 --> 4 & 5 & 6
    5 --> 5 & 6 & 7
    6 --> 6 & 7 & 8
    7 --> 7 & 8 & 9
    8 --> 8 & 9 & 10
    9 --> 9 & 10
{{</mermaid>}}

From a state diagram, we may derive the *transition matrix* $P$ that
has the probability of transitioning from state $i$ to state $j$ as its
$(i,j)$-entry. This transition matrix is our tool of orchestrating
our computations. We need to use two of its properties.

It can be proven (by using induction and the definition of matrix
multiplication) that $P^k$ contains the probabilities of transitioning
from state $i$ to $j$ in exactly $k$ steps as its $(i,j)$-entry. This
is the first property we need and we'll use it to compute the probabilities
of transitioning to the absorbing state in any given number of rounds.

To explain the second property, note that our transition matrix has the
[canonical form](https://en.wikipedia.org/wiki/Absorbing_Markov_chain)

\\[
P =
\begin{pmatrix}
Q & R \\\\
0 & 1
\end{pmatrix},
\\]

where $Q$ is the *transition matrix of transient states*, i.e. of all
non-absorbing states and $R$ is the vector of probabilities of transitioning
to the absorbing state.

From this, it can be shown (as a generalization of the geometric series to
matrices) that
\\[
N := \sum_{i=0}^\\infty Q^i = (\mathrm{id} - Q)^{-1}.
\\]
$N$ is the *fundamental matrix* and contains the expected number of steps of
transitioning from state $i$ to state $j$ before being absorbed as its
$(i,j)$-entry. This means that $N \cdot 1$ (where we abuse notation and denote
the vector that only contains ones by $1$) contains the expected number of steps
before being absorbed when starting at state $i$ as its entry at position $i$.
This is the property we use to compute the expected number of rounds.

### Computing the transition matrix

We have all the theory laid down but haven't actually defined the transition
matrix in our case. Let $1 \leq d \leq 3$ be the number of cards to draw. Then,
for all $0 \leq i, j \leq 10$,

\\[
P_{ij} = \frac{\binom{i}{d - j + i} \binom{10 - i}{j - i}}{\binom{10}{d}},
\\]
where we use the common convention that $\binom{n}{k} = 0$ whenever $k < 0$ or $k > n$
and the not-so-common convention to start the indices of our matrix at 0 because
it fits the names of our states (at least
in mathematics, it's more common to start indices of matrices at 1).

Let's try and make sense of the formula. At the beginning, in state $0$, where
we haven't seen any cards and draw $d$ cards from the Leads deck, the probability
to transition to state $d$ is one (and any other probability should be $0$). Is this
the case? Let's see:

\\[
P_{0j} = \frac{\binom{0}{d - j} \binom{10}{j}}{\binom{10}{d}} =
\begin{cases}
1 & j = d, \\\\
0 & \text{otherwise}.
\end{cases}
\\]

Here, we have used the convention that $\binom{0}{d-j} = 0$ if $j < d$ or $j > d$.
More generally, $P_{ij} = 0$ whenever $j < i$ (cards that have been seen
cannot be unseen).  What about the probability to not see any new cards, i.e.,
about $P_{ii}$? If we have already seen $i$ cards, draw $d$ cards, and haven't
seen any new cards that means that all $d$ cards are among the already seen $i$
cards. There are $\binom{i}{d}$ ways this could happen. Dividing this by the
total amount of possibilities, which is $\binom{10}{d}$, yields the desired
probability. Our formula yields the same result because $\binom{10-i}{0} = 1$.

As it turns out, $P_{ij} = 0$ also holds if $j > i + d$. Indeed, this is
equivalent to $d - j + i < 0$ which means that, by convention, $\binom{i}{d-j+i} = 0$.

In all of the cases we have investigated so far, our transition matrix really
models drawing from the Leads deck. The only cases that remain are $i + 1 \leq j \leq i + d$.
In this case, we want to make sure that the our formula really is
the probability of transitioning from state $i$ to the reachable state $j$. More
down to earth, we want to compute the probability of seeing exactly $j - i$ new
cards assuming that we have already seen $i$ cards. First note that seeing $j-i$
new cards means that $d-(j-i) = d-j+i$ cards are among the cards we have already
seen. There are $\binom{i}{d-j+i}$ possibilities for this, which explains the
first factor in the numerator of our formula. Furthermore, seeing exactly $j -i$
new cards means that they must be among the $10 - i$ cards we have not seen yet.
There are $\binom{10-i}{j-i}$ possibilities for this, which explains the second
factor in the numerator of our formula. Dividing by the total number of
possibilities to draw $d$ cards $\binom{10}{d}$ yields the desired probability.

### Computing the results

I've written a small Python script that computes the first few powers of the
transition matrix and the expected number of rounds. A small trick I've used
is that I've only computed the coefficients (i.e., the numerator of the
probabilities). To make things a little easier to understand, I've decided
for somewhat lengthier but more readable variable names; also, I've decided
against hard-coding the deck size of 10.

```python
from scipy.special import comb as binom


def c(
    cards_seen: int,
    new_cards_seen: int,
    cards_drawn: int,
    deck_size: int = 10,
) -> int:
    return binom(cards_seen, cards_drawn - new_cards_seen, exact=True) * binom(
        deck_size - cards_seen, new_cards_seen, exact=True
    )
```

The [`comb` (for *combinations*) function from the
`scipy`
library](https://docs.scipy.org/doc/scipy/reference/generated/scipy.special.comb.html)
 computes the
exact values of binomial coefficients with the same conventions for negative
values (as long as, you guessed it, you supplied it with the parameter
`exact=True`). You may verify that this really is the numerator of our formula for the entries of the transition matrix by doing the following substitutions.
- `cards_seen` -> $i$
- `new_cards_seen` -> $j - i$
- `cards_drawn` -> $d$
- `deck_size` -> $10$.

When assembling the coefficients, there's another trick I'm using. Note that our
transition matrix $P$ is determined by the transition matrix of transient states
$Q$. This generally holds for Markov chains that only have a single absorbing
state because all rows sum up to $1$. This means that I only need to compute the
transition matrix of transient states. I'm calling this the *transient coefficient matrix* (or *coefficient matrix of transient states*).

```python
import numpy as np


def transient_coefficient_matrix(
    cards_drawn: int, deck_size: int = 10
) -> np.ndarray[Any, np.dtype[np.int32]]:
    m = np.zeros(shape=(deck_size, deck_size), dtype=np.int32)
    for i in range(deck_size):
        m[i, i:] = [c(i, j, cards_drawn) for j in range(deck_size - i)]
    return m
```

I'm first filling up a $10 \times 10$ matrix (the whole transition matrix has
shape $11 \times 11$) with zeroes. We have observed that only up to $d+1$ values
are possibly nonzero in each row, namely, the entries at $(i, i), (i, i+1),
\dotsc, (i,i+d)$. This is what I'm using when filling in the nontrivial values.

Now we're at the heart of the matter where we can apply Markov chain theory.
To explain the following function, note that $C / \binom{10}{d} = Q$ for the
coeffient matrix $C$ of transient states. Thus,
\\[
N = (\mathrm{id} - Q)^{-1}
= \left(\mathrm{id} - C/\binom{10}{d}\right)^{-1}
= \binom{10}{d} \cdot \left(\mathrm{id}\cdot\binom{10}{d} - C\right)^{-1}
\\]

{{< notice info >}} I'm doing this because $C$ only contains integer values
which produces no rounding errors for additions and subtractions. {{< /notice >}}

The expected number of rounds when starting at state $0$ is the first component
of the vector $N \cdot 1$. This is how the following functions computes it.

```python
def expected_number_of_rounds(
    cards_drawn: int,
    deck_size: int = 10,
) -> float:
    m = transient_coefficient_matrix(cards_drawn, deck_size)
    total_possibilities = binom(deck_size, cards_drawn, exact=True)
    expected_rounds_vec = (
        total_possibilities
        * np.linalg.inv(total_possibilities * np.identity(deck_size) - m)
        @ np.ones(deck_size)
    )
    return expected_rounds_vec[0]
```
{{< notice note >}} [`@` is syntactic sugar for matrix multiplication in Python](https://peps.python.org/pep-0465/). {{< /notice >}}


Finally, we compute the probabilities of success by computing matrix powers of
the transition matrix. To do so, we first need to compute the transition matrix
from the coefficient matrix of transient states (by embedding it into an $11
\times 11$ matrix and then computing the final column by subtracting the sum of
each row from $1$).

```python
def probabilities_of_success(
    cards_drawn: int,
    deck_size: int = 10,
    up_to_round: int = 14,
) -> list[float]:
    m = np.zeros((deck_size + 1, deck_size + 1))
    m[:deck_size, :deck_size] = transient_coefficient_matrix(cards_drawn)
    m /= binom(deck_size, cards_drawn, exact=True)
    m[:, deck_size] = np.ones(deck_size + 1) - m.sum(axis=1)
    probs = []
    for power in range(1, up_to_round):
        result_vec = (
            np.linalg.matrix_power(m, power) @ np.eye(1, deck_size + 1, deck_size).T
        )
        probs.append(round(result_vec[0, 0], 4))
    return probs
```

To output the results, we use Pandas to generate a nice markdown table for the probabilities of success (which is basically what I've pasted into this blog post).

```python
def table_of_probs(up_to_round: int = 14) -> str:
    return pd.DataFrame(
        data={
            f"Only drawing {i}": probabilities_of_success(
                cards_drawn=i,
                up_to_round=up_to_round,
            )
            for i in range(1, 4)
        },
        index=pd.RangeIndex(start=1, stop=up_to_round, name="Round"),
    ).to_markdown()


if __name__ == "__main__":
    for drawn in range(1, 4):
        print(f"d = {drawn}")
        print(transient_coefficient_matrix(drawn))
        print(expected_number_of_rounds(drawn))
    print(table_of_probs())
```


## Further reading

It would be nice to obtain a closed formula for the probabilities of success or
the expected number of rounds, like the one we've obtained via bruteforce for
the case $d=1$.

- I believe [the SymPy library](https://www.sympy.org/en/index.html) might be
  helpful in finding one, but that's for another blog post.
- There's the [matrix geometric
  method](https://en.wikipedia.org/wiki/Matrix_geometric_method) that looks like
  it could fit our use-case. However, it requires the matrices along the
  diagonal to be identical which is not the case for us.
