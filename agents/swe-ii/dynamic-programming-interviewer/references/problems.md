# Problem Bank: Dynamic Programming

## Easy Problems

### 1. Climbing Stairs

**Statement**: You are climbing a staircase with `n` steps. Each time you can climb 1 or 2 steps. How many distinct ways can you reach the top?

**DP Table Walkthrough**:
```
n = 5    dp[i] = number of ways to reach step i
         dp[i] = dp[i-1] + dp[i-2],  dp[0]=1, dp[1]=1

Step:    0   1   2   3   4   5
       +---+---+---+---+---+---+
dp:    | 1 | 1 | 2 | 3 | 5 | 8 |
       +---+---+---+---+---+---+
```

**Follow-ups**: What if you can climb 1, 2, or 3 steps? What if certain steps are broken?

---

### 2. House Robber

**Statement**: Given an array representing money in each house, find the maximum you can rob without robbing two adjacent houses.

**DP Table Walkthrough**:
```
nums = [2, 7, 9, 3, 1]    dp[i] = max(dp[i-1], dp[i-2] + nums[i])

Index:   0   1    2    3    4
       +---+---+----+----+----+
dp:    | 2 | 7 | 11 | 11 | 12 |
       +---+---+----+----+----+
  dp[2] = max(7, 2+9) = 11   (rob house 2)
  dp[3] = max(11, 7+3) = 11  (skip house 3)
  dp[4] = max(11, 11+1) = 12 (rob house 4)
```

**Follow-ups**: Houses in a circle (LeetCode 213)? Houses as a binary tree (LeetCode 337)?

---

## Medium Problems

### 3. Coin Change

**Statement**: Given `coins` and a total `amount`, find the fewest coins needed. Return -1 if impossible.

**DP Table Walkthrough**:
```
coins = [1, 3, 4], amount = 6    dp[i] = min coins to make amount i

Amount:  0   1   2   3   4   5   6
       +---+---+---+---+---+---+---+
dp:    | 0 | 1 | 2 | 1 | 1 | 2 | 2 |
       +---+---+---+---+---+---+---+
  dp[3] = min(dp[2], dp[0]) + 1 = 1        coins: [3]
  dp[4] = min(dp[3], dp[1], dp[0]) + 1 = 1 coins: [4]
  dp[6] = min(dp[5], dp[3], dp[2]) + 1 = 2 coins: [3,3]
```

**Why Greedy Fails**:
```
coins = [1, 3, 4], amount = 6
Greedy (largest first): 4 + 1 + 1 = 3 coins
DP:                     3 + 3     = 2 coins
```

**Follow-ups**: Reconstruct which coins were used? Count combinations instead (LeetCode 518)?

---

### 4. Longest Common Subsequence

**Statement**: Given two strings, return the length of their longest common subsequence.

**DP Table Walkthrough**:
```
text1 = "ABCDE", text2 = "ACE"
dp[i][j] = LCS of text1[0..i-1] and text2[0..j-1]

       ""  A  C  E
  ""    0  0  0  0
  A     0  1  1  1      Match:    dp[i][j] = dp[i-1][j-1] + 1
  B     0  1  1  1      Mismatch: dp[i][j] = max(dp[i-1][j], dp[i][j-1])
  C     0  1  2  2
  D     0  1  2  2
  E     0  1  2  3      Answer: 3, LCS = "ACE"

Backtracking:
  dp[5][3]=3: E==E -> include E, go to dp[4][2]
  dp[4][2]=2: D!=C -> go to dp[3][2]
  dp[3][2]=2: C==C -> include C, go to dp[2][1]
  dp[2][1]=1: B!=A -> go to dp[1][1]
  dp[1][1]=1: A==A -> include A -> LCS = "ACE"
```

**Follow-ups**: Print the actual LCS? Longest common substring (contiguous)?

---

### 5. Longest Increasing Subsequence

**Statement**: Given an integer array, find the length of the longest strictly increasing subsequence.

