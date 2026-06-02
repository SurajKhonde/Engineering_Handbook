# Array & String Problems — Bengaluru Backend Interview Prep

> 50 easy-to-medium problems on arrays and strings, in the style commonly asked in product/startup backend interviews in Bengaluru. Focus is on problem-solving, not FAANG-level hard. Sliding window, two-pointer, prefix-sum, hashing, and basic string manipulation dominate.

---

## 1. Subarray with Given Sum

Given an array `arr[]` containing only non-negative integers, find a continuous subarray whose sum equals a specified value `target`. Return the 1-based indices of the leftmost and rightmost elements of this subarray. Find the first such subarray.

**Note:** If no such subarray exists, return `[-1]`.

```
Input: arr[] = [1, 2, 3, 7, 5], target = 12
Output: [2, 4]
Explanation: The sum of elements from 2nd to 4th position is 12.
```

```
Input: arr[] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], target = 15
Output: [1, 5]
Explanation: The sum of elements from 1st to 5th position is 15.
```

```
Input: arr[] = [5, 3, 4], target = 2
Output: [-1]
Explanation: There is no subarray with sum 2.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^6
0 ≤ arr[i] ≤ 10^3
0 ≤ target ≤ 10^9

---

## 2. Two Sum — Pair with Given Sum

Given an array `arr[]` of integers and an integer `target`, determine if there exist two distinct elements whose sum equals `target`. Return `true` if such a pair exists, otherwise `false`.

```
Input: arr[] = [1, 4, 45, 6, 10, 8], target = 16
Output: true
Explanation: 6 + 10 = 16.
```

```
Input: arr[] = [1, 2, 4, 3, 7], target = 20
Output: false
Explanation: No two elements add up to 20.
```

**Constraints:**
2 ≤ arr.size() ≤ 10^5
-10^6 ≤ arr[i] ≤ 10^6
-10^6 ≤ target ≤ 10^6

---

## 3. Maximum Subarray Sum (Kadane's Algorithm)

Given an array `arr[]` of integers, find the contiguous subarray with the largest sum and return that sum.

```
Input: arr[] = [2, 3, -8, 7, -1, 2, 3]
Output: 11
Explanation: The subarray [7, -1, 2, 3] has the maximum sum of 11.
```

```
Input: arr[] = [-2, -4]
Output: -2
Explanation: The single element -2 gives the maximum sum.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^6
-10^4 ≤ arr[i] ≤ 10^4

---

## 4. Move All Zeroes to End

Given an array `arr[]` of integers, move all zeroes to the end while maintaining the relative order of the non-zero elements. Do it in-place.

```
Input: arr[] = [1, 2, 0, 4, 3, 0, 5, 0]
Output: [1, 2, 4, 3, 5, 0, 0, 0]
Explanation: All zeroes shifted to the end, order of non-zeros preserved.
```

```
Input: arr[] = [10, 20, 30]
Output: [10, 20, 30]
Explanation: No zeroes present, array unchanged.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
0 ≤ arr[i] ≤ 10^6

---

## 5. Rotate Array by K

Given an array `arr[]` and an integer `k`, rotate the array to the left by `k` positions.

```
Input: arr[] = [1, 2, 3, 4, 5, 6, 7], k = 2
Output: [3, 4, 5, 6, 7, 1, 2]
Explanation: Each element shifts 2 places to the left.
```

```
Input: arr[] = [1, 2, 3], k = 4
Output: [2, 3, 1]
Explanation: Rotating by 4 is the same as rotating by 1.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
0 ≤ k ≤ 10^9
0 ≤ arr[i] ≤ 10^6

---

## 6. Second Largest Element

Given an array `arr[]`, find the second largest distinct element. If it does not exist, return `-1`.

```
Input: arr[] = [12, 35, 1, 10, 34, 1]
Output: 34
Explanation: Largest is 35, second largest is 34.
```

