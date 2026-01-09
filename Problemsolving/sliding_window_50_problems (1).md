# Sliding Window Problems (Easy to Medium)

This document contains **50 Sliding Window problems** formatted similar
to HackerRank style, with **Problem Statement**, **Input**, **Output**,
and **Sample Input/Output**.

------------------------------------------------------------------------

## 1. Maximum Sum Subarray of Size K

**Problem Statement:**\
Given an array of integers and an integer K, find the maximum sum of any
contiguous subarray of size K.

**Input:**\
First line: N and K\
Second line: N integers

**Output:**\
Maximum sum

**Sample Input:**\
5 3\
2 1 5 1 3

**Sample Output:**\
9
**solution**
```js
function maximumSum(arr,k){
    let maxSum =0;
    let windowSum =0;
    for(let i=0;i<k;i++){
        windowSum+=arr[i];
    };
    maxSum=windowSum ;
    for(let i=k;i<arr.length;i++){
        windowSum +=arr[i];
        windowSum -=arr[i-k];
        if(windowSum >maxSum) maxSum=windowSum
    }
   return maxSum;
   
}
```
------------------------------------------------------------------------

## 2. Minimum Sum Subarray of Size K

**Problem Statement:**\
Find the minimum sum of any contiguous subarray of size K.

**Input:**\
6 2\
3 1 2 6 4 2

**Output:**\
3

**Sample Output:**\
3
**solution**
```js
function minSum(arr,k){
    let minSum=0;
    let windowMinSum =0;
    for(let i=0;i<k;i++){
        windowSum+=arr[i];

    }
    minSum =windowSum;
    for(let i=k;i<arr.length;i++){
        windowSum +=arr[i];
        windowSum -=arr[i-k]
        if(windowSum<minSum) minSum=windowSum
    }
    return minSum
}
```
------------------------------------------------------------------------

## 3. Average of Subarrays

**Problem Statement:**\
Print the average of all contiguous subarrays of size K.

**Input:**\
5 2\
1 2 3 4 5

**Output:**\
1.5 2.5 3.5 4.5
**Solution**
```JS
function avrageSubArray(arr, k) {
    let totalAvr = [];
    let windowSum = 0;
    for (let i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    totalAvr.push(windowSum / k);
    for (let i = k; i < arr.length; i++) {
        windowSum += arr[i];
        windowSum -= arr[i - k]; 
        totalAvr.push(windowSum / k);
    }

    return totalAvr;
}

console.log(avrageSubArray([1,2,3,4,5], 2));
// [1.5, 2.5, 3.5, 4.5]

``` 
------------------------------------------------------------------------

## 4. Maximum Element in Window

**Problem Statement:**\
Find the maximum element in every window of size K.

**Input:**\
8 3\
1 3 -1 -3 5 3 6 7

**Output:**\
3 3 5 5 6 7

```js
function maximumElem(arr, k) {
    let result = [];
    let window = [];
    for (let i = 0; i < k; i++) {
        window.push(arr[i]);
    }
    result.push(Math.max(...window));
    for (let i = k; i < arr.length; i++) {
        window.shift();  
        window.push(arr[i]); 
        result.push(Math.max(...window));
    }

    return result;
}

```
------------------------------------------------------------------------

## 5. Minimum Element in Window
**Problem Statement:**\
Find the minimum element in every window of size K.

**Input:**\
5 2\
4 2 1 3 5

**Output:**\
2 1 1 3
**solution**
```js
function minimumElem(arr,k){
    let minumElem =[];
    let window =[];
    for(let i=0;i<k;i++){
        window.push(arr[i]);
    };
    minumElem.push(Math.min(...window));
    for(let i=k;i<arr.length;i++){
        window.shift();
        window.push(arr[i]);
        minumElem.push(Math.min(...window));
    };
    return minumElem ;
     
}
```
------------------------------------------------------------------------

## 6. Count Subarrays with Sum K

**Problem Statement:**\
Count contiguous subarrays whose sum equals K.

**Input:**\
5 5\
1 2 3 2 2

**Output:**\
2
```js
function countSubarraysWithSum(arr,k){
    let windowstart =0;
    let count=0;
    let windowSum =0;
    for(let i=0;i<arr.length;i++){
        windowSum+=arr[i];
        while(windowSum >k){
            windowSum -=[arr[windowStart]]
            windowStart ++
        }
        if(windowSum === k){
            count ++
        }
        
    }
    return count ;
};
```
------------------------------------------------------------------------

