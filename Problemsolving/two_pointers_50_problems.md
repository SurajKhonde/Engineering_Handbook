# Two Pointers Practice Set (50) — Easy + Medium
> Each problem includes **Goal**, **Sample input**, **Sample output**.
> Use patterns: **left/right**, **fast/slow**, **merge**, **partition**, **sliding window**.

## 1. Reverse Array In-Place (Easy)
**Goal:** Reverse the array using two pointers.

**Sample input:**
```text
[1,2,3,4]
```
**Sample output:**
```text
[4,3,2,1]
```

---

## 2. Reverse String (Easy)
**Goal:** Reverse a string in-place.

**Sample input:**
```text
hello
```
**Sample output:**
```text
olleh
```

---

## 3. Reverse Only Vowels (Easy)
**Goal:** Reverse only vowels; keep other chars in place.

**Sample input:**
```text
leetcode
```
**Sample output:**
```text
leotcede
```

---

## 4. Reverse Only Letters (Easy)
**Goal:** Reverse only letters; keep digits/punctuation in place.

**Sample input:**
```text
a-bC-dEf-ghIj
```
**Sample output:**
```text
j-Ih-gfE-dCba
```

---

## 5. Check Palindrome String (Easy)
**Goal:** Return YES if string is palindrome (case-sensitive).

**Sample input:**
```text
racecar
```
**Sample output:**
```text
YES
```

---

## 6. Check Palindrome Alphanumeric (Easy)
**Goal:** Ignore non-alphanumeric and case.

**Sample input:**
```text
A man, a plan, a canal: Panama
```
**Sample output:**
```text
YES
```

---

## 7. Remove Duplicates From Sorted Array (Easy)
**Goal:** Return new length after removing duplicates in-place.

**Sample input:**
```text
[1,1,2,2,3]
```
**Sample output:**
```text
len=3, arr=[1,2,3,...]
```

---

## 8. Remove Element (In-Place) (Easy)
**Goal:** Remove all occurrences of val; return new length.

**Sample input:**
```text
arr=[3,2,2,3], val=3
```
**Sample output:**
```text
len=2, arr=[2,2,...]
```

---

## 9. Move Zeros (Easy)
**Goal:** Move all 0s to end while keeping order of non-zeros.

**Sample input:**
```text
[0,1,0,3,12]
```
**Sample output:**
```text
[1,3,12,0,0]
```

---

## 10. Partition by Parity (Easy)
**Goal:** Move all even numbers before odd numbers (order not required).

**Sample input:**
```text
[3,1,2,4]
```
**Sample output:**
```text
[4,2,1,3] (any valid)
```

---

## 11. Sort Colors (0/1) (Easy)
**Goal:** Sort array containing only 0 and 1.

**Sample input:**
```text
[1,0,1,0,1]
```
**Sample output:**
```text
[0,0,1,1,1]
```

---

## 12. Two Sum in Sorted Array (Easy)
**Goal:** Given sorted array, return indices (1-based) of two numbers summing to target.

**Sample input:**
```text
arr=[2,7,11,15], target=9
```
**Sample output:**
```text
[1,2]
```

---

## 13. Pair With Given Difference (Sorted) (Easy)
**Goal:** Check if there exists i<j with arr[j]-arr[i]=d.

**Sample input:**
```text
arr=[1,3,5,8], d=2
```
**Sample output:**
```text
YES
```

---

## 14. Merge Two Sorted Arrays (Easy)
**Goal:** Merge two sorted arrays into one sorted array.

**Sample input:**
```text
a=[1,3,5], b=[2,4,6]
```
**Sample output:**
```text
[1,2,3,4,5,6]
```

---

## 15. Squares of Sorted Array (Easy)
**Goal:** Return sorted squares of a sorted array (can have negatives).

**Sample input:**
```text
[-4,-1,0,3,10]
```
**Sample output:**
```text
[0,1,9,16,100]
```

---

## 16. Valid Mountain Array (Easy)
**Goal:** Return YES if array strictly increases then strictly decreases.

**Sample input:**
```text
[0,3,2,1]
```
**Sample output:**
```text
YES
```

---