```
Input: arr[] = [10, 10, 10]
Output: -1
Explanation: All elements are equal, no second largest.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
0 ≤ arr[i] ≤ 10^6

---

## 7. Maximum Consecutive Ones

Given a binary array `arr[]`, return the maximum number of consecutive 1s.

```
Input: arr[] = [1, 1, 0, 1, 1, 1]
Output: 3
Explanation: The last three 1s are consecutive.
```

```
Input: arr[] = [0, 0, 0]
Output: 0
Explanation: There are no 1s.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
arr[i] ∈ {0, 1}

---

## 8. Missing Number in Array

Given an array `arr[]` of size `n-1` containing distinct numbers in the range `[1, n]`, find the one missing number.

```
Input: arr[] = [1, 2, 3, 5], n = 5
Output: 4
Explanation: The number 4 is missing from the range 1 to 5.
```

```
Input: arr[] = [1, 2, 3, 4, 5, 6, 7, 8, 9], n = 10
Output: 10
Explanation: The number 10 is missing.
```

**Constraints:**
1 ≤ n ≤ 10^6
1 ≤ arr[i] ≤ n

---

## 9. Majority Element (Moore's Voting)

Given an array `arr[]`, find the element that appears more than `n/2` times. If no such element exists, return `-1`.

```
Input: arr[] = [1, 1, 2, 1, 3, 5, 1]
Output: 1
Explanation: 1 appears 4 times out of 7 elements.
```

