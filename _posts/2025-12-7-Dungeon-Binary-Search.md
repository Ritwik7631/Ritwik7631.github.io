---
layout: default
title: Total Score of Dungeons (LC 3771)
category: dev
published: true
use_math: true
---

# The Problem
You have a positive integer hp and two 1-indexed arrays damage and requirements. There are n rooms in a dungeon numbered from 1 to n. Entering room i reduces your healthy by damage[i] and after that reduction if your remaining hp is at least requirements[i] then you get one score point.

Then score(j) = the total score you get by entering rooms j to n starting with hp health points.

I have to return score(1) + score(2) + ... + score(n).

# The Mathematical insights
So when you read this problem you realize if you try to simulate this problem it will be too slow because there can be up to $10^5$ rooms and the time complexity will be $O(n^2)$ which is too slow.

Then if you think about why it is so slow, I realized that I am doing repeated sums of the same values. This is the classic cue for a prefix sum.

So lets say start walking from room j to room i.

$$
j, j+1, j+2, ..., i
$$

So the damage you take is can be defined with the function D(x) is the cumulative damage you take from room 1 to room x.

$$
D[0] = 0,
$$

$$
D[i] = \sum_{t=1}^{i} \text{damage}[t].
$$

Therefore for the range sum from j to i is

$$
D[i] - D[j-1]
$$

Naturally we want to know the health points remaining after entering rooms j to i.

$$
H(i,j) = \text{hp} - D[i] + D[j-1]
$$

And based on the problem if the health points remaining is at least the requirement of the room then you get one score point.

$$
H(i,j) \geq \text{requirements}[j]
$$

$$
\text{hp} - D[i] + D[j-1] \geq \text{requirements}[j]
$$

Now we isolate the term that depends on the start, j.

$$
D[j-1] \geq D[i] + \text{requirements}[i] - \text{hp}
$$

Now score(j) as given is the cumulative points earned from entering rooms j to n.

But if we write this more formally (remember j <= i)

$$
\text{score}(j) = |\{i \mid j \leq i \leq n, H(i,j) \geq \text{requirements}[i]\}|
$$

$$
\Rightarrow
$$

$$
\text{score}(j) = |\{i \mid j \leq i \leq n, D[j-1] \geq D[i] + \text{requirements}[i] - \text{hp}\}|
$$

Ultimately I need to find score(1) + score(2) + ... + score(n) which can be written as

$$
\sum_{i=1}^{n} |\{j \mid 1 \leq j \leq i, D[j-1] \geq D[i] + \text{requirements}[i] - \text{hp}\}|
$$

Here's where I put on my thinking cap. Initally I was thinking for each start j, walk forward and compute score(j). Then I reversed my viewpoint and then thought for each room i, count how many starting positions j could earnt the point at room i.

It is sufficient to find the minimum starting position j that can earn the point at room i since everything to the right of the minimum starting position j will also earn the point at room i.

Concretely, what I am saying is that if 

$$
D[j-1] \geq D[i] + \text{requirements}[i] - \text{hp}
$$

then 

$$
D[j] \geq D[i] + \text{requirements}[i] - \text{hp}
$$

and so on and so forth.

Because of the monotonicity of the prefix sum, we can use a binary search to find the minimum starting position j that can earn the point at room i where i can range from 1 to n.

This will give us a time complexity of $O(n \log n)$ which is good for this problem.

The implementation:
```cpp
class Solution {
public:
    long long totalScore(int hp, vector<int>& damage, vector<int>& requirement) {
        int n = damage.size();

        vector<int> D(n+1,0);

        for(int i = 0; i < n; i++){
            D[i+1] = D[i] + damage[i];
        }

        long long ans = 0;

        for(int i = 1; i <= n; i++){
            int T = D[i] + requirement[i-1] - hp;

            int hi = i-1, lo = 0;

            while(lo < hi){
                int mid = lo + (hi-lo)/2;

                if(D[mid] >= T){
                    hi = mid;
                }
                else{
                    lo = mid + 1;
                }
            }

            long long points = 0;
            if(D[lo] >= T) points = i - lo;
            
            ans += points;
        }

        return ans;
    }
};

The only tricky part is after the binary search, we need to check in fact if the index we landed on is >= T because there is a chance where it is < T in which case no points are earned.

```

<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
