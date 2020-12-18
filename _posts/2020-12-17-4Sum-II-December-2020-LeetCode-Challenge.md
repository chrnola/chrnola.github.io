---
layout: post
title: December 2020 LeetCode Challenge Day 17 - 4Sum II
---

[Today's challenge](https://leetcode.com/explore/challenge/card/december-leetcoding-challenge/571/week-3-december-15th-december-21st/3569/) is:
> Given four lists A, B, C, D of integer values, compute how many tuples `(i, j, k, l)` there are such that `A[i] + B[j] + C[k] + D[l]` is zero.
>
> To make problem a bit easier, all A, B, C, D have same length of N where 0 ≤ N ≤ 500. All integers are in the range of -2^28 to 2^28 - 1 and the result is guaranteed to be at most 2^31 - 1.

# First Attempt

The first thing that comes to mind here is the naive, brute force approach in which we enumerate all of the possible tuples and compute their sums.
I already know that there's no chance that such a solution is going to be accepted due to its exponential runtime, but it's a decent excuse to practice writing a recursive algorithm, so I did it anyway.

The general strategy here is to design a recursive function that will:

- Take in a list-of-lists that have yet to be visited as well as a running total for the current "path" through the lists.
- Recursively accumulate and return the number of tuples at every level that have been found to sum to 0.
- In the base case in which a tuple has been formed using inputs from all 4 lists, return 1 if said tuple sums to zero, and otherwise 0.

```csharp
public class Solution
{
    private int sumAllCombinations(int sumSoFar, List<int[]> listsToGo)
    {
        if (listsToGo.Count == 0)
            // If there are no more lists to process,
            // return 1 if the final sum == 0, otherwise 0.
            return sumSoFar == 0 ? 1 : 0;

        // Pop the next list off the head of the list-of-lists
        var nextList = listsToGo[0];
        listsToGo.RemoveAt(0);

        // Set up an accumulator for all the paths.
        int acc = 0;

        // When we come to a fork in the road...take it.
        foreach (int n in nextList)
            acc += sumAllCombinations(sumSoFar + n, new List<int[]>(listsToGo));

        return acc;
    }

    public int FourSumCount(int[] A, int[] B, int[] C, int[] D)
    {
        return sumAllCombinations(0, new List<int[]>(new[]{A, B, C, D}));
    }
}
```

As expected, it produces correct outputs, but is slow. So slow that LeetCode won't accept it.

# Second Attempt

As I started tracing through the path of execution for my initial attempt I realized that one of the most wasteful parts was the fact that it repeatedly computed the same sums for the lower layers.
In other words, the sum of `B[1]`, `C[6]`, and `D[25]` would always be the same, regardless of which element of `A` we started on.
That led me to thinking of pre-computing sums that could be reused later on.

I decided to write a function that would produce a map of all of the sums from a pair of input lists to their occurrence counts.
Applying this function to lists A+B and then C+D would leave us with two maps of complementary sums.
At that point we would just need to iterate over one of them and check for each element's complement (the element * -1) in the other map.
Since the value of these maps represents the number of times that a particular sum was found in a pair of input lists, we would multiply these values together when we found a complementary pair, and accumulate those products to produce our final answer.

```csharp
public class Solution
{
    private Dictionary<int, int> buildSumMap(int[] one, int[] two)
    {
        var map = new Dictionary<int, int>();

        foreach (int i in one)
        {
            foreach (int j in two)
            {
                var sum = i + j;

                int current = 0;
                map.TryGetValue(sum, out current);
                map[sum] = current + 1;
            }
        }

        return map;
    }

    private int findMatches(Dictionary<int, int> AB, Dictionary<int, int> CD)
    {
        int acc = 0;
        foreach (KeyValuePair<int, int> ab in AB)
        {
            int complement = -ab.Key;
            int numComplements = 0;
            CD.TryGetValue(complement, out numComplements);
            acc += ab.Value * numComplements;
        }
        return acc;
    }

    public int FourSumCount(int[] A, int[] B, int[] C, int[] D)
    {
        var AB = buildSumMap(A, B);
        var CD = buildSumMap(C, D);

        return findMatches(AB, CD);
    }
}
```

Success! This time the implementation was accepted.

I was curious about the effective difference in execution times between the two solutions, so I ran them both locally with a modest sized input (N=100) and the results were, well, not that surprising: 2945ms vs. 1ms on an Intel Core i7-9700K.