```
Input: arr[] = [1, 2, 3]
Output: -1
Explanation: No element appears more than n/2 times.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^6
0 ≤ arr[i] ≤ 10^6

---

## 10. Best Time to Buy and Sell Stock

Given an array `prices[]` where `prices[i]` is the price of a stock on day `i`, find the maximum profit from a single buy and a later sell. If no profit is possible, return `0`.

```
Input: prices[] = [7, 1, 5, 3, 6, 4]
Output: 5
Explanation: Buy at 1, sell at 6, profit = 5.
```

```
Input: prices[] = [7, 6, 4, 3, 1]
Output: 0
Explanation: Prices only fall, no profit possible.
```

**Constraints:**
1 ≤ prices.size() ≤ 10^5
0 ≤ prices[i] ≤ 10^4

---

## 11. Longest Substring Without Repeating Characters

Given a string `s`, find the length of the longest substring without repeating characters.

```
Input: s = "abcabcbb"
Output: 3
Explanation: The longest such substring is "abc".
```

```
Input: s = "bbbbb"
Output: 1
Explanation: The longest such substring is "b".
```

**Constraints:**
0 ≤ s.length ≤ 10^5
s consists of English letters, digits, symbols, and spaces.

---

## 12. Maximum Sum Subarray of Size K

Given an array `arr[]` and an integer `k`, find the maximum sum of any contiguous subarray of size exactly `k`.

```
Input: arr[] = [100, 200, 300, 400], k = 2
Output: 700
Explanation: Subarray [300, 400] gives the max sum.
```

```
Input: arr[] = [1, 4, 2, 10, 23, 3, 1, 0, 20], k = 4
Output: 39
Explanation: Subarray [4, 2, 10, 23] gives the max sum.
```

**Constraints:**
1 ≤ k ≤ arr.size() ≤ 10^5
0 ≤ arr[i] ≤ 10^5

---

## 13. Count Distinct Elements in Every Window of Size K

Given an array `arr[]` and an integer `k`, return the count of distinct elements in every contiguous window of size `k`.

```
Input: arr[] = [1, 2, 1, 3, 4, 2, 3], k = 4
Output: [3, 4, 4, 3]
Explanation: Windows give 3, 4, 4 and 3 distinct elements respectively.
```

```
Input: arr[] = [1, 1, 1, 1], k = 2
Output: [1, 1, 1]
Explanation: Each window has only one distinct value.
```

**Constraints:**
1 ≤ k ≤ arr.size() ≤ 10^5
0 ≤ arr[i] ≤ 10^6

---

## 14. Reverse Words in a String

Given a string `s`, reverse the order of words. A word is a maximal sequence of non-space characters. Collapse multiple spaces and trim leading/trailing spaces.

```
Input: s = "the sky is blue"
Output: "blue is sky the"
Explanation: Words are reversed in order.
```

```
Input: s = "  hello   world  "
Output: "world hello"
Explanation: Extra spaces removed, words reversed.
```

**Constraints:**
1 ≤ s.length ≤ 10^4
s contains English letters, digits, and spaces.

---

## 15. Valid Anagram

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, otherwise `false`.

```
Input: s = "anagram", t = "nagaram"
Output: true
Explanation: Both strings have the same character counts.
```

```
Input: s = "rat", t = "car"
Output: false
Explanation: Character counts differ.
```

**Constraints:**
1 ≤ s.length, t.length ≤ 5 × 10^4
s and t consist of lowercase English letters.

---

## 16. First Non-Repeating Character

Given a string `s`, return the index of the first non-repeating character. If none exists, return `-1`.

```
Input: s = "leetcode"
Output: 0
Explanation: 'l' is the first character that does not repeat.
```

```
Input: s = "aabb"
Output: -1
Explanation: Every character repeats.
```

**Constraints:**
1 ≤ s.length ≤ 10^5
s consists of lowercase English letters.

---

## 17. Group Anagrams

Given an array of strings `strs[]`, group the anagrams together. Return the groups in any order.

```
Input: strs[] = ["eat", "tea", "tan", "ate", "nat", "bat"]
Output: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
Explanation: Strings with the same character set are grouped.
```

```
Input: strs[] = [""]
Output: [[""]]
Explanation: A single empty string forms one group.
```

**Constraints:**
1 ≤ strs.size() ≤ 10^4
0 ≤ strs[i].length ≤ 100
strs[i] consists of lowercase English letters.

---

## 18. Longest Common Prefix

Given an array of strings `strs[]`, find the longest common prefix. If there is no common prefix, return an empty string `""`.

```
Input: strs[] = ["flower", "flow", "flight"]
Output: "fl"
Explanation: "fl" is shared by all three strings.
```

```
Input: strs[] = ["dog", "racecar", "car"]
Output: ""
Explanation: No common prefix exists.
```

**Constraints:**
1 ≤ strs.size() ≤ 200
0 ≤ strs[i].length ≤ 200
strs[i] consists of lowercase English letters.

---

## 19. Check if String is a Palindrome

Given a string `s`, determine if it is a palindrome considering only alphanumeric characters and ignoring case.

```
Input: s = "A man, a plan, a canal: Panama"
Output: true
Explanation: After cleaning, it reads the same forwards and backwards.
```

```
Input: s = "race a car"
Output: false
Explanation: Cleaned string "raceacar" is not a palindrome.
```

**Constraints:**
1 ≤ s.length ≤ 2 × 10^5
s consists of printable ASCII characters.

---

## 20. String Compression (Run-Length)

Given a string `s`, compress it using counts of repeated characters. If the compressed string is not shorter than the original, return the original.

```
Input: s = "aaabbbcccd"
Output: "a3b3c3d1"
Explanation: Each run is replaced by character + count.
```

```
Input: s = "abcd"
Output: "abcd"
Explanation: Compression "a1b1c1d1" is longer, so original returned.
```

**Constraints:**
1 ≤ s.length ≤ 10^5
s consists of lowercase English letters.

---

## 21. Trapping Rain Water

Given an array `height[]` representing an elevation map where each bar has width 1, compute how much water can be trapped after raining.

```
Input: height[] = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
Output: 6
Explanation: 6 units of water are trapped between the bars.
```

```
Input: height[] = [4, 2, 0, 3, 2, 5]
Output: 9
Explanation: 9 units of water are trapped.
```

**Constraints:**
1 ≤ height.size() ≤ 2 × 10^4
0 ≤ height[i] ≤ 10^5

---

## 22. Container With Most Water

Given an array `height[]`, where each element is a vertical line, find two lines that together with the x-axis form a container holding the most water. Return the max area.

```
Input: height[] = [1, 8, 6, 2, 5, 4, 8, 3, 7]
Output: 49
Explanation: Lines at index 1 and 8 hold the most water.
```

```
Input: height[] = [1, 1]
Output: 1
Explanation: The only container holds area 1.
```

**Constraints:**
2 ≤ height.size() ≤ 10^5
0 ≤ height[i] ≤ 10^4

---

## 23. Merge Two Sorted Arrays

Given two sorted arrays `a[]` and `b[]`, merge them into a single sorted array.

```
Input: a[] = [1, 3, 5, 7], b[] = [0, 2, 6, 8, 9]
Output: [0, 1, 2, 3, 5, 6, 7, 8, 9]
Explanation: Both arrays merged in sorted order.
```

```
Input: a[] = [10, 12], b[] = [5, 18, 20]
Output: [5, 10, 12, 18, 20]
Explanation: Merged result is fully sorted.
```

**Constraints:**
1 ≤ a.size(), b.size() ≤ 10^5
0 ≤ a[i], b[i] ≤ 10^7

---

## 24. Sort 0s, 1s and 2s (Dutch National Flag)

Given an array `arr[]` containing only 0s, 1s, and 2s, sort it in-place without using a sorting library.

```
Input: arr[] = [0, 2, 1, 2, 0]
Output: [0, 0, 1, 2, 2]
Explanation: All 0s first, then 1s, then 2s.
```

```
Input: arr[] = [2, 2, 2, 0, 1]
Output: [0, 1, 2, 2, 2]
Explanation: Sorted in a single pass.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^6
arr[i] ∈ {0, 1, 2}

