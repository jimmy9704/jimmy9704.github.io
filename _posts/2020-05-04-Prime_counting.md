---
layout: post
title:  "Prime counting function"
date:   2020-05-04
desc: "Quick test on writing code snippets in a blog post"
keywords: "Prime counting function"
categories: [PYTHON]
tags: [PYTHON]
icon: icon-html
---

# Ex) Prime counting function

__<div style="text-align: right"> EE370: Software lab, Kyung Hee University. </div>__
_<div style="text-align: right"> Jong-Han Kim (jonghank@khu.ac.kr) </div>_

The prime-counting function is the function counting the number of prime numbers less than or equal to some positive integet $n$. We usually call the function by $\pi(n)$.

In this problem, you will be developing a code that computes $\pi(n)$.
First write a function Write a function `Eratosthenes(n)` that lists all the prime numbers less than or equal to `n`, by using the process of the Sieve of Eratosthenes as follows.

1. Create a list of consecutive integers from $2$ through $n$: ($2, 3, 4, \dots, n$).
2. Initially, let $p$ equal $2$, the smallest prime number.
3. Enumerate the multiples of $p$ by counting in increments of $p$ from $2p$ to $n$, and mark them in the list (these will be $2p, 3p, 4p,\dots$; the $p$ itself should not be marked).
4. Find the first number greater than $p$ in the list that is not marked. If there was no such number, stop. Otherwise, let $p$ now equal this new number (which is the next prime), and repeat from step 3.
5. The algorithm terminates when $p^2$ is greater than $n$. Then the numbers remaining not marked in the list are all the primes below $n$.

Once you've found the list of all the prime numbers less than or equal to $n$, you will be able to easily compute $\pi(n)$.

What is $\pi(1000000)$, the number of the prime numbers less than one million?


```python
a = []
b = []
n = 1000000
for i in range(0,n):
  a.append(i)

a[1] = 0
```


```python
for i in a:
  if (i != 0):
    b.append(i)
    for j in range(i, n, i):
        a[j] = 0
print(len(b))
```

    78498