## 7. Longest Subarray with Sum \<= K
**Problem Statement:**\
Find the length of the longest subarray with sum less than or equal to
K.

**Input:**\
6 7\
2 1 5 1 3 2

**Output:**\
3
**solution**
```js
function LongestSubArray(arr, k) {
    let startWindow = 0;
    let windowSum = 0;
    let maxLength = 0;

    for (let end = 0; end < arr.length; end++) {
        windowSum += arr[end];
        while (windowSum > k) {
            windowSum -= arr[startWindow];
            startWindow++;
        }

        maxLength = Math.max(maxLength, end - startWindow + 1);
    }

    return maxLength;
}
```
------------------------------------------------------------------------

## 8. Smallest Subarray with Sum \>= K

**Problem Statement:**\
Find the length of the smallest contiguous subarray with sum greater
than or equal to K.

**Input:**\
6 8\
2 3 1 2 4 3

**Output:**\
2
**solution**
```js
function smallestSubArraywithSum(arr, k) {
  let start = 0;
  let sum = 0;
  let minLen = Infinity;
  for (let end = 0; end < arr.length; end++) {
    sum += arr[end];
    while (sum >= k) {
      minLen = Math.min(minLen, end - start + 1);
      sum -= arr[start];
      start++;
    }
  }
  return minLen === Infinity ? 0 : minLen;
}
```
------------------------------------------------------------------------

## 9. Maximum Number of Vowels in Substring

**Problem Statement:**\
Find the maximum number of vowels in any substring of size K.

**Input:**\
abciiidef 3

**Output:**\
3
```js
function maximumNumberOfVowels(str, k) {
  const vowels = new Set(["a", "e", "i", "o", "u"]);
  let windowCount = 0;
  for (let i = 0; i < k && i < str.length; i++) {
    if (vowels.has(str[i])) windowCount++;
  }
  let maxCount = windowCount;
  for (let i = k; i < str.length; i++) {
    if (vowels.has(str[i])) windowCount++; 
    if (vowels.has(str[i - k])) windowCount--; 

    if (windowCount > maxCount) maxCount = windowCount;
  }

  return maxCount;
}
```
------------------------------------------------------------------------

## 10. Longest Substring Without Repeating Characters

**Problem Statement:**\
Find the length of the longest substring without repeating characters.

**Input:**\
abcabcbb

**Output:**\
3
**solution**
```js
function longestSubString(str){
    let seen =new Set();
    let start=0
    let maxSize = 0 ;
    for (let i=0;i<str.length;i++){
        while(seen.has(str[i])){
            seen.delete(str[start])
            start ++;
        }else {
            seen.add(str[i]);
            maxSize=Math.max(maxSize,i-start + 1);
        }
    }
    return maxSize;

}
```
------------------------------------------------------------------------

## 11. Longest Substring with At Most K Distinct Characters

**Problem Statement:**\
Find the length of the longest substring containing at most K distinct
characters.

**Input:**\
araaci 2

**Output:**\
4
**solution**
```js
function longestSubstringAtMostKDistinct(str, k) {
  if (k <= 0) return 0;
  let start = 0;
  let maxLen = 0;
  const freq = new Map();
  for (let end = 0; end < str.length; end++) {
    const ch = str[end];
    freq.set(ch, (freq.get(ch) || 0) + 1);
    while (freq.size > k) {
      const leftChar = str[start];
      freq.set(leftChar, freq.get(leftChar) - 1);
      if (freq.get(leftChar) === 0) freq.delete(leftChar);
      start++;
    }

    maxLen = Math.max(maxLen, end - start + 1);
  }

  return maxLen;
}
```
------------------------------------------------------------------------

## 12. Longest Substring with Exactly K Distinct Characters

**Problem Statement:**\
Find the length of the longest substring containing exactly K distinct
characters.

**Input:**\
aabacbebebe 3

**Output:**\
7
**solution**
```js
function longestSubstringExactlyKDistinct(str, k) {
  if (k <= 0) return 0;

  let start = 0;
  let maxLen = 0;
  const freq = new Map();

  for (let end = 0; end < str.length; end++) {
    const ch = str[end];
    freq.set(ch, (freq.get(ch) || 0) + 1);

    while (freq.size > k) {
      const left = str[start];
      freq.set(left, freq.get(left) - 1);
      if (freq.get(left) === 0) freq.delete(left);
      start++;
    }
    if (freq.size === k) {
      maxLen = Math.max(maxLen, end - start + 1);
    }
  }

  return maxLen;
};

```
------------------------------------------------------------------------