---

## 25. Two Sum II — Sorted Input

Given a sorted array `arr[]` and a `target`, return the 1-based indices of the two numbers that add up to `target`. Exactly one solution exists.

```
Input: arr[] = [2, 7, 11, 15], target = 9
Output: [1, 2]
Explanation: 2 + 7 = 9.
```

```
Input: arr[] = [2, 3, 4], target = 6
Output: [1, 3]
Explanation: 2 + 4 = 6.
```

**Constraints:**
2 ≤ arr.size() ≤ 3 × 10^4
-1000 ≤ arr[i] ≤ 1000
arr is sorted in non-decreasing order.

---

## 26. Three Sum (Triplets That Sum to Zero)

Given an array `arr[]`, find all unique triplets `[a, b, c]` such that `a + b + c = 0`.

```
Input: arr[] = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
Explanation: These are the only unique triplets summing to zero.
```

```
Input: arr[] = [0, 1, 1]
Output: []
Explanation: No triplet sums to zero.
```

**Constraints:**
3 ≤ arr.size() ≤ 3 × 10^3
-10^5 ≤ arr[i] ≤ 10^5

---

## 27. Longest Subarray with Sum K

Given an array `arr[]` (may contain negatives) and an integer `k`, find the length of the longest contiguous subarray whose sum equals `k`.

```
Input: arr[] = [10, 5, 2, 7, 1, 9], k = 15
Output: 4
Explanation: Subarray [5, 2, 7, 1] has sum 15 and length 4.
```

```
Input: arr[] = [-1, 2, 3], k = 6
Output: 0
Explanation: No subarray sums to 6.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
-10^5 ≤ arr[i] ≤ 10^5
-10^9 ≤ k ≤ 10^9

---

## 28. Subarray Sum Equals K (Count)

Given an array `arr[]` and an integer `k`, count the total number of contiguous subarrays whose sum equals `k`.

```
Input: arr[] = [1, 1, 1], k = 2
Output: 2
Explanation: [1,1] appears twice.
```

```
Input: arr[] = [1, 2, 3], k = 3
Output: 2
Explanation: [3] and [1,2] both sum to 3.
```

**Constraints:**
1 ≤ arr.size() ≤ 2 × 10^4
-1000 ≤ arr[i] ≤ 1000
-10^7 ≤ k ≤ 10^7

---

## 29. Equilibrium Index

Given an array `arr[]`, find an index where the sum of elements to its left equals the sum of elements to its right. Return the leftmost such index, or `-1`.

```
Input: arr[] = [-7, 1, 5, 2, -4, 3, 0]
Output: 3
Explanation: Left sum (-7+1+5) = right sum (-4+3+0) = -1.
```

```
Input: arr[] = [1, 2, 3]
Output: -1
Explanation: No equilibrium index exists.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^6
-10^6 ≤ arr[i] ≤ 10^6

