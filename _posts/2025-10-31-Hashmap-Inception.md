---
layout: default
title: Stable Subarrays With Equal Boundary and Interior Sum (LC 3728)
category: dev
published: true
use_math: true
---

# The Mathematical Insight

So just for some context I have to count the number of *stable* subarrays. What does stable mean?
1. It is at least length 3
2. The first and last elements are equal to the sum of the interior elements.

So Given:

$$\text{arr}[l], \text{arr}[l+1], \ldots, \text{arr}[r-1], \text{arr}[r]$$

and since:

$$\text{arr}[l] = \text{arr}[r] = \sum_{i=l+1}^{r-1} \text{arr}[i]$$

then:

$$\text{arr}[l],\, \text{arr}[l+1],\, \dots,\; \text{arr}[r-1],\, \text{arr}[r] \;\;\;\longrightarrow\;\;\; \text{arr}[r],\, \text{arr}[r],\, \dots,\; \text{arr}[r]$$

and then:

$$\sum_{i=l}^{r} \text{arr}[i] = \text{arr}[l] + \sum_{i=l+1}^{r-1} \text{arr}[i] + \text{arr}[r]$$

$$\sum_{i=l}^{r} \text{arr}[i] = 3 \cdot \text{arr}[r]$$

We can rewrite the sum as prefix sum(exclusive):

$$\text{prefix}[r+1] - \text{prefix}[l] = 3 \cdot \text{arr}[r]$$

Conceptually if we are iterating the endpoint of the subarray then we can maintain a lot of possible start points that satisfy the conditions:

$$\text{prefix}[l] = \text{prefix}[r+1] - 3 \cdot \text{arr}[r]$$

1. $r - l + 1 \geq 3$
2. if there exists an $\text{arr}[l] = \text{arr}[r]$ such that $\text{prefix}[l] = \text{prefix}[r+1] - 3 \cdot \text{arr}[r]$ and then we count how many $\text{prefix}[l]$ satisfy this condition


# The Key Data Structure

We can use a hashmap to determine for a given value v, if there exists a previous value v where the prefix sum uptill that point is equal to prefix[r+1] - 3 * arr[r] and see how many times it appears.

Translation: unordered_map<int, unordered_map<int,int>> cnt;

This hashmap has two keys:
1. The value v
2. The multiple (given there are multiple values v) prefix sums uptill that point

The value:
1. The frequency of the prefix sums uptill that value of v


# The implementation
```cpp

class Solution {
public:
    long long countStableSubarrays(vector<int>& capacity) {
        int n = capacity.size();

        vector<long long> prefix(n+1, 0);

        for(int i = 0; i < n; i++) prefix[i+1] = prefix[i] + capacity[i];

        unordered_map<long long, unordered_map<long long,long long>> cnt;

        long long ans = 0;

        for(int r = 0; r < n; r++){

            if(r - 2 >= 0){ // r - l + 1 >= 3 -> solve for l
                int l = r-2; // I am storing potential l's as I iterate the endpoint
                cnt[capacity[l]][prefix[l]]++;
            }

            long long need = prefix[r+1] - 3LL * capacity[r];

            if(cnt.count(capacity[r])){
                auto &mp = cnt[capacity[r]];
                if(mp.count(need)) ans += mp[need];
            }
        }

        return ans;
    }
};
```


<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
