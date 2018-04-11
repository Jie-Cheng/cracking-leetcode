---
layout: post
title: Backtracking
data: 2018-04-10
img: search_tree.png
tags: [Backtracking, DFS]
---

According to Wikipedia, backtracking is a general algorithm for finding all (or some) solutions to some computational problems that incrementally builds candidates to the solutions, and abandons a candidate ("backtrack") as soon as it determines that the candidate is not valid. Indeed, backtracking is essentially a brute-force algorithm with a bit more consideration: constuct a search tree where you enumerate all the possible solutions one by one, stop as early as possible if you find a candidate is invalid. Depth-first search with recursion is often used to enumerate the candidates.

## Permutation
One type of problems that backtracking is suitable to solve is permutation. Take [LeetCode 46](https://leetcode.com/problems/permutation/description/) for example: find all permutations of a set of distinct numbers. We know that every permutation is a rearrangement of the number set, so the number at any digit can be any number in the set that has not been used. The enumeration process is straight-forward: we determine the digits one by one, try every number that has not been used at every position, once we have determined all the digits, we have one valid solution. Conceptually we construct a tree where the first level are the n choices for the first digit, every node further lead to n-1 branches which are at the second level etc. The code is as follows:

{% highlight cpp %}
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> res;
        vector<int> current;
        vector<bool> used(nums.size(), false);
        helper(res, current, nums, used);
        return res;
    }
private:
    void helper(vector<vector<int>>& res, vector<int>& current, vector<int>& nums, vector<bool>& used) {
        if (current.size() == nums.size()) {
            res.push_back(current);
            return;
        }
        for (int i = 0; i < nums.size(); ++i) {
            if (used[i]) continue;
            used[i] = true;
            current.push_back(nums[i]);
            helper(res, current, nums, used);
            used[i] = false;
            current.pop_back();
        }
    }
};
{% endhighlight %}

This example barely "backtracks" except that we avoid using the numbers. [LeetCode 47](https://leetcode.com/problems/permutations-ii/description/) asks for permutations of a set that contains duplicate numbers. For example, given \[1, 1, 2\] we should return \[\[1, 1, 2\], \[1, 2, 1\], \[2, 1, 1\]\]. We can definitely generate all the permutations as if all numbers are distinct and then get rid of the duplicate answers. But a smarter way would be checking if we are searching along the wrong branch and backtracking if that is the case. It turns out we can do this by just modifying one line of code:

{% highlight cpp %}
if (used[i] || (i > 0 && nums[i] == nums[i-1] && !used[i-1])) continue;
{% endhighlight %}

This is to say, at each level there can be only one instance of a specific number. The `!used[i-1]` indicates `nums[i-1]` hasn't been used in this candidate, which means it is on a sibling branch, then we should stop searching further. Note that we have to sort the numbers beforehand so that duplicate numbers are next to each other.

## Combination
Combination can also be solved with backtracking. In [LeetCode 77](https://leetcode.com/problems/combinations/description/) we are asked to list all the k-combinations out of n. The enumeartion process is different from permutation. Instead of considering all the numbers at every position, we only consider the numbers in a specific order. Because every number can either be in the solution or not, and the order of the numbers in the solution does not matter, there is no point considering a number again if we have already made the decision to include it or not before. So we need an indicator to mark the starting point of the candidates.

{% highlight cpp %}
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> res;
        vector<int> current;
        helper(res, current, n, k, 1);
        return res;
    }
private:
    void helper(vector<vector<int>>& res, vector<int>& current, int n, int k, int index) {
        if (k == 0) {
            res.push_back(current);
            return;
        }
        for (int i = index; i <= n; ++i) {
            current.push_back(i);
            helper(res, current, n, k-1, i+1);
            current.pop_back();
        }
    }
};
{% endhighlight %}

## Combination sum and subsets
These two examples shows the basic techniques of backtracking. There are many variations of them. For instance combination sum with or without duplicate numbers, which we have to add the additional logics to check if a number is too large to contribute to the sum. For another instance the subsets problems, where you need to list all the subsets of a set. In this problem, you don't need to check whether the size of current solution or the starting index: any solution candidate that is passed to the helper function is valid.

## Advanced tricks
Sometimes it is worth memorizing the intermediate results in a set or a map so that next time the same subproblem appears we do not need to do the work again. A good example is [LeetCode 294](https://leetcode.com/problems/flip-game-ii/description/).


{% highlight cpp %}
class Solution {
public:
    bool canWin(string s) {
        unordered_map<string, bool> memo;
        return canWin(s, memo);
    }
private:
    bool canWin(string& s, unordered_map<string, bool>& memo) {
        if (memo.find(s) != memo.end()) return memo[s];
        for (int i = 0; i + 1 < s.size(); ++i) {
            if (s[i] == '+' && s[i+1] == '+') {
                auto t(s);
                t[i] = t[i+1] = '-';
                if (!canWin(t, memo)) return memo[s] = true;
            }
        }
        return memo[s] = false;
    }
};
{% endhighlight %}

Another trick is binary mask. In backtracking problems, one variable often has two exclusive states, which makes binary representation natural. For example in the subsets problem, for any element, we either include it in a specific subset or not. Therefore, there are 2^n subsets. Every j in the range of [0, 2^n) represents a subset: if and only if the i-th digit is set then we include nums[i].

{% highlight cpp %}
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();
        int nsets = (1 << n);
        vector<vector<int>> res(nsets, vector<int>());
        for (int j = 0; j < nsets; ++j)
            for (int i = 0; i < n; ++i)
                if ((j >> i) & 1)
                    res[j].push_back(nums[i]);
        return res;
    }
};
{% endhighlight %}

Finally, backtracking algorithms normally do not have a nice time complexity, the worst case could easily be the same as brute-force which is exponential. Ask yourself a question before applying them: are these different subproblems related to each other? If one subproblem can be built on a previous one, then maybe dynamic programming can be used!