---

## 30. Product of Array Except Self

Given an array `arr[]`, return an array `out[]` where `out[i]` is the product of all elements except `arr[i]`. Do it without division.

```
Input: arr[] = [1, 2, 3, 4]
Output: [24, 12, 8, 6]
Explanation: Each value is the product of all others.
```

```
Input: arr[] = [-1, 1, 0, -3, 3]
Output: [0, 0, 9, 0, 0]
Explanation: The single zero makes most products zero.
```

**Constraints:**
2 ≤ arr.size() ≤ 10^5
-30 ≤ arr[i] ≤ 30

---

## 31. Maximum Product Subarray

Given an array `arr[]`, find the contiguous subarray with the largest product and return that product.

```
Input: arr[] = [2, 3, -2, 4]
Output: 6
Explanation: Subarray [2, 3] has the largest product.
```

```
Input: arr[] = [-2, 0, -1]
Output: 0
Explanation: The best product attainable is 0.
```

**Constraints:**
1 ≤ arr.size() ≤ 2 × 10^4
-10 ≤ arr[i] ≤ 10

---

## 32. Find All Duplicates in an Array

Given an array `arr[]` of length `n` where each element is in `[1, n]`, return all elements that appear exactly twice.

```
Input: arr[] = [4, 3, 2, 7, 8, 2, 3, 1]
Output: [2, 3]
Explanation: 2 and 3 each appear twice.
```

```
Input: arr[] = [1, 1, 2]
Output: [1]
Explanation: Only 1 is duplicated.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
1 ≤ arr[i] ≤ arr.size()

---

## 33. Longest Consecutive Sequence

Given an unsorted array `arr[]`, find the length of the longest run of consecutive integers (order in the array does not matter).

```
Input: arr[] = [100, 4, 200, 1, 3, 2]
Output: 4
Explanation: The sequence [1, 2, 3, 4] has length 4.
```

```
Input: arr[] = [0, 3, 7, 2, 5, 8, 4, 6, 0, 1]
Output: 9
Explanation: The sequence 0..8 has length 9.
```

**Constraints:**
0 ≤ arr.size() ≤ 10^5
-10^9 ≤ arr[i] ≤ 10^9

---

## 34. Minimum Size Subarray Sum

Given an array of positive integers `arr[]` and a value `target`, find the minimal length of a contiguous subarray whose sum is ≥ `target`. If none exists, return `0`.

```
Input: arr[] = [2, 3, 1, 2, 4, 3], target = 7
Output: 2
Explanation: Subarray [4, 3] meets the target with minimal length.
```

```
Input: arr[] = [1, 1, 1, 1], target = 11
Output: 0
Explanation: No subarray reaches the target.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^5
1 ≤ arr[i] ≤ 10^4
1 ≤ target ≤ 10^9

---

## 35. Longest Substring with At Most K Distinct Characters

Given a string `s` and an integer `k`, find the length of the longest substring containing at most `k` distinct characters.

```
Input: s = "eceba", k = 2
Output: 3
Explanation: "ece" has 2 distinct characters and length 3.
```

```
Input: s = "aa", k = 1
Output: 2
Explanation: "aa" has 1 distinct character.
```

**Constraints:**
1 ≤ s.length ≤ 5 × 10^4
0 ≤ k ≤ s.length

---

## 36. Find All Anagrams in a String

Given strings `s` and `p`, return the start indices of all substrings in `s` that are anagrams of `p`.

```
Input: s = "cbaebabacd", p = "abc"
Output: [0, 6]
Explanation: "cba" at index 0 and "bac" at index 6 are anagrams of "abc".
```

```
Input: s = "abab", p = "ab"
Output: [0, 1, 2]
Explanation: "ab", "ba", and "ab" are all anagrams of "ab".
```

**Constraints:**
1 ≤ s.length, p.length ≤ 3 × 10^4
s and p consist of lowercase English letters.

---

## 37. Sort Characters by Frequency

