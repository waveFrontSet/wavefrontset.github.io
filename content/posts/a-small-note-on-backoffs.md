---
title: "A Small Note on Backoffs"
date: 2023-08-27T15:00:00+02:00
categories:
  - programming
  - math
tags:
  - python
  - http
---

Unfortunately, exchanging data with external services doesn't always go
smoothly. A mundane example is that the external service could simply be not
reachable at the moment you request data from it: It could be offline for
maintenance or it could be under heavy load, unable to respond to your request
in time. To make your own applications depending on external services more
robust, they will need to deal with these temporary downtimes.

One way to do so is using _retries with backoffs_. To account for the external
service being under heavy load, you _back off_ and give it some time to recover
until you retry. Naturally, this is an improvement over simply hammering the
server with requests in case it's already under heavy load.

How much time you let pass between retries is a matter of choice. You could be
clever and [incorporate the concrete exception or HTTP status code into your
decision like in `boto3`'s _adaptive_
mode](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/retries.html#adaptive-retry-mode)
or you could
simply double the time until the next retry. For example, the Python library
[`httpx` does
this](https://github.com/encode/httpcore/blob/master/httpcore/_sync/connection.py)
while its older sibling [`requests` also allows for adding some randomization
in](https://github.com/urllib3/urllib3/blob/main/src/urllib3/util/retry.py#L299).

In the context of serverless functions, you can't retry indefinitely because
you'll hit a timeout (for example, [the maximum timeout for an AWS lambda
function is 15
minutes](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-common.html#configuration-timeout-console)).
Let us a take a look at how to relate a given timeout with the number of
retries.

## Relating a timeout requirement with the number of retries

Following the simpler implementation of retries with backoffs in `httpx`, we
immediately retry after the first failure, then wait $b$ seconds ($b$ is the
configurable _backoff factor_) after the second and wait twice as long after
each following failure until we've reached the maximum number $n$ of retries. We
can compute the total backoff time in this case as

\\[
0 + b 2^0 + b 2^1 + \dotsb + b 2^{n-2} = \sum_{i=0}^{n-2} b 2^i.
\\]

This happens to be a [geometric
sum](https://en.wikipedia.org/wiki/Geometric_series) for which we have a closed formula:
\\[
\sum_{i=0}^{n-2} b 2^i = \frac{b (2^{n-1} - 1)}{2 - 1} = b(2^{n-1} - 1)
\\]

A necessary condition for your serverless function to complete successfully is
that this backoff time is smaller than or equal to the timeout $t$. I'd like to
call this the _Retry-Timeout-Inequality_:

\\[
b(2^{n-1} - 1) \leq t.
\\]

The way we've written the inequality down is probably not the most useful one:
Here, given the number of retries you'd get a lower bound for your timeout.
What's more interesting is fixing the timeout $t$ and then computing how many
retries we may afford. Let's rearrange to obtain $2^{n-1} \leq t/b + 1$. Then,
applying the $2$-logarithm and using the [logarithmic
identities](https://en.wikipedia.org/wiki/List_of_logarithmic_identities), we
end up with the following.

\\[\begin{align*}
n &\leq \log_2(t/b + 1) + 1 \\\\
&= \log_2(b+t) - \log_2(b) + 1
\end{align*}\\]

Let's call this the _logarithmic form of the Retry-Timeout-Inequality_.

**Example**. Let's assume our backoff factor is $1/2$ (as is the default for
`httpx`). Then the right-hand-side of the logarithmic form simplifies even
further:

\\[\begin{align*}
&\log_2(t+1/2) - \log_2(1/2) + 1 \\\\
&= \log_2\left(\frac{2t+1}{2}\right) + 2 \\\\
&= \log_2(2t + 1) + 1
\end{align*}\\]

If we assume further that our timeout happens to be $t = 30$ seconds, we see
that $2t + 1 = 61 < 64 = 2^6$ so that

\\[\begin{align*}
n &\leq \log_2(2t+1) + 1 \\\\
&< \log_2(2^6) + 1 \\\\
&= 7.
\end{align*}\\]

This means that we can retry 6 times without reaching the timeout, at least not
from the backoff times alone.

### Conclusion

To sum it up and generalize the last example, you can compute
the maximum number of retries when using `httpx` as follows.

1. Find the smallest power of two greater than $2t + 1$ where $t$ is your
   desired timeout in seconds. This means, find an integer $k$ such that
   $2^{k-1} < 2t + 1 < 2^k$.
2. Your maximum number of retries is $k$.