## 17. Backspace String Compare (Easy)
**Goal:** Compare two strings with # as backspace.

**Sample input:**
```text
s=ab#c, t=ad#c
```
**Sample output:**
```text
YES
```

---

## 18. Remove Adjacent Duplicates (Easy)
**Goal:** Repeatedly remove adjacent duplicate pairs.

**Sample input:**
```text
abbaca
```
**Sample output:**
```text
ca
```

---

## 19. Is Subsequence (Easy)
**Goal:** Return YES if s is a subsequence of t.

**Sample input:**
```text
s=abc, t=ahbgdc
```
**Sample output:**
```text
YES
```

---

## 20. Container With Most Water (Medium)
**Goal:** Max water area between lines using two pointers.

**Sample input:**
```text
[1,8,6,2,5,4,8,3,7]
```
**Sample output:**
```text
49
```

---

## 21. 3Sum (Unique Triplets) (Medium)
**Goal:** Return unique triplets that sum to 0.

**Sample input:**
```text
[-1,0,1,2,-1,-4]
```
**Sample output:**
```text
[[-1,-1,2],[-1,0,1]]
```

---

## 22. 3Sum Closest (Medium)
**Goal:** Return sum of triplet closest to target.

**Sample input:**
```text
arr=[-1,2,1,-4], target=1
```
**Sample output:**
```text
2
```

---

## 23. 4Sum (Unique Quadruplets) (Medium)
**Goal:** Return unique quadruplets summing to target.

**Sample input:**
```text
arr=[1,0,-1,0,-2,2], target=0
```
**Sample output:**
```text
[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]
```

---

## 24. Remove Duplicates II (At Most Twice) (Medium)
**Goal:** From sorted array, keep each element at most twice.

**Sample input:**
```text
[0,0,1,1,1,1,2,3,3]
```
**Sample output:**
```text
len=7, arr=[0,0,1,1,2,3,3,...]
```

---

## 25. Minimum Size Subarray Sum (Medium)
**Goal:** Smallest length subarray with sum >= target (positive nums).

**Sample input:**
```text
arr=[2,3,1,2,4,3], target=7
```
**Sample output:**
```text
2
```

---

## 26. Count Subarrays Product < K (Medium)
**Goal:** Count subarrays with product < k (positive nums).

**Sample input:**
```text
arr=[10,5,2,6], k=100
```
**Sample output:**
```text
8
```

---

## 27. Longest Substring Without Repeating (Medium)
**Goal:** Return length of longest substring without repeating chars.

**Sample input:**
```text
abcabcbb
```
**Sample output:**
```text
3
```

---

## 28. Longest Repeating Char Replacement (Medium)
**Goal:** Longest substring after replacing at most k chars to same char.

**Sample input:**
```text
s=AABABBA, k=1
```
**Sample output:**
```text
4
```

---

## 29. Minimum Window Substring (Medium)
**Goal:** Smallest substring of s containing all chars of t.

**Sample input:**
```text
s=ADOBECODEBANC, t=ABC
```
**Sample output:**
```text
BANC
```

---

## 30. Valid Palindrome II (Medium)
**Goal:** Return YES if string can be palindrome after deleting at most one char.

**Sample input:**
```text
abca
```
**Sample output:**
```text
YES
```

---

## 31. Trapping Rain Water (Medium)
**Goal:** Compute trapped water between bars.

**Sample input:**
```text
[0,1,0,2,1,0,1,3,2,1,2,1]
```
**Sample output:**
```text
6
```

---

## 32. Dutch National Flag (0,1,2) (Medium)
**Goal:** Sort array of 0,1,2 in one pass (two pointers + mid).

**Sample input:**
```text
[2,0,2,1,1,0]
```
**Sample output:**
```text
[0,0,1,1,2,2]
```

---

## 33. Intersection of Two Sorted Arrays (Easy)
**Goal:** Return common elements (with multiplicity).

**Sample input:**
```text
a=[1,2,2,3], b=[2,2,4]
```
**Sample output:**
```text
[2,2]
```

---

## 34. Intersection of Two Arrays (Unique) (Easy)
**Goal:** Return unique common elements (use two pointers on sorted).

