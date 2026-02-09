---
layout: default
title: Minimum Time to Complete All Deliveries (LC 3733)
category: dev
published: true
use_math: true
---

# The Problem Restated

Drone A and Drone B must make d1 deliveries and d2 deliveries respectively and must recharge at r1 spaced intervals and r2 spaced intervals respectively and one drone can make a delivery at any given hour and we have to find the minimum number of hours to complete all deliveries.

# Mathematical Insight

We can NOT simulate the deliveries for each drone because there can be *1e9* deliveries for each drone. So we need to use a binary search to find the minimum hours to deliver everything.

## Binary Search
The range of all possible answers ranges from 0 to max(d1, d2) * max(r1, r2) because the maximum number of hours to complete all deliveries is the maximum number of deliveries * the maximum recharge interval.

To reduce the search space for a given total hours, $t$:

For Drone A:

$\text{total hours} - \text{busy hours} \geq d_1$

so

$t - (t/r_1) \geq d_1$

For Drone B:

$t - (t/r_2) \geq d_2$

But what we are not done yet. There will be hours where both drones have to recharge so neither drone can make a delivery. 

Key: The LCM of $r_1$ and $r_2$ and subsequent multiples cannot be used to make deliveries.

Thus:

$t - (t/\text{lcm}(r_1, r_2)) \geq d_1 + d_2$

**If ALL 3 conditions are satisfied, then t is valid.**

# The implementation
```cpp
class Solution {
public:
    long long gcdll(long long a, long long b){
        while (b) { long long t = a % b; a = b; b = t; }
        return a;
    }

    long long lcm(long long a, long long b){
        return (a / gcdll(a, b)) * b;
    }

    bool valid(long long t, vector<int>& d, vector<int>& r){
        if((t - (t/r[0]) >= d[0]) && (t - (t/r[1]) >= d[1]) && (t - (t/lcm((long long)r[0], (long long)r[1]))) >= (d[0] + d[1])) return true;

        return false;
    }

    long long minimumTime(vector<int>& d, vector<int>& r) {
        long long hi = (long long)(d[0] + d[1]) * max(r[0], r[1]);
        long long lo = 0;

        while(lo < hi){
            long long mid = lo + (hi - lo)/2LL;

            if(valid(mid, d, r)){
                hi = mid;
            }
            else{
                lo = mid + 1;
            }
        }

        return hi;
    }
};

```

# Euclidean Algorithm for GCD

## Algorithm

**Input:** Two positive integers $a$ and $b$.

The algorithm proceeds by repeatedly applying the division algorithm:

$$
\begin{align}
a &= bq + r \quad \text{where } 0 \leq r < b \\
b &= rq_1 + r_1 \quad \text{where } 0 \leq r_1 < r \\
r &= r_1q_2 + r_2 \quad \text{where } 0 \leq r_2 < r_1 \\
&\vdots \\
r_{i-2} &= r_{i-1}q_i + r_i \quad \text{where } 0 \leq r_i < r_{i-1} \\
r_{i-1} &= r_iq_{i+1} + 0
\end{align}
$$

**Result:** $\gcd(a,b) = r_i$ (the last non-zero remainder).

## Why it works

**Theorem:** If $a = bq + r$, then $\gcd(a,b) = \gcd(b,r)$.

Applying this theorem iteratively:

$$
\begin{align}
\gcd(a,b) &= \gcd(b,r) \\
&= \gcd(r, r_1) \\
&= \gcd(r_1, r_2) \\
&= \cdots \\
&= \gcd(r_{i-1}, r_i) = \gcd(r_i, 0) = r_i
\end{align}
$$

**Proof of Theorem:**

We prove that $\gcd(a,b) = \gcd(b,r)$ by showing that the sets of common divisors of $(a,b)$ and $(b,r)$ are identical.

**Part 1:** Let $d$ be any common divisor of $a$ and $b$. Then $d \mid a$ and $d \mid b$.

Since $r = a - bq$ and $d$ divides both $a$ and $b$, we have:
$$
d \mid a, \quad d \mid b \quad \Rightarrow \quad d \mid (a - bq) \quad \Rightarrow \quad d \mid r
$$

Therefore, $d$ is also a common divisor of $b$ and $r$.

**Part 2:** Let $e$ be any common divisor of $b$ and $r$. Then $e \mid b$ and $e \mid r$.

Since $a = bq + r$ and $e$ divides both $b$ and $r$, we have:
$$
e \mid b, \quad e \mid r \quad \Rightarrow \quad e \mid (bq + r) \quad \Rightarrow \quad e \mid a
$$

Therefore, $e$ is also a common divisor of $a$ and $b$.

**Conclusion:** Combining both parts, we see that $d$ is a common divisor of $a$ and $b$ if and only if $d$ is a common divisor of $b$ and $r$. Since the sets of common divisors are identical, their greatest common divisor must also be identical:

$$
\gcd(a,b) = \gcd(b,r)
$$


<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
