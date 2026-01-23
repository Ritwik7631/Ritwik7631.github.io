---
layout: default
title: Count Caesar Cipher Pairs (LC 3805)
category: dev
published: true
use_math: true
---

# The Problem
I am given an array of strings of length $$n$$: $$words$$ and each word has length $$m$$.

$$
\begin{aligned}
words &= [word_0, word_1, word_2, \ldots, word_{n-1}] \\
word_i &\text{ is of size } m \text{ for } 0 \leq i \leq n-1 \\
word_i &\text{ contains only lowercase english letters}
\end{aligned}
$$

Then we are given an operation $$op$$ which is one of the following:

"Replace every letter in chosen string with the next letter in the alphabet. $z \rightarrow a$. Cyclic"

We are told $word_s$ is **SIMILAR** to $word_t$ if you can apply $$op$$ on either $word_s$ or $word_t$ **ANY number of times** such that $word_s == word_t$.

The goal is to return the number of pairs of words that are **SIMILAR**. Mathematically:
$$
\text{Number of } (i,j) \text{ such that } j > i \text{ and } word_i \text{ is SIMILAR to } word_j
$$

**Constraints:**
$$
\begin{aligned}
1 &\leq n \leq 10^5 \\
1 &\leq m \leq 10 \\
1 &\leq n \cdot m \leq 10^5
\end{aligned}
$$

# My Solution

Initially I was *naively* thinking of looping from $0$ to $n-1$ and for each $word_j$, I will see how many operated $word_i$ strings are **SIMILAR** to $word_j$. And then for each $word_j$, I will generate all the operated strings of $word_j$ and push it in a `unordered_map`. After working it out I realized my time complexity is $O(26nm \cdot \text{sizeof(map)})$.

I realized that generating all the operated strings of $word_j$ is *too expensive*. At this point I started to think: *can you represent each word in a way that is independent of how many times you apply the operation?*

Then I had the *intuition* that **the relative distance between characters remains constant** no matter how many times you apply the operation.

Now *relative to what* is the question. It doesn't really matter, we can measure distance between any character and the first character.

$\oplus$ denotes the concatenation operator.

$$
\bigoplus_{k=1}^{m-1} f(k) = \text{signature of } word_j \text{ where } f(k) = [word_j[k] - word_j[0] + 26] \bmod 26
$$

**This signature is the key to solving the problem.** Basically I will iterate each $word_i$ and compute its signature and I will add the frequency of this signature from $0$ to $i-1$ to my running total. I am treating $word_i$ as the $j$ in the $(i,j)$ pair. I am basically saying: *how many previous words are there that share the same signature as $word_i$?*


**The implementation:**
```cpp
class Solution {
public:
    long long countPairs(vector<string>& words) {
        int n = words.size();
        unordered_map<string,int> sig_freq;
        long long ans = 0;

        for (int i = 0; i < n; i++) {
            string sig_j = "";
            for(int k = 1; k < words[i].size(); k++){
                sig_j += to_string((words[i][k] - words[i][0] + 26)%26);
            }

            ans += sig_freq[sig_j];
            sig_freq[sig_j]++;
        }

        
        return ans;
    }
};
```

<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
