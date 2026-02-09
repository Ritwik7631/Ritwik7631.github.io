---
layout: default
title: Longest Balanced Subarray II (LC 3721)
category: dev
published: true
---

# Lazy Propagation + Segment Trees

This problem forced me to consider the data structure known as segment trees or segtrees. **Segtrees are primarily useful for efficient querying and updating intervals or segments of an array.**

I will not be going into how segtrees work but rather how we can use them here, specifically those that support lazy propagation.

Lazy Propagation comes into the picture when the range updates must happen in O(log(n)) time as opposed to O(n) time (regular segtree). Critically, lazy propagation allows for overlap handling. Simply updates and queries are logarithmic.

# The Problem

Given an array of positive integers, I have to return the longest balanced subarray meaning there are an equal number of _distinct_ evens and odds.

The word _distinct_ really put a hair in my soup. Without it, the problem would be trivial using prefix sums and balance mapping. But once "distinct" enters, the prefix structure collapses: duplicates must be _uncounted_ for all subarrays that start before the previous occurence. That means the contribution of each element isn't fixed - it can appear, disappear, or reappear depending on the last index. Prefix sums cannot handle this because the basic assumption of prefix sums is that every value's effect is permanent and _cumulative_, while "distinct" makes each value's impact conditional and reversible.

The approach to solve this problem is to start with a an array of length n initialized to 0. Then as you iterate each index i, you update the _range_ [0,i] with +1 for even value and -1 for odd value. The leftmost 0 you see will be the start of a balanced subarray that ends at i. But now if you consider duplicates, you dont want to add +1 for multiple duplicates. So you need to use a hashmap to store the last index of each value so that your start point is the last index + 1. This way when adding +1 or -1 it is only for one unique instance of the value from [last index + 1, i]. If there is no last index the range update is from [0,i]. Now is where lazy propagation segtrees come in to play. We can update and query the range in O(log(n)) time.

## Putting It All Together

Below is the complete C++ implementation. It uses a lazy segment tree that supports **range add** and a **search for the leftmost zero**. The key idea is to maintain an array `F[L]` representing the difference between the number of distinct even and odd values in every subarray starting at `L` and ending at the current index `i`.

When a new number `nums[i]` arrives:
- If it's even, add `+1` to all prefixes `[0, i]`.
- If it's odd, add `-1` to all prefixes `[0, i]`.
- If this number appeared before at index `prev`, then remove its old contribution (`-parity`) for all prefixes `[0, prev]`.

At each step, the **first index L** where `F[L] == 0` gives the **longest balanced subarray** ending at `i`.  
The lazy segtree keeps these range updates and zero searches in `O(log(n))` time.

```cpp
class Solution {
public:
    struct SegTree {
        int n;
        vector<long long> seg, lz, mn, mx;
        SegTree(int n): n(n), seg(4*n,0), lz(4*n,0), mn(4*n,0), mx(4*n,0) {}

        inline int L(int p){ return 2*p; }
        inline int R(int p){ return 2*p + 1; }

        void apply(int p, int l, int r, long long v){
            seg[p] += v * (r - l + 1);
            mn[p]  += v;
            mx[p]  += v;
            lz[p]  += v;
        }

        void push(int p, int l, int r){
            if(lz[p] == 0 || l == r) return;
            int m = (l + r) / 2;
            apply(L(p), l, m, lz[p]);
            apply(R(p), m + 1, r, lz[p]);
            lz[p] = 0;
        }

        void pull(int p){
            seg[p] = seg[L(p)] + seg[R(p)];
            mn[p]  = min(mn[L(p)], mn[R(p)]);
            mx[p]  = max(mx[L(p)], mx[R(p)]);
        }

        void add(int p, int l, int r, int ql, int qr, long long v){
            if(qr < l || r < ql) return;
            if(ql <= l && r <= qr){
                apply(p, l, r, v);
                return;
            }
            push(p, l, r);
            int m = (l + r) / 2;
            add(L(p), l, m, ql, qr, v);
            add(R(p), m + 1, r, ql, qr, v);
            pull(p);
        }

        // find first index in [ql, qr] where value == 0
        int first_zero(int p, int l, int r, int ql, int qr){
            if(qr < l || r < ql) return -1;
            if(mn[p] > 0 || mx[p] < 0) return -1;
            if(l == r) return l; 
            push(p, l, r);
            int m = (l + r) / 2;
            int left = first_zero(L(p), l, m, ql, qr);
            if(left != -1) return left;
            return first_zero(R(p), m + 1, r, ql, qr);
        }
    };

    int longestBalanced(vector<int>& nums) {
        int n = nums.size();
        SegTree st(n);
        unordered_map<int,int> last;
        int ans = 0;

        for(int i = 0; i < n; i++){
            int parity = (nums[i] % 2 == 0) ? +1 : -1;
            int prev = last.count(nums[i]) ? last[nums[i]] : -1;

            st.add(1, 0, n - 1, 0, i, parity);
            if(prev != -1) st.add(1, 0, n - 1, 0, prev, -parity);

            int L = st.first_zero(1, 0, n - 1, 0, i);
            if(L != -1) ans = max(ans, i - L + 1);

            last[nums[i]] = i;
        }
        return ans;
    }
};
```

seg, lz, mn, mx are the segment tree, lazy tagging, minimum, and maximum arrays respectively.

Apply is the function that sets the values to from l to r at index p in the segment tree. Then it adds the value to mn, mx, and lz arrays.

Push is the function that propagates the lazy tag to the children both left (l, mid) and right (mid + 1, r). The _lazy_ part comes when it abdicates it responsibility of the lazy tag by passing it to the children.

Pull is the function that aggreates the values from the children back to the parent node for the segment tree, mn, and mx arrays. If there is nothing to push aka lz[p] == 0 or when the range is a single input element l == r then we return/do nothing.

Add is the function that updates the range in the segment tree. It takes the current node p, its segment boundaries [l, r], the target update range [ql, qr], and the value v to add. If the node’s range is completely outside [ql, qr], it returns. If it is fully inside, it applies the update using apply(), adding v to that segment and marking it lazy so children inherit it later. Otherwise, it calls push() to send down any pending updates, splits into left and right halves, and calls add() on each. Finally, it calls pull() to recompute the node’s values from its children. This lets large ranges update in logarithmic time.

First_zero is the function that finds the leftmost 0 in the segment tree. This function is why we needed mn and mx arrays. The `mn` array stores the minimum value in each segment, and the `mx` array stores the maximum. These two arrays let the function quickly determine whether a segment could contain a zero. If the entire range of a node is above zero or below zero, there is no need to search deeper. If zero might exist, the function calls `push()` to apply any pending lazy updates, then checks the left child first since we want the leftmost zero. If no zero is found on the left, it searches the right. When a leaf node has both `mn` and `mx` equal to zero, that index is returned as the answer. When l == r, we've ruled out segments that can't contain zero so reaching a leaf node means the element's value is exactly zero.



<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