Given a string `s`, sort it in decreasing order based on the frequency of characters.

```
Input: s = "tree"
Output: "eert"
Explanation: 'e' appears twice, so it comes before 't' and 'r'.
```

```
Input: s = "cccaaa"
Output: "aaaccc"
Explanation: Both appear 3 times; any valid frequency order is accepted.
```

**Constraints:**
1 ≤ s.length ≤ 5 × 10^5
s consists of uppercase/lowercase letters and digits.

---

## 38. Valid Parentheses

Given a string `s` containing only the characters `()[]{}`, determine if the brackets are balanced and correctly nested.

```
Input: s = "()[]{}"
Output: true
Explanation: All brackets are matched and properly closed.
```

```
Input: s = "(]"
Output: false
Explanation: Brackets do not match.
```

**Constraints:**
1 ≤ s.length ≤ 10^4
s consists only of the characters ()[]{}.

---

## 39. Implement strStr (Find Substring Index)

Given two strings `haystack` and `needle`, return the index of the first occurrence of `needle` in `haystack`, or `-1` if it is not present.

```
Input: haystack = "sadbutsad", needle = "sad"
Output: 0
Explanation: "sad" first appears at index 0.
```

```
Input: haystack = "leetcode", needle = "leeto"
Output: -1
Explanation: "leeto" does not occur in the haystack.
```

**Constraints:**
1 ≤ haystack.length, needle.length ≤ 10^4
Both consist of lowercase English letters.

---

## 40. Maximum of All Subarrays of Size K (Sliding Window Maximum)

Given an array `arr[]` and an integer `k`, return the maximum of each contiguous window of size `k`.

```
Input: arr[] = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
Output: [3, 3, 5, 5, 6, 7]
Explanation: Maximum of each sliding window of size 3.
```

```
Input: arr[] = [1, -1], k = 1
Output: [1, -1]
Explanation: Each window of size 1 is the element itself.
```

**Constraints:**
1 ≤ k ≤ arr.size() ≤ 10^5
-10^4 ≤ arr[i] ≤ 10^4

---

## 41. Set Matrix Zeroes

Given an `m × n` matrix, if an element is 0, set its entire row and column to 0. Do it in-place.

```
Input: mat = [[1, 1, 1], [1, 0, 1], [1, 1, 1]]
Output: [[1, 0, 1], [0, 0, 0], [1, 0, 1]]
Explanation: The row and column of the single 0 are zeroed out.
```

```
Input: mat = [[0, 1, 2, 0], [3, 4, 5, 2], [1, 3, 1, 5]]
Output: [[0, 0, 0, 0], [0, 4, 5, 0], [0, 3, 1, 0]]
Explanation: Every row/column containing a 0 is cleared.
```

**Constraints:**
1 ≤ m, n ≤ 200
-2^31 ≤ mat[i][j] ≤ 2^31 - 1

---

## 42. Spiral Order of a Matrix

Given an `m × n` matrix, return all its elements in spiral order.

```
Input: mat = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
Output: [1, 2, 3, 6, 9, 8, 7, 4, 5]
Explanation: Traverse the matrix in a clockwise spiral.
```

```
Input: mat = [[1, 2], [3, 4]]
Output: [1, 2, 4, 3]
Explanation: Spiral traversal of a 2×2 matrix.
```

**Constraints:**
1 ≤ m, n ≤ 10
-100 ≤ mat[i][j] ≤ 100

---

## 43. Rotate Matrix by 90 Degrees

Given an `n × n` matrix, rotate it 90 degrees clockwise in-place.

```
Input: mat = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
Output: [[7, 4, 1], [8, 5, 2], [9, 6, 3]]
Explanation: The matrix is rotated clockwise by 90 degrees.
```

```
Input: mat = [[1, 2], [3, 4]]
Output: [[3, 1], [4, 2]]
Explanation: 2×2 matrix rotated clockwise.
```

**Constraints:**
1 ≤ n ≤ 1000
-10^9 ≤ mat[i][j] ≤ 10^9

---

## 44. Merge Overlapping Intervals