## 13. Fruits into Baskets

**Problem Statement:**\
Pick at most two types of fruits and find the maximum number collected.

**Input:**\
ABAC

**Output:**\
3
**solution**
```js
function maxFruitInBasket(str) {
  let start = 0;
  let maxLen = 0;
  const freq = new Map();
  for (let end = 0; end < str.length; end++) {
    const fruit = str[end];
    freq.set(fruit, (freq.get(fruit) || 0) + 1);
    while (freq.size > 2) {
      const leftFruit = str[start];
      freq.set(leftFruit, freq.get(leftFruit) - 1);
      if (freq.get(leftFruit) === 0) freq.delete(leftFruit);
      start++;
    }
    maxLen = Math.max(maxLen, end - start + 1);
  }

  return maxLen;
}
```
------------------------------------------------------------------------

## 14. Longest Subarray of Ones After Replacement

**Problem Statement:**\
Replace at most K zeros with ones and find the longest subarray of ones.

**Input:**\
1 1 1 0 0 0 1 1 1 1 0\
2

**Output:**\
6
**solution**
```js
function replaceMostZero(str,k){
  let window =""
  let start =0;
  let countOfZero =0;
   let maxLength =0;
  for(let i=0;i<str.length;i++){
   if(str[i]==="0")countOfZero ++
   while(countOfZero>k){
    if(str[start]==="0")countOfZero --;
    start++;
   }
    return maxLength= Math.max(maxLength,i-start+1)
  }
}
```
------------------------------------------------------------------------

## 15. Permutation in String

**Problem Statement:**\
Check if string S contains a permutation of string P.

**Input:**\
oidbcaf\
abc

**Output:**\
true
**solution**
```js
function permutationTester(s, p) {
  if (!p || p.length === 0) return false;
  if (!s || s.length < p.length) return false;

  const freq = new Map();
  for (let i = 0; i < p.length; i++) {
    const ch = p[i];
    freq.set(ch, (freq.get(ch) || 0) + 1);
  }
  let start =0;
  let matched =0;
  for(let end =0;end<s.length;end++){
    if(freq.has(s[end])){
      freq.set(s[end],freq.get(s[end])-1||0);
      if(freq.get(s[end])===0)matched++ ;
    }
    if(matched===freq.size) return true;
    if(end>=p.length-1){
      let char =s[start];
      start++
      if(freq.has(char)){
        if(freq.get(char)===0)matched --;
        freq.set(char,freq.get(char)+1);
        
      }

    }
  }
  return false ;
}

```
------------------------------------------------------------------------

## 16. Find All Anagrams

**Problem Statement:**\
Find starting indices of all anagrams of P in S.

**Input:**\
cbaebabacd\
abc

**Output:**\
0 6
**solution**
```js
function anagramIndex(s, p) {
  if (!p || p.length === 0) return [];
  if (!s || s.length < p.length) return [];
  const freqMap = new Map();
  for (let i = 0; i < p.length; i++) {
    const ch = p[i];
    freqMap.set(ch, (freqMap.get(ch) || 0) + 1);
  }
  let start = 0;
  let matched = 0;   
  const result = [];

  for (let end = 0; end < s.length; end++) {
    const right = s[end];

    if (freqMap.has(right)) {
      freqMap.set(right, freqMap.get(right) - 1);
      if (freqMap.get(right) === 0) matched++;
    }
    if (matched === freqMap.size) {
      result.push(start);
    }
    if (end >= p.length - 1) {
      const left = s[start];
      start++;

      if (freqMap.has(left)) {
        if (freqMap.get(left) === 0) matched--;
        freqMap.set(left, freqMap.get(left) + 1);
      }
    }
  }

  return result;
}
``` 

------------------------------------------------------------------------

## 17. Maximum Sum of Distinct Subarray

**Problem Statement:**\
Find maximum sum of subarray with all distinct elements.

**Input:**\
5\
4 2 4 5 6

**Output:**\
17
**solution**
```js
function maximumSumDistinctElem(arr) {
  const set = new Set();
  let start = 0;
  let sum = 0;
  let maxSum = 0;
  for (let end = 0; end < arr.length; end++) {
    const x = arr[end];
    while (set.has(x)) {
      set.delete(arr[start]);
      sum -= arr[start];
      start++;
    }
    set.add(x);
    sum += x;
    maxSum = Math.max(maxSum, sum);
  }

  return maxSum;
}
```
------------------------------------------------------------------------

