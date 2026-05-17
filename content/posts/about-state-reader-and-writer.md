---
title: "About State, Reader, and Writer"
date: 2026-05-18T20:00:00+02:00
draft: true
slug: about-state-reader-and-writer
categories:
  - programming
  - math
tags:
  - category theory
  - haskell
series: []
---

The Reader and Writer monads are possibly the first monads a reader
encounters when working through a Haskell book.
They both depend on a type `s` and map a type `a` to

```haskell
Reader s a = s -> a
Writer s a = (s, a)
```

For the Writer monad to truly become a monad, the type `s` needs to have a monoid
instance (we'll see why later).

The `State` monad seems to be closely related when inspecting its definition:

```haskell
State s a = s -> (s, a)
```

{{< notice info >}}
I've stripped away the usual type definition to make the relationship clearer.
{{< /notice >}}

This begs the question: Is `State` the composition of `Reader` and `Writer`?
More precisely, is `State s a = Reader s (Writer s a)`?

## A closer look at `Reader` and `Writer` as functors

For a mathematician, `Reader` is more well-known as the `Hom`-functor
\\[
\mathrm{Hom}(s,-)\colon a \mapsto \mathrm{Hom}(s,a)
\\]
which may be defined for any [(locally small) category](https://ncatlab.org/nlab/show/locally+small+category).

As for `Writer`, we need a monoidal category with tensor product $\otimes$ but
we can certainly confine ourselves with the category of sets and take the tensor
product to be the cartesian product of sets if you like.
In any case, `Writer s` is then the product functor with the object `s`:
\\[
s \otimes -\colon a \mapsto s \otimes a
\\]

As for their functor instances, `fmap` for `Reader` is given by post-composition
while `fmap` for `Writer` is given by applying $f$ to the second factor. More
precisely, for any $f\colon a \to b$, we have
\\[
\mathrm{Hom}(s,f) = f_\ast\colon (g\colon s \to a) \mapsto (f \circ g\colon s \to b)
\\]
\\[
s \otimes f = \mathrm{id}\_s \otimes f\colon s \otimes a \mapsto s \otimes f(a)
\\]

{{< notice info >}}
Note that we haven't restricted the object `s` for `Writer` at any point here.
Indeed, the functor instance for `Writer` doesn't impose any restrictions on
the object `s`.
{{< /notice >}}

And now, in fact, we observe that _as functors_, `State s` is the composition of
`Reader s` and `Writer s`.

## The monad instances for `State`

The same can't hold for the monad instances, simply because `Writer` demands
the object `s` to be a monoid while the monad instance of `State` doesn't restrict
`s` at all. So, what is going on here?

Let's take a look at the unit (or the `return` function). For `Reader`, it is given
by sending an object to the constant function:
\\[
\lambda\_R\colon a \to \mathrm{Hom}(s,a) \, c \mapsto (y \mapsto c) = \mathrm{const}\_c.
\\]

As for `Writer`, here's where the monoid instance comes into play. If `s` is a monoid,
we can send any object to the product with the neutral element $e_s$ of the monoid:
\\[
\lambda\_W\colon a \to s \otimes a \, x \mapsto e\_s \otimes x.
\\]

Now, if `State` was the composition of `Reader` and `Writer` as monads, its `return` function
would need to be the composition of `Reader`'s and `Writer`'s. However,
\\[
\lambda\_R \circ \lambda\_W\colon x \mapsto (y \mapsto e\_s \otimes x) = \mathrm{const}\_{e\_s \otimes x}
\\]
while `State`'s return function is given by
\\[
\lambda\_S\colon x \mapsto (y \mapsto y \otimes x).
\\]

This is not the end of the story. We could still ask where this _canonical_ monad
structure comes from if it doesn't come from the monad instances of `Reader` and
`Writer`. The observation that `State` is the composition of `Reader` and `Writer`
as functors is the key: As it turns out, these two functors are _adjoint_, i.e. there's
a natural isomorphism
\\[
\mathrm{Hom}(s \otimes a, b) \cong \mathrm{Hom}(a, \mathrm{Hom}(s, b)).
\\]

An [adjunction always gives rise to a monad](https://en.wikipedia.org/wiki/Monad_(category_theory)),
with the monad's structure maps (unit, join) coming from the natural isomorphism.

{{< notice info >}}
It is also true that a monad gives rise to an adjunction, but there are
many adjunctions giving rise to the same monad - [there's a whole category
of them](https://en.wikipedia.org/wiki/Monad_(category_theory)#Monads_and_adjunctions)!
{{< /notice >}}

## Conclusion

To sum it up, `State` is not only the composition of `Reader` and `Writer`
as functors; its whole structure as a monad is determined by the _adjunction_ between
`Reader` and `Writer`.