**Sample input:**
```text
a=[4,9,5], b=[9,4,9,8,4]
```
**Sample output:**
```text
[4,9]
```

---

## 35. Check If Pair Sum Exists (Unsorted) (Easy)
**Goal:** Return YES if any pair sums to target (sort + two pointers).

**Sample input:**
```text
arr=[3,5,1,7], target=8
```
**Sample output:**
```text
YES
```

---

## 36. Count Pairs With Sum K (Sorted) (Medium)
**Goal:** Count number of pairs that sum to k (array sorted).

**Sample input:**
```text
arr=[1,1,2,2,3,4], k=4
```
**Sample output:**
```text
3
```

---

## 37. Triplet Sum Exists (Medium)
**Goal:** Return YES if any triplet sums to target (sort + 2-sum).

**Sample input:**
```text
arr=[12,3,4,1,6,9], target=24
```
**Sample output:**
```text
YES
```

---

## 38. Closest Pair From Two Sorted Arrays (Medium)
**Goal:** Pick one element from each array to minimize |a-b|.

**Sample input:**
```text
a=[1,4,5,7], b=[10,20,30,40]
```
**Sample output:**
```text
(7,10)
```

---

## 39. Maximize Sum of Pair With Constraint (Medium)
**Goal:** Find max a[i]+a[j] with i<j and a[i]<a[j] (two pointers after sorting pairs).

**Sample input:**
```text
[3,5,2,8]
```
**Sample output:**
```text
13
```

---

## 40. Boats to Save People (Medium)
**Goal:** Min boats with limit; each boat up to 2 people.

**Sample input:**
```text
people=[3,2,2,1], limit=3
```
**Sample output:**
```text
3
```

---

## 41. Assign Cookies (Easy)
**Goal:** Max content children with greed factors and cookie sizes.

**Sample input:**
```text
g=[1,2,3], s=[1,1]
```
**Sample output:**
```text
1
```

---

## 42. Minimum Platforms (Two pointers on sorted times) (Medium)
**Goal:** Given arrivals/departures, min platforms needed.

**Sample input:**
```text
arr=[900,940,950], dep=[910,1200,1120]
```
**Sample output:**
```text
2
```

---

## 43. Merge Intervals (Sort + Two pointers scan) (Medium)
**Goal:** Merge overlapping intervals.

**Sample input:**
```text
[[1,3],[2,6],[8,10],[15,18]]
```
**Sample output:**
```text
[[1,6],[8,10],[15,18]]
```

---

## 44. Find Pair Closest to Target (Easy)
**Goal:** Return pair sum closest to target (sorted).

**Sample input:**
```text
arr=[1,4,6,8], target=10
```
**Sample output:**
```text
(4,6)
```

---

## 45. Find Pair With Sum Closest to 0 (Medium)
**Goal:** Return pair with sum closest to 0.

**Sample input:**
```text
arr=[1,60,-10,70,-80,85]
```
**Sample output:**
```text
(-80,85)
```

---

## 46. Max Consecutive Ones After Flipping K Zeros (Medium)
**Goal:** Longest subarray with at most k zeros.

**Sample input:**
```text
arr=[1,1,1,0,0,0,1,1,1,1,0], k=2
```
**Sample output:**
```text
6
```

---

## 47. Longest Subarray With At Most Two Distinct (Medium)
**Goal:** Length of longest subarray with <=2 distinct numbers.

**Sample input:**
```text
[1,2,1,2,3]
```
**Sample output:**
```text
4
```

---

## 48. Minimum Swaps to Group All 1s Together (Medium)
**Goal:** Min swaps to bring all 1s together (window).

**Sample input:**
```text
[1,0,1,0,1]
```
**Sample output:**
```text
1
```

---

## 49. Remove All Occurrences of Substring (Easy)
**Goal:** Remove all occurrences of pattern p from s (stack/two-pointer scan).

**Sample input:**
```text
s=abcxxabcxx, p=abc
```
**Sample output:**
```text
xxxx
```

---

## 50. Compare Version Numbers (Two pointers) (Medium)
**Goal:** Compare version strings; return -1/0/1.

**Sample input:**
```text
v1=1.01, v2=1.001
```
**Sample output:**
```text
0
```

---