## 18. Longest Subarray with Equal 0s and 1s

**Problem Statement:**\
Find length of longest subarray with equal number of 0s and 1s.

**Input:**\
0 1 0

**Output:**\
2
**solution**
```js
function longestEqualZeroOne(arr) {
  let sum = 0;
  let maxLen = 0;
  const firstIndex = new Map();
  firstIndex.set(0, -1);
  for (let i = 0; i < arr.length; i++) {
    sum += (arr[i] === 1) ? 1 : -1;
    if (firstIndex.has(sum)) {
      maxLen = Math.max(maxLen, i - firstIndex.get(sum));
    } else {
      firstIndex.set(sum, i);
    }
  }
  return maxLen;
}
```
------------------------------------------------------------------------

## 19. Binary Subarrays with Sum

**Problem Statement:**\
Count subarrays with sum equals K in binary array.

**Input:**\
1 0 1 0 1\
2

**Output:**\
4
**solution**
```js
function countSubarraysSumK(arr, k) {
  let sum = 0;
  let ans = 0;
  const freq = new Map();
  freq.set(0, 1);
  for (const x of arr) {
    sum += x;
    const need = sum - k;
    if (freq.has(need)) ans += freq.get(need);
    freq.set(sum, (freq.get(sum) || 0) + 1);
  }
  return ans;
}
```
------------------------------------------------------------------------

## 20. Maximum Consecutive Ones

**Problem Statement:**\
Find maximum consecutive ones in array.

**Input:**\
1 1 0 1 1 1

**Output:**\
3

**solution**
```js
function maximumConOnes(arr) {
  let start = 0;
  let maxLen = 0;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === 0) {
      start = i + 1; 
    }
    maxLen = Math.max(maxLen, i - start + 1);
  }
  return maxLen;
}
```
------------------------------------------------------------------------

## 21. Maximum Sum Circular Subarray (Window Concept)

**Problem Statement:**\
Find maximum sum of subarray in circular array.

**Input:**\
5\
8 -1 3 4

**Output:**\
15

------------------------------------------------------------------------

## 22. Longest Substring with At Least K Repeating Characters

**Problem Statement:**\
Find length of longest substring where each character appears at least K
times.

**Input:**\
aaabb 3

**Output:**\
3

------------------------------------------------------------------------

## 23. Fixed Window Sum Queries

**Problem Statement:**\
Answer multiple queries for sum of subarray of size K.

**Input:**\
Array + Queries

**Output:**\
Sums

------------------------------------------------------------------------

## 24. Sliding Window Median

**Problem Statement:**\
Find median of every sliding window.

**Input:**\
nums, k

**Output:**\
medians

------------------------------------------------------------------------

## 25. Count Nice Subarrays

**Problem Statement:**\
Count subarrays with exactly K odd numbers.

**Input:**\
1 1 2 1 1\
3

**Output:**\
2

------------------------------------------------------------------------

## 26. Max Sum Subarray with Unique Elements

**Problem Statement:**\
Find maximum sum of subarray with unique elements.

------------------------------------------------------------------------

## \## 27. Longest Subarray with At Most Two Distinct Integers

## \## 28. Longest Repeating Character Replacement

## \## 29. Sliding Window Product Less Than K

## \## 30. Subarrays with K Different Integers

## \## 31. Maximum Points You Can Obtain from Cards

## \## 32. Defuse the Bomb

## \## 33. Count Number of Good Subarrays

## \## 34. Longest Substring with At Most K Replacements

## \## 35. Substring with Concatenation of All Words

## \## 36. Sliding Window Maximum (Deque)

## \## 37. Maximum Sum Submatrix (1D Window)

## \## 38. Longest Continuous Increasing Subsequence

## \## 39. Maximum Average Subarray

## \## 40. Replace the Substring for Balanced String

## \## 41. Number of Subarrays with Bounded Maximum

## \## 42. Diet Plan Performance

## \## 43. Minimum Window Substring

## \## 44. Longest Turbulent Subarray

## \## 45. Grumpy Bookstore Owner

## \## 46. Subarrays with Product Less Than K

## \## 47. Max Consecutive Ones III

## \## 48. Frequency of the Most Frequent Element

## \## 49. Longest Nice Subarray

## \## 50. Count Subarrays with Fixed Bounds
