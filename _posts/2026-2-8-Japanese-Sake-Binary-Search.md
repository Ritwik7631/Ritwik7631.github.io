---
layout: default
title: AtCoder Sake or Water (ABC 441 C)
category: dev
published: true
use_math: true
---

# The Problem
There are N cups each with a volume of A_i mL of liquid. We do not know if it is sake or water.

It is known that there are exactly K cups that have sake.

I can either drink one or more cups.

How many do I have to drink to ensure that I drink at least X mL of sake? If impossible, return -1.

$$
\text{cups} = [\text{cup}_0, \text{cup}_1, \text{cup}_2, \ldots, \text{cup}_{N-1}], \quad
\text{cup}_i = A_i, \quad \textbf{\(K\) cups of sake}
$$

# My Solution
My first instinct To minimize the number of cups, for each 
𝑌
Y you should choose the 
𝑌
Y largest cups, because that maximizes the guaranteed minimum sake you will drink. If you pick a smaller cup while skipping a larger one, the adversary can assign sake to hurt you (put water in the big skipped cup and, if forced, put sake in the smaller chosen cup), so that choice is never better.

When I thought "most liquid", I then started thinking about sorting it which then gave me the intuition to think about binary search. Because let me say the answer is M, then M+1, M+2, M+3, ... all guarantee that I will drink >= X mL of sake.

Now lets restate the problem. Say I arbitrarily choose Y cups to drink. I need a predicate statement that returns either true or false. Here is the question I came up with: Can Y cups be consumed to ensure >= X mL of sake?

Now the real thinking how can I efficiently determine the predicate statement?

This requires a mindset shift where you thinking about the worst case. Like the worst case where you drink Y cups and none of them are sake. To make it more tangible, you can rethink the problem where you pick Y cups and someone who knows what you picks will do their best to make you drink as little sake as possible by swapping the cups.

Worst Case:
None of the Y cups are sake. Meaning the rest of cups have sake. I would have to drink \(\boldsymbol{N - Y}\) cups to fill the quota of \(X\) mL of sake.

Guaranteed Sake Case:
The Y cups you chose is greater than the number of cups NOT chosen, then some of the Y is guaranteed to have sake by the Pigeonhole Principle.

$$
\begin{aligned}
&\textbf{Givens:} \quad N \text{ cups},\quad K \text{ cups of sake},\quad X \text{ mL quota} \\
&\textbf{Not given:} \quad Y \text{ cups} \\[0.5em]
&K \geq N - Y \quad \Rightarrow \quad K - N + Y \geq 0 \\[0.25em]
&\text{Let } \mathbf{q = K - N + Y}. \\
&\textit{If } q > 0\textit{, then we know some sake will be consumed.}
\end{aligned}
$$

Now the question is how much is consumed when q > 0? So think from your rival's pov. They will do their best to minimize the N-Y cups with sake but they can't because K - N + Y > 0. So in the Y cups you are choosing, they will pick the cups of sake with the LEAST volume. We don't want to do a linear scan to find the least volume cups, so we sort and calculate the prefix sum for quick lookup. In the guaranteed case, you among the larger Y cups you will pick q.

$$
\begin{aligned}
&\texttt{start} = N - Y \\
&\texttt{end} = \texttt{start} + q \\
&\texttt{worst} = \texttt{prefix\_sum[end]} - \texttt{prefix\_sum[start]} \\[0.5em]
&\textbf{if } \texttt{worst} \geq X \textbf{ then} \text{ cut off right half} \\
&\textbf{else } \text{ cut off left half}
\end{aligned}
$$

**The implementation:**
```cpp
#include <bits/stdc++.h>

using namespace std;

int main(){
	int N, K;
	long long X;

	cin >> N >> K >> X;

	vector<long long> cups(N);

	for(int i = 0; i < N; i++){
		cin >> cups[i];
	}

	N = cups.size();

	sort(cups.begin(), cups.end());

	vector<long long> prefix_cup_mL(N+1,0);
	for(int i = 0; i < N; i++){
		prefix_cup_mL[i+1] = prefix_cup_mL[i] + cups[i]; // exclusive prefix sum
	}

	long long lo = 1;
	long long hi = N+1;

	while(lo < hi){
		long long Y_cups_drank = lo + (hi - lo)/2;

		int q = (int)max(0LL, 1LL*Y_cups_drank + K - N); // guaranteed cups of sake among Y

		long long worst = 0;
		if(q > 0){
			long long start = N - Y_cups_drank;
			long long end = start + q;
			worst = prefix_cup_mL[end] - prefix_cup_mL[start];
		}
		

		if(worst >= X){
			hi = Y_cups_drank;
		}
		else{
			lo = Y_cups_drank + 1;
		}
	}

	if(lo == N + 1) cout << -1;
	else cout << lo;

	return 0;
}
```

<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
