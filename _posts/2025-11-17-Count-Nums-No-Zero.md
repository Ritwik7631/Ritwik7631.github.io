---
layout: default
title: Count Distinct Integers After Removing Zeros (LC 3747)
category: dev
published: true
use_math: true
---

# The Problem
Given a positive integer n, I have to return the number of distinct integers from 1 to n after removing all zeros.

# The Mathematical Solution

The key observation is that the set of distinct integers after removing all zeros are going to be <= n. $$g(x) \leq x \leq n$$

So we can break this into the *sum* of two cases:
1. numbers with fewer digits than n
2. numbers with the same number of digits as n

**Case 1:** Suppose n has k digits. A number with exactly L digits (where 1 <= L <= k-1) has the form
aL, aL-1, ..., a1 

and must satisfy the condition where ai must be in the range [1,9] for all i.

**Combinatorics Multiplication Principle:** When counting numbers with digit restrictions with a fixed number of allowed choices, then the total number of possible numbers is the product of the number of choices for each digit.

So by the *Combinatorics Multiplication Principle*:
the first digit has 9 choices,
the second digit has 9 choices,

the Lth digit has 9 choices.

The total number of valid L-digit numbers is (9^L)

Since there are many L-digit numbers where L ranges from 1 to k-1, we can sum up the number of valid L-digit numbers

$$\sum_{L=1}^{k-1} 9^L$$

**Case 2:** If we use the same logic as case 1, then we will count numbers that are greater than n. So we have to think of something different.

Let's say our n is a number whose digits are abcde and lets say the number we are counting is xyzwv.

To make sure xyzwv is less than abcde, we have to make sure that:
1. x are numbers less than a
2. y are numbers less than b
3. z are numbers less than c
4. w are numbers less than d
5. v are numbers less than e

If you think about it logically, if we pick an x < a then then the remaining digits y, z, w, v can be anything so there are 9^(remaining digits) possibilities. 
Then we can add the count of above with the following
If we pick an y < b then then the remaining digits z, w, v can be anything so there are 9^(remaining digits) possibilities.
So on and so forth.

Now here is an edge case. What if we encounter a 0 in n. Very Simply we just stop because we cannot have a number with a digit less than 0 in it.

If n itself has no zero, then we can increment the count by 1 since we haven't counted n yet.

$$(k\text{-digit part}) = \sum_{i=0}^{k-1} (d_i - 1) 9^{k-i-1} + \begin{cases} 1, & \text{if } n \text{ has no zero digit,} \\ 0, & \text{otherwise.} \end{cases}$$

$$k-i-1$$ is the number of digits left to choose for the remaining digits.

Mathematical Implementation:
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    long long countDistinct(long long n) {
        string s = to_string(n);
        int k = (int)s.size();

        // precompute powers of 9
        long long pow9[20];
        pow9[0] = 1;
        for (int i = 1; i < 20; ++i) {
            pow9[i] = pow9[i - 1] * 9;
        }

        long long ans = 0;

        // 1) numbers with fewer digits than n
        for (int L = 1; L < k; ++L) {
            ans += pow9[L];      // 9^L numbers with no zero digit
        }

        // 2) numbers with same number of digits as n
        bool hasZero = false;
        for (int i = 0; i < k; ++i) {
            int d = s[i] - '0';
            int remaining = k - i - 1;

            if (d == 0) {
                hasZero = true;
                break;
            }

            // digits 1..(d-1) at this position, then any non-zero digits for the rest
            ans += (long long)(d - 1) * pow9[remaining];
        }

        // if n itself has no zero digit, count it
        if (!hasZero) ans += 1;

        return ans;
    }
};

```

# The Digit DP Solution
You can *NOT* brute force this problem because n = 10^15.

```cpp

```


<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