**DP Table Walkthrough**:
```
nums = [10, 9, 2, 5, 3, 7, 101, 18]
dp[i] = LIS length ending at index i    dp[i] = max(dp[j]+1) for j<i where nums[j]<nums[i]

Index:    0   1   2   3   4   5    6    7
nums:    10   9   2   5   3   7  101   18
       +---+---+---+---+---+---+----+----+
dp:    | 1 | 1 | 1 | 2 | 2 | 3 |  4 |  4 |
       +---+---+---+---+---+---+----+----+
  dp[3] = dp[2]+1 = 2    (2 < 5)
  dp[5] = dp[4]+1 = 3    (3 < 7, best extension)
  dp[6] = dp[5]+1 = 4    (7 < 101)
  Answer: max(dp) = 4    e.g. [2, 3, 7, 101]
```

**Follow-ups**: Reconstruct the subsequence? O(n log n) with binary search?

---

## Hard Problems

### 6. 0/1 Knapsack

**Statement**: Given items with weights and values and a capacity `W`, find the maximum value without exceeding capacity. Each item used at most once.

**DP Table Walkthrough**:
```
weights = [1, 3, 4, 5], values = [1, 4, 5, 7], W = 7
dp[i][w] = max value using first i items with capacity w

       w=0  w=1  w=2  w=3  w=4  w=5  w=6  w=7
  0     0    0    0    0    0    0    0    0
  1     0    1    1    1    1    1    1    1
  2     0    1    1    4    5    5    5    5
  3     0    1    1    4    5    6    6    9
  4     0    1    1    4    5    7    8    9

  dp[3][7] = max(dp[2][7], dp[2][3]+5) = max(5, 9) = 9   (take item 3)
  Answer: 9
```

**Space Optimization**: Single 1D array, iterate capacity in reverse.

**Follow-ups**: Unbounded knapsack (unlimited items)? Subset sum (boolean variant)?

---

### 7. Edit Distance

**Statement**: Given two strings, return the minimum operations (insert, delete, replace) to convert one to the other.

**DP Table Walkthrough**:
```
word1 = "horse", word2 = "ros"
dp[i][j] = edit distance of word1[0..i-1] and word2[0..j-1]

       ""  r  o  s
  ""    0  1  2  3     Match:    dp[i][j] = dp[i-1][j-1]
  h     1  1  2  3     Mismatch: dp[i][j] = 1 + min(
  o     2  2  1  2         dp[i-1][j],      delete
  r     3  2  2  2         dp[i][j-1],      insert
  s     4  3  3  2         dp[i-1][j-1]     replace
  e     5  4  4  3     )

  dp[2][2]: o==o -> dp[1][1] = 1
  dp[5][3]: e!=s -> 1 + min(2, 4, 2) = 3
  Answer: 3   (horse -> rorse -> rose -> ros)
```

**Follow-ups**: Reconstruct the operation sequence? Different costs per operation? Applications in spell checking and DNA alignment.

---

## Session Flows

### Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "What makes a problem suitable for DP?"
**Candidate**: "I'm not really sure... when you need to optimize something?"
**Interviewer**: "Two key properties: optimal substructure and overlapping subproblems. Let me show you."

*[Start with Climbing Stairs, walk through the framework step by step]*

### Flow 2: Candidate Excels (Hard Path)

**Interviewer**: "What is the difference between memoization and tabulation?"
**Candidate**: "Memoization is top-down with a cache. Tabulation is bottom-up. Tabulation avoids stack overhead but memoization only computes needed subproblems."

*[Jump to LCS, then challenge with space optimization and Edit Distance]*

### Flow 3: Average Candidate (Standard Path)

**Interviewer**: "Can you explain overlapping subproblems with an example?"
**Candidate**: "Like Fibonacci, where fib(5) calls fib(3) in two different branches."

*[Start with Coin Change, apply the four-step framework together, move to LIS if time]*
