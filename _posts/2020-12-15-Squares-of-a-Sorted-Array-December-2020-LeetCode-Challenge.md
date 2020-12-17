---
layout: post
title: December 2020 LeetCode Challenge Day 15 - Squares of a Sorted Array
---

I was chatting with a friend recently who mentioned that they were working their way through [LeetCode's December coding challenge](https://leetcode.com/discuss/general-discussion/655704/) which inspired me to check it out. I previously tricked myself into thinking that by _conducting_ interviews on a regular basis that I was keeping my coding interview skills sharp, but since the COVID-19 pandemic hit (eight? ...nine? months ago) my company hasn't been hiring. I'm probably a bit rusty. This challenge seems like a good way to force myself to spend a few minutes a day honing these skills.

Anyway, [today's challenge](https://leetcode.com/explore/challenge/card/december-leetcoding-challenge/571/week-3-december-15th-december-21st/3567/) is:

> Given an integer array `nums` sorted in **non-decreasing** order, return _an array of **the squares of each number** sorted in non-decreasing order_.

And they provide two examples:
```
Example 1:
Input: nums = [-4,-1,0,3,10]
Output: [0,1,9,16,100]
Explanation: After squaring, the array becomes [16,1,0,9,100].
After sorting, it becomes [0,1,9,16,100].

Example 2:
Input: nums = [-7,-3,2,3,11]
Output: [4,9,9,49,121]
```

I'll admit I thought this was a trivial problem until I looked at the examples and remembered that integers can be negative (like I said, it's been a while). Since squaring a negative number yields a positive number, the array exhibits a few interesting properties after squaring:

- zero serves as a pivot point (even if it doesn't appear in the input) such that...
- inputs that were originally negative, once squared, will appear in descending order at the beginning of the array (before zero)
- inputs that were originally positive, once squared, will appear in ascending order at the end of the array (after zero)

We're essentially left with two sorted sub-arrays (`[ descending .. 0 .. ascending ]`) that we need to combine into one totally ordered array.

Here's my initial implementation:

```csharp
public class Solution
{
    public int[] SortedSquares(int[] nums)
    {
        if (nums.Length == 0)
            return nums;

        // e.g. [16, 1]
        var descending = new List<int>();
        // e.g. [0, 9, 100]
        var ascending = new List<int>();

        // Square our inputs and place them into the appropriate list based on their sign
        foreach (int n in nums)
        {
            int squared = n * n;

            if (n < 0)
                descending.Add(squared);
            else
                ascending.Add(squared);
        }

        // Our input only contained positive integers, just return them
        if (descending.Count == 0)
            return ascending.ToArray();

        // Our input only contained negative integers, reverse and return them
        if (ascending.Count == 0)
        {
            descending.Reverse(); // sigh, this returns void
            return descending.ToArray();
        }

        // Start from the end of the descending list
        var descIndex = descending.Count - 1;
        // and the beginning of the ascending list
        var ascIndex = 0;

        // Merge together our two lists in ascending order
        for (int i = 0; i < nums.Length; i++)
        {
            // Determine if we've drained either of our lists
            bool ascendingRemaining = ascIndex < ascending.Count;
            bool descendingRemaining = descIndex >= 0;

            if (ascendingRemaining && descendingRemaining)
            {
                // Use the lesser of the two.
                int asc = ascending[ascIndex];
                int desc = descending[descIndex];
                
                if (asc < desc)
                {
                    nums[i] = asc;
                    ascIndex++;
                }
                else
                {
                    nums[i] = desc;
                    descIndex--;
                }
                continue;
            }

            if (ascendingRemaining)
            {
                nums[i] = ascending[ascIndex];
                ascIndex++;
            }
            else
            {
                nums[i] = descending[descIndex];
                descIndex--;
            }
        }

        return nums;
    }
}
```

And the results? Top 68% for runtime (eh okay) and top 10% for memory usage (ouch). Admittedly this solution did not aim to minimize memory utilization. Yes, it could be done in-place by swapping the descending elements around the pivot. But for spending 20 minutes on it at the end of a taxing work day and going a few years without writing any C#, I'm fairly satisfied with the results. Or, at the very least, feeling good enough about it to attempt tomorrow's challenge.
