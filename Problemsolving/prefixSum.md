# prefix Sum
1. Range Sum Query

👉 Given array, multiple range sum queries
Seekhoge: basic prefix array

🔹 LeetCode: Range Sum Query – Immutable
```js
function rangeSumQuery(arr){
    let n =arr.length
    let result =new Array(n);
    result =arr[0]
    for(let i=1;i<n;i++){
        result[i]+=result[i-1]+arr[i]
    }
    return result
}
```
2. Running Sum of 1D Array

👉 Har index tak sum banana
Seekhoge: prefix building
```js
function runningSum(arr) {
  let sum = 0;
  const result = [];

  for (let num of arr) {
    sum += num;
    result.push(sum);
  }

  return result;
}

function runningSum(arr) {
  for (let i = 1; i < arr.length; i++) {
    arr[i] += arr[i - 1];
  }
  return arr;
}
```
3. Find Pivot Index

👉 Left sum == Right sum
Seekhoge: prefix + total sum logic
```js
function pivotIndex(arr) {
  const total = arr.reduce((a, b) => a + b, 0);
  let left = 0;
  for (let i = 0; i < arr.length; i++) {
    const right = total - left - arr[i];

    if (left === right) return i;

    left += arr[i];
  }

  return -1;
}


```
4. Subarray Sum Equals K (basic version)

👉 K sum ka subarray exist karta hai?
Seekhoge: prefix difference thinking

5. Count Elements with Equal Left & Right Sum

👉 Partition problem
Seekhoge: prefix balance

🟡 Medium Level (real interview zone)
6. Subarray Sum Equals K

🔥 Very important
Seekhoge: prefix + hashmap trick

7. Continuous Subarray Sum

👉 sum divisible by k
Seekhoge: modulo prefix trick

8. Maximum Size Subarray Sum Equals K

👉 longest subarray with sum k
Seekhoge: hashmap index storage

9. Product of Array Except Self

👉 indirect prefix usage
Seekhoge: prefix + suffix

10. Count Number of Nice Subarrays

👉 odd numbers count
Seekhoge: prefix frequency trick

11. Binary Subarray With Sum

👉 0/1 array sum count
Seekhoge: prefix counting

12. Contiguous Array (Equal 0 and 1)

🔥 Classic FAANG question
Seekhoge: transform + prefix

13. Minimum Size Subarray Sum

👉 prefix + binary search
Seekhoge: optimization

14. Range Sum Query 2D

👉 2D matrix prefix
Seekhoge: 2D prefix sum

15. Corporate Flight Bookings

👉 difference array trick
Seekhoge: reverse prefix idea

🔴 Hard Level (advanced pattern mastery)
16. Subarray Sums Divisible by K

🔥 super important
Seekhoge: modulo + hashmap

17. Maximum Subarray Sum with One Deletion

👉 prefix + suffix combine

18. Make Sum Divisible by P

👉 advanced prefix modulo

19. Number of Submatrices That Sum to Target

👉 2D prefix + hashmap

20. Count Range Sum

👉 prefix + merge sort idea
(very advanced)