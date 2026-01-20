---
layout: default
title: Maximum Capacity within Budget (LC 3814)
category: dev
published: true
use_math: true
---

# The Problem
We are given two arrays of length $$n$$: $$costs$$ and $$capacity$$.

$$
\begin{aligned}
costs &= [c_0, c_1, c_2, \ldots, c_{n-1}] \text{ such that } costs[i] \text{ is the cost of } i\text{th machine} \\
capacity &= [cap_0, cap_1, cap_2, \ldots, cap_{n-1}] \text{ such that } capacity[i] \text{ is the } i\text{th machine's capacity}
\end{aligned}
$$

Now within a budget $$b$$, we have to find the maximum capacity we can achieve selecting at most 2 machines where the total cost is strictly less than $$b$$.

Mathematically equivalent to:
$$
\max_{\substack{M \subseteq \{0,1,\ldots,n-1\} \\ |M| \leq 2 \\ \sum_{x \in M} c_x < b}} \sum_{x \in M} cap_x
$$

# My Solution
I had the *intuition* to do *scenario analysis* when I saw at most 2 machines can be selected. So I thought of the following scenarios:
1. **No machines are selected**
2. **One machine is selected**
3. **Two machines are selected**

Now when **no machines are selected**, the capacity is 0 and the cost is 0. So the best answer here is 0.

When **one machine is selected**, all we have to do is find the machine with the maximum capacity that costs strictly less than $$b$$. This is *trivial with a linear scan*.

When **two machines are selected**, *things get more interesting*. I have to find the two machines with the maximum capacity that costs strictly less than $$b$$. Mathematically, this is:
$$
\begin{aligned}
\text{Find the } (i,j) \text{ where } j > i: \\
1. \quad &costs[i] + costs[j] < b \\
   &costs[i] < b - costs[j] \\
   &X = b - costs[j] \\
2. \quad &\max(capacity[i] + capacity[j]) \text{ for all } (i,j).
\end{aligned}
$$

**One trick** to get rid of nested loops for $i,j$ is to loop over all values of $j$ once and use a data structure to remember previous states aka $i$.

Now at this point, I was thinking to myself while iterating over $j$ ($costs[j]$, $capacity[j]$) how can I find a previous state that maximizes my capacity and is less than or equal to $X-1$. *Why X-1 you might ask?* Well $costs[i] < b - costs[j]$ is equivalent statement to $costs[i] \leq X-1$.

**X** represents the upper bound for the cost of machine $i$. Here I had the *intuition* to sort the machines by cost and then use a **prefix max array** to get the maximum capacity of a machine whose costs ranges from 0 to $X-1$.

It might be natural for some to ask *what if X-1 isn't an explicit cost in our sorted (increasing) cost array?* It would be easy to just look at `prefix_max_capacity[X-1]` to return the max capacity of machines 0 to machine $j-1$. But this doesn't work.

**The relevant question is:** "What is the largest index $i < j$ such that $costs[i] \leq X - 1$?"

This is a *classic boundary-finding problem* that can be solved with a **binary search**. I won't dive into why binary searches requires monotonicity of the search space. Binary search is a technique that involves asking a question that returns either a true or false answer. We take the midpoint of the search space, $k$, and evaluate $costs[k] \leq X-1$. We want the largest possible $k$. So our monotonic array looks something like this:

$$
T, T, T, \ldots, T, F, F, F, \ldots, F
$$

**We want the last True.** This is the largest index $i < j$ such that $costs[i] \leq X - 1$.

So if $k$ is **True**, then we cleave everything to the left of $k$ and set $lo = k + 1$. All the machines to the left of $k$ are valid, but we continue searching for the largest $k$ so our search space is now $[k+1, hi]$.

So if $k$ is **False**, then we cleave everything to the right of $k$ and set $hi = k - 1$. All the machines to the right of $k$ are invalid, but we continue searching for the largest $k$ so our search space is now $[lo, k-1]$.

**The implementation:**
```cpp
class Solution {
public:
    int getMaxCapacity(int remaining, int i, vector<pair<int,int>>& arr2, vector<int>& prefix_max_capacity){
        if (i == 0) return 0;
        int lo = 0, hi = i-1;

        while(lo < hi){
            int mid = lo + (hi-lo)/2;

            if(arr2[mid].first <= remaining){
                lo = mid + 1;
            }
            else{
                hi = mid;
            }
        }

        int idx;
        if (lo >= 0 && arr2[lo].first <= remaining) {
            idx = lo;        // all YES case
        } else {
            idx = lo - 1;    // first NO case
        }

        if (idx < 0) return 0; // no valid machines
        return prefix_max_capacity[idx];
    }
    
    int maxCapacity(vector<int>& costs, vector<int>& capacity, int budget) {
        int n = costs.size();
        int ans = 0;

        for(int i = 0; i < n; i++){
            if(costs[i] < budget) ans = max(ans, capacity[i]);
        }

        vector<pair<int,int>> arr2;
        for(int i = 0; i < n; i++){
            int cst = costs[i];
            int cap = capacity[i];
            arr2.push_back({cst,cap});
        }

        sort(arr2.begin(), arr2.end());

        vector<int> prefix_max_capacity(n, arr2[0].second);
        for(int i = 1; i < n; i++){
            prefix_max_capacity[i] = max(prefix_max_capacity[i-1], arr2[i].second);
        }

        for(int i = 0; i < n; i++){
            int cst = arr2[i].first, cap = arr2[i].second;
            
            if(cst >= budget) continue;

            int rem = budget - cst - 1;

            if(rem < 0) continue;

            int x = getMaxCapacity(rem,i,arr2,prefix_max_capacity);

            ans = max(ans, cap+x);
        }

        return ans;
    }
};
```

<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