Given an array of intervals where `intervals[i] = [start, end]`, merge all overlapping intervals and return the result.

```
Input: intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]
Output: [[1, 6], [8, 10], [15, 18]]
Explanation: [1,3] and [2,6] overlap and merge into [1,6].
```

```
Input: intervals = [[1, 4], [4, 5]]
Output: [[1, 5]]
Explanation: Touching intervals are merged.
```

**Constraints:**
1 ≤ intervals.size() ≤ 10^4
0 ≤ start ≤ end ≤ 10^4

---

## 45. Search in Rotated Sorted Array

Given a sorted array `arr[]` that has been rotated at some pivot, and a `target`, return its index, or `-1` if not found. Aim for O(log n).

```
Input: arr[] = [4, 5, 6, 7, 0, 1, 2], target = 0
Output: 4
Explanation: 0 is located at index 4.
```

```
Input: arr[] = [4, 5, 6, 7, 0, 1, 2], target = 3
Output: -1
Explanation: 3 is not present.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^4
-10^4 ≤ arr[i] ≤ 10^4
All values are distinct.

---

## 46. Kth Largest Element in an Array

Given an array `arr[]` and an integer `k`, return the `k`th largest element.

```
Input: arr[] = [3, 2, 1, 5, 6, 4], k = 2
Output: 5
Explanation: The 2nd largest element is 5.
```

```
Input: arr[] = [3, 2, 3, 1, 2, 4, 5, 5, 6], k = 4
Output: 4
Explanation: The 4th largest element is 4.
```

**Constraints:**
1 ≤ k ≤ arr.size() ≤ 10^5
-10^4 ≤ arr[i] ≤ 10^4

---

## 47. Find the Duplicate Number

Given an array `arr[]` of `n+1` integers where each value is in `[1, n]`, find the single repeated number without modifying the array and using O(1) extra space.

```
Input: arr[] = [1, 3, 4, 2, 2]
Output: 2
Explanation: 2 is the repeated number.
```

```
Input: arr[] = [3, 1, 3, 4, 2]
Output: 3
Explanation: 3 is the repeated number.
```

**Constraints:**
1 ≤ n ≤ 10^5
arr.size() = n + 1
1 ≤ arr[i] ≤ n

---

## 48. Longest Palindromic Substring

Given a string `s`, return the longest substring that is a palindrome.

```
Input: s = "babad"
Output: "bab"
Explanation: "aba" is also a valid answer of the same length.
```

```
Input: s = "cbbd"
Output: "bb"
Explanation: "bb" is the longest palindromic substring.
```

**Constraints:**
1 ≤ s.length ≤ 1000
s consists of lowercase and uppercase English letters.

---

## 49. Minimum Window Substring

Given strings `s` and `t`, return the smallest substring of `s` that contains all characters of `t` (including duplicates). If none exists, return `""`.

```
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
Explanation: "BANC" is the smallest window containing A, B and C.
```

```
Input: s = "a", t = "aa"
Output: ""
Explanation: s has only one 'a', so no valid window exists.
```

**Constraints:**
1 ≤ s.length, t.length ≤ 10^5
s and t consist of uppercase and lowercase English letters.

---

## 50. Count and Say / Frequency Encoding

Given an array `arr[]` of integers sorted in non-decreasing order, return an encoding as pairs of `[value, count]` for each run of equal values.

```
Input: arr[] = [1, 1, 2, 3, 3, 3]
Output: [[1, 2], [2, 1], [3, 3]]
Explanation: 1 appears twice, 2 once, 3 thrice.
```

```
Input: arr[] = [5]
Output: [[5, 1]]
Explanation: A single element appears once.
```

**Constraints:**
1 ≤ arr.size() ≤ 10^6
-10^9 ≤ arr[i] ≤ 10^9

---

*Tip: For backend interviews in Bengaluru, after solving, be ready to discuss time/space complexity, edge cases (empty input, overflow on large sums, duplicates), and why your approach beats brute force. Patterns to master: sliding window, two pointers, prefix sum + hashmap, and frequency counting — they cover the majority of these.*
