---
layout: default
title: Longest Balanced Substring II (LC 3714)
category: dev
published: true
---

# Prefix Sum + Balance Mapping

This problem generalizes the classic LC 525 Contiguous Array [here](https://leetcode.com/problems/contiguous-array/description/) idea from 2 to 3 symbols, revealing how balance mapping extends into higher dimensions.

Basically the way to solve this problem is to use a prefix sum and a balance mapping.

Balance mapping is a technique used to keep track of state variables that evolve *cumulatively* as you traverse a sequence. When two states are equal at i and j, it means there is a net zero change between i and j.

So frequency balance is just one type of balance. We also have sum balance, parity balance, modular balance, and even structural balance (for example, matching parentheses).

So in this problem, we are dealing with frequency balance. I need to find the longest substring where the frequency of distinct characters are equal. Since we only have 3 possible characters in our sequence, we only have 3 cases to consider:
1. The substring only contains one character.
2. The substring contains two characters, and the frequency of the two characters are equal.
3. The substring contains three characters, and the frequency of the three characters are equal.

Case 1 is trivial. The longest streak of one character is the ans.

Case 2 gives use the framework for case 3. Imagine you have two big buckets one for char 'a' and char 'b', and as you traverse the sequence, you add 1 cup of water to the corresponding bucket. Now imagine a gauge (a-b) that tells you the difference between the two buckets. Now if the gauge is X at index i and X at index j, it means that the difference a-b did not change. This implies that a went up or down by the same amount as b went up or down. Thus there is an equal number of a's and b's between i and j. To find the longest, I will just maintain a hashmap to store the first index of each gauge (a-b) reading.

Consider the empty substring that is also valid because there are 0 'a's and 'b's. So when the gauge (a-b) is 0 for the empty substring, we should add it to the hashmap[0] = -1. We seed it with -1 because when counting the length of a substring at index i where the gauge (a-b) reads 0 then we know the substring from 0 to i is valid. The length of 0 to i is i + 1. Normally to get the length we do i - hashmap[state]. So when state is 0 we do i - (-1) = i + 1.

If you encounter char 'c', then you can reset your search since case 2 only considers two characters.

Case 3 is the crux of the problem. I extended the above analogy for 3 characters and now have 3 buckets for char 'a', char 'b', and char 'c'. Now there will be two gauges (a-b) and (a-c). If frequency of a == b and frequency a == c, then this implies frequency of b == c. With 2 gauges now, we track the cumulative sum for each gauge. As I traverse the sequence, if I encounter 'a' then I add 1 to gauge (a-b) and (a-c). If I encounter 'b' then I add -1 to gauge (a-b) and nothing to gauge (a-c). If I encounter 'c' then I add -1 to gauge (a-c) and nothing to gauge (a-b). Now the state is not just (a-b) or (a-c) individually but rather (a-b, a-c). So if we see a state (X,Y) at index i and (X,Y) at index j, it means there is an equal number of a's and b's and a's and c's which implies an equal number of b's and c's.

The empty substring is valid with 0 a's, b's and c's. So we should seed the hashmap with (0,0) = -1.

C++ Code implementation:

```c++
class Solution {
public:
    int two(string& s, char a, char b){
        int n = s.size();

        int ans = 0;

        int cur = 0;

        unordered_map<int,int> last_seen;
        last_seen[0] = -1;

        for(int i = 0; i < n; i++){
            if(s[i] == a){
                cur++;
            }
            else if(s[i] == b){
                cur--;
            }
            else{
                cur = 0;
                last_seen.clear();
                last_seen[0] = i;
                continue;
            }

            if(!last_seen.count(cur)){
                last_seen[cur] = i;
            }
            else{
                ans = max(ans, i - last_seen[cur]);
            }
        }

        return ans;
    }

    int longestBalanced(string s) {
        int n = s.size();

        int ans = 0;

        int streak = 0;
        char cur = '1';

        for(int i = 0; i < n; i++){
            if(s[i] == cur){
                streak++;
            }
            else{
                cur = s[i];
                streak = 1;
            }

            ans = max(ans, streak);
        }

        ans = max(ans, two(s, 'a', 'b'));
        ans = max(ans, two(s, 'a', 'c'));
        ans = max(ans, two(s, 'b', 'c'));

        int ab = 0;
        int ac = 0;

        unordered_map<string,int> last_seen;

        last_seen["0#0"] = -1;

        for(int i = 0; i < n; i++){
            if(s[i] == 'a'){
                ab += 1;
                ac += 1;
            }
            else if(s[i] == 'b'){
                ab -= 1;
            }
            else{
                ac -= 1;
            }

            string state = to_string(ab) + "#" + to_string(ac);

            if(!last_seen.count(state)){
                last_seen[state] = i;
            }
            else{
                ans = max(ans, i - last_seen[state]);
            }
        }

        return ans;

    }
};
```

Thinking about like buckets really helped!

<script src="https://utteranc.es/client.js" repo="Ritwik7631/Ritwik7631.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
