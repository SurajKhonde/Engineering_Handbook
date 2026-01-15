# 100 Easy–Medium String Problems (HackerRank-style)

## How to use
- Solve in JS/TS (or any language).
- Aim for **O(n)** or **O(n log n)** when possible.
- For medium ones, expect hashing / two-pointers / sliding window / stack / DP.

---

## 1. Reverse the String (Easy)

**Problem Statement:** Given a string **S**, print the reverse of **S**.
**Input Format:**
- One line containing the string **S** (may contain spaces).

**Output Format:**
- Print the reversed string.


**Sample Input:**
```text
abcd
```
**Sample Output:**
```text
dcba
```
```js
function reverseString(str){
  let arr =str.spilit("");
  let left =0;
  let right =arr.length-1;
  while(left<right){
    [arr[left],arr[right]]=[arr[right],arr[left]];
    left ++ ;
    right --;

  }
  return arr.join("")

}
```
---

## 2. Check Palindrome (Easy)

**Problem Statement:** Given a string **S**, determine whether it reads the same forward and backward (case-sensitive).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** if **S** is a palindrome, otherwise print **"NO"**.


**Sample Input:**
```text
madam
```

**Sample Output:**
```text
YES
```
```js
function checkPalindrome(str){
  let arr =str.spilit("");
  let left =0;
  let right =arr.length-1;
  while(left<right){
    if(arr[left]!==arr[right]) return "NO"
  }
  return "YES"
}
```
---

## 3. Count Vowels (Easy)

**Problem Statement:** Given a string **S**, count how many vowels it contains. Vowels are: a, e, i, o, u (both lowercase and uppercase).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print an integer: the number of vowels in **S**.


**Sample Input:**
```text
Hello World
```

**Sample Output:**
```text
3
```
```js
function countVolves(str){
let volvesFreq=new Set(["a","i","o","e","u"]);
let count =0
for(let i<str.length;i++){
  if(volvesFreq.has(str[i])) count ++

}
return count
} 
```
---

## 4. Count Consonants (Easy)

**Problem Statement:** Given a string **S**, count how many consonant letters it contains (English letters only). Ignore digits, spaces, and punctuation.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the number of consonant letters.


**Sample Input:**
```text
a1b! c
```

**Sample Output:**
```text
2
```
```js
function isAlphachar(char){
    let targetChar =char.toLowercase().charCodeAt()
    if(targetChar>=97 &&targetChar<=122) return true
    return false
  };
 function isVolves (char){
  let isvloves=new Set(["a","e","i","o","u"]);
  return isvloves.has(char.toLowerCase()) ? true:false ;  
 }
function countConstant(str){
  let count=0;
   for(let i=0;i<str.length;i++){
    if(isAlphachar(str[i]) && !isVolves(str[i]) )count ++;

   }
   retrurn count
} 
```
 
---

## 5. Toggle Case (Easy)

**Problem Statement:** Given a string **S**, toggle the case of every English letter: lowercase becomes uppercase and vice versa. Other characters remain unchanged.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the transformed string.


**Sample Input:**
```text
AbC-12xY
```

**Sample Output:**
```text
aBc-12Xy
```
```js
function isAlphaChar(char) {
  const code = char.charCodeAt(0);
  return (code >= 65 && code <= 90) || (code >= 97 && code <= 122); 
};
function caseTransformer(char) {
  const code = char.charCodeAt(0);
  if (code >= 97 && code <= 122) return char.toUpperCase();
  if (code >= 65 && code <= 90) return char.toLowerCase(); 

  return char;
};
function caseTransform(str) {
  let result = "";
  for (let i = 0; i < str.length; i++) {
    const ch = str[i];
    result += isAlphaChar(ch) ? caseTransformer(ch) : ch;
  }
  return result;
};

```
---

## 6. Remove Vowels (Easy)

**Problem Statement:** Given a string **S**, remove all vowels (a, e, i, o, u in both cases) and print the resulting string.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the string after removing vowels.


**Sample Input:**
```text
Beautiful Day
```

**Sample Output:**
```text
Btfl Dy
```
```js
function isVowel(char) {
  const vowels = ["a", "e", "i", "o", "u"];
  return vowels.includes(char.toLowerCase());
}
function removeVowels(str){
  let result ="" ;
  for(let i=0;i<str.length;i++){
    if(!isVloves(str[i])) result+=str[i]
    
  }
  return result

}
```

---

## 7. Replace Spaces with Dashes (Easy)

**Problem Statement:** Given a string **S**, replace every space `' '` with a dash `'-'`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the updated string.


**Sample Input:**
```text
i love dsa
```

**Sample Output:**
```text

i-love-dsa
```

```js
function replaceSpaceswithDashes(str){
  let result =""
  for(let i=0;i<str.length;i++){
    result += (str[i] ===" ")? "-" :str[i];
  }
  return result
}
```

---

## 8. Reverse Words in a Sentence (Easy)

**Problem Statement:** Given a sentence **S** (words separated by single spaces), reverse the order of words.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the sentence with words in reversed order.


**Sample Input:**
```text
learn dsa daily
```

**Sample Output:**
```text
daily dsa learn
```
```js
function reverseWordinSetance(str){
  let spilitChar=str.spilit(" ");
  let start =0;
  let end=spilitChar.length-1;
  while(start<end){
    [spilitChar[start],spilitChar[end]] =[spilitChar[end],spilitChar[start]];
    start++;
    end--;
  };
  return spilitChar.join(" ")
}
```
---

## 9. Capitalize Each Word (Easy)

**Problem Statement:** Given a sentence **S**, convert it to title case: capitalize the first letter of each word, and make the remaining letters lowercase.


**Input Format:**

- One line containing **S** (words separated by single spaces).


**Output Format:**

- Print the title-cased sentence.


**Sample Input:**
```text
heLLo wORld
```

**Sample Output:**
```text
Hello World
```

```js

function capitalizeFirstLetter(str){
  return str.spilit(" ").map((m)=>m ?m[0].toUpperCase()+m.slice(1).toLowerCase():w).join(" ")
}
```

---

## 10. Count Words (Easy)

**Problem Statement:** Given a line **S**, count the number of words. Words are maximal sequences of non-space characters.


**Input Format:**
- One line containing **S**.
**Output Format:**
- Print the number of words.
**Sample Input:**
```text

  one   two three  
```

**Sample Output:**
```text
3
```

```js
function countThewords(str){
  return str.spilit(" ").length
}
```

---

## 11. First Non-Repeating Character (Easy)

**Problem Statement:** Given a string **S**, find the first character that appears exactly once. If none exist, print `-1`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the first non-repeating character, or `-1` if not found.


**Sample Input:**
```text
aabbcddee
```

**Sample Output:**
```text
c
```
```js
const firstNonRepeating = function (str) {
  const freq = new Map();
  for (const ch of str) {
    freq.set(ch, (freq.get(ch) || 0) + 1);
  };
  for (const ch of str) {
    if (freq.get(ch) === 1) return ch;
  }
  return "-1";
}; 
```
---

## 12. First Repeating Character (Easy)

**Problem Statement:** Given a string **S**, find the first character that repeats (i.e., the first character whose second occurrence appears earliest). If no character repeats, print `-1`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the first repeating character, or `-1`.

**Sample Input:**
```text
abca
```

**Sample Output:**
```text
a
```

```js
function firstReaptingChar(str){
  let freq=new Map();
  for(const char of str ){
    freq.set(char,(freq.get(char)||0)+1);
  };
  for(const char of str){
    if(freq.get(char)>1) return char 
  }
  return -1

}

```
---

## 13. Character Frequency Table (Easy)

**Problem Statement:** Given a string **S**, print frequencies of all characters in lexicographic order of characters.


**Input Format:**

- One line containing **S** (no newlines).


**Output Format:**

- For each distinct character `c`, print a line: `c count` in increasing order of `c`.


**Sample Input:**
```text
baba
```

**Sample Output:**
```text
a 2
b 2
```
```js
function charCount(str) {
  const freq = new Map();
  for (let i = 0; i < str.length; i++) {
    const ch = str[i];
    freq.set(ch, (freq.get(ch) || 0) + 1);
  };
  // if order is not required then time coplexity is O(n)
  const keys = Array.from(freq.keys()).sort();
  for (const ch of keys) {
    console.log(`${ch} ${freq.get(ch)}`);
  }
}
// 2
function charCount(str) {
  const freq = {};
  for (let i = 0; i < str.length; i++) {
    const ch = str[i];
    freq[ch] = (freq[ch] || 0) + 1;
  }
  // if order is not required then time coplexity is O(n) just remove sort
  const keys = Object.keys(freq).sort();

  for (const ch of keys) {
    console.log(`${ch} ${freq[ch]}`);
  }
}
```
---

## 14. Most Frequent Character (Easy)

**Problem Statement:** Given a string **S**, find the character with maximum frequency. If there is a tie, choose the lexicographically smallest character.

**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the chosen character.


**Sample Input:**
```text
bbccaaab
```

**Sample Output:**
```text
b
```
```js
function mostFreqChar(str) {
  let bestCount = 0;
  let bestChar = "";
  const freq = new Map();

  for (let i = 0; i < str.length; i++) {
    const ch = str[i];
    freq.set(ch, (freq.get(ch) || 0) + 1);
  }

  for (const [ch, count] of freq) {
    if (count > bestCount || (count === bestCount && (bestChar === "" || ch < bestChar))) {
      bestCount = count;
      bestChar = ch;
    }
  }

  return bestChar;
}
```

---

## 15. Least Frequent Character (Easy)

**Problem Statement:** Given a string **S**, find the character with minimum frequency. If there is a tie, choose the lexicographically smallest character.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the chosen character.


**Sample Input:**
```text
aabccc
```

**Sample Output:**
```text
b
```
```js
function leastFrequntChar(str){
  let leastChar ="";
  let leastcount =Infinity; 
  let freq =new Map();
  for(let i=0;i<str.length;i++){
      freq.set(str[i],(freq.get(str[i]||0)+1))
    }
    for(const [ch,count]of freq){
      if(count < leastcount || (count === leastcount && (leastChar === "" || ch <leastChar))){
        leastcount = count;
        leastChar = ch;
      }
          }
           return leastChar;
  }

```
---

## 16. Check Anagram (Easy)

**Problem Statement:** Given two strings **A** and **B**, determine if they are anagrams (same multiset of characters). Case-sensitive.


**Input Format:**

- Two lines: string **A**, then string **B**.


**Output Format:**

- Print **"YES"** if they are anagrams, else **"NO"**.


**Sample Input:**
```text
listen
silent
```

**Sample Output:**
```text
YES
```
```js
function checkAnagram(str1,str2){
  if (str1.length !== str2.length) return "NO";
  let freq =new Map();
  for(let i=0;i<str1.length;i++){
    let ch =str1[i]
    freq.set(ch,freq.get(ch)||0+1);
  }
    for(let i=0;i<str2.length;i++){
    let ch =str2[i]
    if (!freq.has(ch)) return "NO";
    freq.set(ch, freq.get(ch) - 1);
    if (freq.get(ch) === 0) freq.delete(ch);
  }
  return freq.size === 0 ? "YES" : "NO";
}
```
---

## 17. Anagram by One Swap (Easy)

**Problem Statement:** Given two strings **A** and **B** of the same length, determine if you can make **A** equal to **B** by swapping exactly one pair of characters in **A**. If **A** is already equal to **B**, answer should be **NO** (because you must swap).


**Input Format:**

- Two lines: **A** and **B**.


**Output Format:**

- Print **"YES"** or **"NO"**.


**Sample Input:**
```text
abcd
abdc
```

**Sample Output:**
```text
YES
```
```js
function canMakeEqualWithOneSwap(A, B) {
  if (A.length !== B.length) return "NO";
  if (A === B) return "NO";
  const diff = [];
  for (let i = 0; i < A.length; i++) {
    if (A[i] !== B[i]) diff.push(i);
    if (diff.length > 2) return "NO";
  }
  if (diff.length !== 2) return "NO";

  const [i, j] = diff;
  return (A[i] === B[j] && A[j] === B[i]) ? "YES" : "NO";
}
```
---

## 18. Check Isomorphic Strings (Medium)

**Problem Statement:** Two strings are **isomorphic** if characters in the first string can be replaced to get the second string, with a one-to-one mapping (no two characters map to the same character). Given **A** and **B**, determine if they are isomorphic.


**Input Format:**

- Two lines: **A**, **B** (same length).


**Output Format:**

- Print **"YES"** if isomorphic, else **"NO"**.


**Sample Input:**
```text
egg
add
```

**Sample Output:**
```text
YES
```
```js
function isIsomorphic(A, B) {
  if (A.length !== B.length) return "NO";
  const aToB = new Map();
  const bToA = new Map();
  for (let i = 0; i < A.length; i++) {
    const a = A[i];
    const b = B[i];
    if (aToB.has(a) && aToB.get(a) !== b) return "NO";
    if (bToA.has(b) && bToA.get(b) !== a) return "NO";
    aToB.set(a, b);
    bToA.set(b, a);
  }
  return "YES";
}
```
---

## 19. Check Rotation (Easy)

**Problem Statement:** Given strings **A** and **B**, determine if **B** is a rotation of **A** (obtained by moving some prefix of **A** to the end).


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print **"YES"** if **B** is a rotation of **A**, else **"NO"**.


**Sample Input:**
```text
abcd
cdab
```
```js
function isRotation(A, B) {
  if (A.length !== B.length) return "NO";
  return (A + A).includes(B) ? "YES" : "NO";
}
console.log(isRotation("abcd", "cdab"));

```
**Sample Output:**
```text
YES
```

---

## 20. Check Subsequence (Easy)

**Problem Statement:** Given strings **S** and **T**, determine whether **S** is a subsequence of **T** (you can delete characters from **T** without reordering).


**Input Format:**

- Two lines: **S** then **T**.


**Output Format:**

- Print **"YES"** if **S** is a subsequence of **T**, else **"NO"**.


**Sample Input:**
```text
ace
abcde
```

**Sample Output:**
```text
YES
```
```js
function isSubsequence(S, T) {
  let i = 0;
  for (let j = 0; j < T.length && i < S.length; j++) {
    if (T[j] === S[i]) i++;
  }
  return i === S.length ? "YES" : "NO";
}
```
---

## 21. Longest Common Prefix (Easy)

**Problem Statement:** Given **N** strings, find their longest common prefix. If no common prefix exists, print `-1`.


**Input Format:**

- First line integer **N**. Next **N** lines contain strings.


**Output Format:**

- Print the longest common prefix or `-1`.


**Sample Input:**
```text
3
flower
flow
flight
```

**Sample Output:**
```text
fl
```
```js
//startsWith means does the big string begin with the smaller string:
"flower".startsWith("flow") // true   (because "flower" begins with "flow")
"flow".startsWith("flower") // false  (because "flow" is shorter)
function longestCommonPrefix(arr) {
  if (!arr.length) return "-1";
  let prefix = arr[0];
  for (let i = 1; i < arr.length; i++) {
    while (!arr[i].startsWith(prefix)) {
      prefix = prefix.slice(0, -1);
      if (prefix === "") return "-1";
    }
  }

  return prefix;
}

console.log(longestCommonPrefix(["flower", "flow", "flight"])); // "fl"

```
---

## 22. Remove Adjacent Duplicates (Easy)

**Problem Statement:** Given a string **S**, repeatedly remove pairs of adjacent equal characters until no such pair exists. Print the final string (or `EMPTY` if it becomes empty).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the reduced string or `EMPTY`.


**Sample Input:**
```text
abbaca
```

**Sample Output:**
```text
ca
```
```js
function removeAdjacentChar(str) {
  const stack = [];
  for (let i = 0; i < str.length; i++) {
    const ch = str[i];
    if (stack.length && stack[stack.length - 1] === ch) stack.pop();
    else stack.push(ch);
  }
  const res = stack.join("");
  return res === "" ? "EMPTY" : res;
}
console.log(removeAdjacentChar("abbaca")); // "ca"
```

---

## 23. Remove Duplicate Characters (Keep First) (Easy)

**Problem Statement:** Given a string **S**, remove duplicate occurrences of characters while keeping the **first** occurrence of each character and preserving order.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the resulting string.


**Sample Input:**
```text
banana
```

**Sample Output:**
```text
ban
```
```js
function removeDuplicate(str){
  let seen = new Set();
  result =""
  for(let i=0;i<str.length;i++){
    if(!seen.has(str[i])){
       seen.add(str[i]);
       result +=str[i];
    }
    
  }
  return result
}
```

---

## 24. Reverse Only Letters (Easy)

**Problem Statement:** Given a string **S** containing letters and non-letters, reverse only the letters while keeping non-letters in the same positions.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the transformed string.


**Sample Input:**
```text
a-bC-dEf-ghIj
```

**Sample Output:**
```text
j-Ih-gfE-dCba
```
```js
function isAlpha(char) {
  const code = char.charCodeAt(0);
  return (code >= 65 && code <= 90) || (code >= 97 && code <= 122);
}

function reverseOnlyLetter(str) {
  const arr = str.split("");
  let left = 0;
  let right = arr.length - 1;

  while (left < right) {
    if (isAlpha(arr[left]) && isAlpha(arr[right])) {
      [arr[left], arr[right]] = [arr[right], arr[left]];
      left++;
      right--;
    } else if (!isAlpha(arr[left])) {
      left++;
    } else {
      right--;
    }
  }

  return arr.join("");
}
```
---

## 25. Reverse Only Vowels (Easy)

**Problem Statement:** Given a string **S**, reverse the positions of vowels only (a, e, i, o, u in both cases). Other characters stay where they are.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the transformed string.


**Sample Input:**
```text
hello
```

**Sample Output:**
```text
holle
```
```js
function isVolves(char){
  let Voloves = new Set(["a","e","i","o","u"]);
 return Voloves.has(char.toLowerCase()) 
}
function reverseVolves(str){
  let arr =str.spilit("");
  let left =0;
  let right =arr.length -1;
  while(left<right){
    if(isVolves(arr[left])&&isVolves(arr[right])){
      [arr[left],arr[right]]=[arr[right],arr[left]];
      left++;
      right--:
    }else if(!isVolves(str[left])){
      left ++
    }else{
      right --;
    }

    
  }
  return arr.join("")
}
```
---

## 26. Run-Length Encode (Easy)

**Problem Statement:** Given a string **S** consisting of lowercase letters, compress it using run-length encoding. Replace each group of same consecutive characters by `char + count`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the encoded string.


**Sample Input:**
```text
aaabbc
```

**Sample Output:**
```text
a3b2c1
```
```js
function runLengthEncode(str) {
  if (str.length === 0) return "";
  let result = "";
  let count = 1;
  for (let i = 1; i <= str.length; i++) {
    if (i < str.length && str[i] === str[i - 1]) {
      count++;
    } else {
      result += str[i - 1] + count;
      count = 1;
    }
  }

  return result;
}
```
---

## 27. Run-Length Decode (Easy)

**Problem Statement:** Given a run-length encoded string like `a3b2c1` (lowercase letters followed by positive integers), decode it.


**Input Format:**

- One line containing the encoded string **E**.


**Output Format:**

- Print the decoded string.


**Sample Input:**
```text
a3b2c1
```

**Sample Output:**
```text
aaabbc
```
```js
function runLengthdecode(str){
  let result =""
  for(let i=0;i<str.length;i+=2){
    let char = str[i];
    let count =  Number(str[i]);
    for(k=0;k<count k++){
      result +=char
    }
  }
return result
}
```
---

## 28. Validate Parentheses (Easy)

**Problem Statement:** Given a string **S** containing only the characters `()[]{}`, determine if it is valid. A string is valid if brackets close in the correct order and type.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** if valid, else **"NO"**.


**Sample Input:**
```text
{[()]}
```

**Sample Output:**
```text
YES
```
```js
function validateParenthesis(str) {
  const stack = [];
  const closeToOpen = {
    ")": "(",
    "]": "[",
    "}": "{",
  };

  for (let i = 0; i < str.length; i++) {
    const ch = str[i];

    if (ch === "(" || ch === "[" || ch === "{") {
      stack.push(ch);
    } else {
      if (stack.length === 0) return "NO";
      const top = stack.pop();
      if (top !== closeToOpen[ch]) return "NO";
    }
  }

  return stack.length === 0 ? "YES" : "NO";
}
//2 nd only follow YES for one senario {[()]} another sinario it will fail like  ()[]{}
// Proper rule: [ opened, must close with ] before )
function validateParenthesis(str) {
  let left =0;
  let right= str.length-1;
    const closeToOpen = {
    "(":")",
    "[":"]",
    "{":"}"
  };
while(left <right){
  if(closeToOpen[str[left]]!==str[right])return "NO";
  left++
  right--
}
return "YES"
}


```
---

## 29. Minimum Adds to Make Parentheses Valid (Easy)

**Problem Statement:** Given a string **S** consisting of only `'('` and `')'`, find the minimum number of parentheses you must add (anywhere) to make it valid.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the minimum number of additions needed.


**Sample Input:**
```text
()))(
```

**Sample Output:**
```text
3
```
```js
function minAddsToMakeValid(str) {
  let balance = 0; // unmatched '('
  let adds = 0;    // needed '(' additions
  for (const ch of str) {
    if (ch === "(") {
      balance++;
    } else { // ')'
      if (balance > 0) balance--;
      else adds++; // add one '('
    }
  }
  return adds + balance; // add ')' for leftover '('
}

```

**Correct logic (greedy)**
Track:
balance = how many unmatched '(' you have
adds = how many '(' you need to add when you see an extra ')'
Rule:
If char is '(' → balance++
If char is ')':
if balance > 0 → match it → balance--
else → no '(' to match → must add one '(' → adds++
At the end, any remaining balance means unmatched '(' → need that many ')' additions.
---

## 30. Longest Valid Parentheses Length (Medium)

**Problem Statement:** Given a string **S** of `'('` and `')'`, find the length of the longest contiguous substring that forms a valid parentheses sequence.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print an integer length.


**Sample Input:**
```text
)()())
```

**Sample Output:**
```text
4
```

---

## 31. Count Occurrences of a Substring (Easy)

**Problem Statement:** Given a string **S** and a pattern **P**, count how many times **P** occurs in **S** (overlapping allowed).


**Input Format:**

- Two lines: **S** then **P**.


**Output Format:**

- Print an integer: number of occurrences.


**Sample Input:**
```text
aaaa
aa
```

**Sample Output:**
```text
3
```

---

## 32. Find First Index of Substring (Easy)

**Problem Statement:** Given strings **S** and **P**, find the index (0-based) of the first occurrence of **P** in **S**. If not found, print `-1`.


**Input Format:**

- Two lines: **S**, **P**.


**Output Format:**

- Print an integer index or `-1`.


**Sample Input:**
```text
hello
ll
```

**Sample Output:**
```text
2
```

---

## 33. Replace All Occurrences (Easy)

**Problem Statement:** Given a string **S**, a pattern **P**, and a replacement **R**, replace all non-overlapping occurrences of **P** with **R**.


**Input Format:**

- Three lines: **S**, **P**, **R**.


**Output Format:**

- Print the updated string.


**Sample Input:**
```text
foo bar foo
foo
baz
```

**Sample Output:**
```text
baz bar baz
```

---

## 34. Remove All Occurrences of a Substring (Easy)

**Problem Statement:** Given a string **S** and a substring **P**, remove **all** occurrences of **P** (including those created after removals) and print the final string.


**Input Format:**

- Two lines: **S** then **P**.


**Output Format:**

- Print the resulting string (or `EMPTY` if empty).


**Sample Input:**
```text
daabcbaabcbc
abc
```

**Sample Output:**
```text
dab
```

---

## 35. Check Pangram (Easy)

**Problem Statement:** A sentence is a pangram if it contains every English letter at least once. Given **S**, determine if it is a pangram (case-insensitive).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** if pangram, else **"NO"**.


**Sample Input:**
```text
The quick brown fox jumps over the lazy dog
```

**Sample Output:**
```text
YES
```

---

## 36. Valid Palindrome (Ignore Non-Alphanumeric) (Easy)

**Problem Statement:** Given a string **S**, determine if it is a palindrome considering only alphanumeric characters and ignoring case.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** or **"NO"**.


**Sample Input:**
```text
A man, a plan, a canal: Panama
```

**Sample Output:**
```text
YES
```

---

## 37. Normalize Spaces (Easy)

**Problem Statement:** Given a string **S**, remove leading/trailing spaces and replace multiple spaces between words with a single space.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the normalized string.


**Sample Input:**
```text
  i   love   dsa  
```

**Sample Output:**
```text
i love dsa
```

---

## 38. Longest Substring Without Repeating Characters (Medium)

**Problem Statement:** Given a string **S**, find the length of the longest substring with all distinct characters.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print an integer length.


**Sample Input:**
```text
abcabcbb
```

**Sample Output:**
```text
3
```

---

## 39. Longest Substring With At Most K Distinct Characters (Medium)

**Problem Statement:** Given a string **S** and integer **K**, find the length of the longest substring that contains at most **K** distinct characters.


**Input Format:**

- Line 1: string **S**. Line 2: integer **K**.


**Output Format:**

- Print the maximum length.


**Sample Input:**
```text
araaci
2
```

**Sample Output:**
```text
4
```

---

## 40. Minimum Window Containing All Characters (Medium)

**Problem Statement:** Given strings **S** and **T**, find the length of the smallest substring of **S** that contains all characters of **T** (including duplicates). If impossible, print `-1`.


**Input Format:**

- Two lines: **S**, **T**.


**Output Format:**

- Print the minimum window length, or `-1`.


**Sample Input:**
```text
ADOBECODEBANC
ABC
```

**Sample Output:**
```text
4
```

---

## 41. All Anagram Start Indices (Medium)

**Problem Statement:** Given a string **S** and pattern **P**, output all 0-based indices in **S** where an anagram of **P** begins, in increasing order.


**Input Format:**

- Two lines: **S**, **P** (lowercase).


**Output Format:**

- Print space-separated indices (or an empty line if none).


**Sample Input:**
```text
cbaebabacd
abc
```

**Sample Output:**
```text
0 6
```

---

## 42. Count Palindromic Substrings (Medium)

**Problem Statement:** Given a string **S**, count the number of palindromic substrings (every single character counts as a palindrome).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the count.


**Sample Input:**
```text
aaa
```

**Sample Output:**
```text
6
```

---

## 43. Longest Palindromic Substring Length (Medium)

**Problem Statement:** Given a string **S**, find the length of the longest palindromic substring.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the maximum length.


**Sample Input:**
```text
babad
```

**Sample Output:**
```text
3
```

---

## 44. Minimum Deletions to Make Anagram (Easy)

**Problem Statement:** Given two strings **A** and **B**, find the minimum number of character deletions needed (from either string) so that the two strings become anagrams.


**Input Format:**

- Two lines: **A**, **B** (lowercase).


**Output Format:**

- Print the minimum deletions.


**Sample Input:**
```text
cde
abc
```

**Sample Output:**
```text
4
```

---

## 45. Sort Characters by Frequency (Medium)

**Problem Statement:** Given a string **S**, sort its characters in decreasing order of frequency. If multiple characters have the same frequency, sort them lexicographically.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the sorted string.


**Sample Input:**
```text
tree
```

**Sample Output:**
```text
eert
```

---

## 46. Caesar Cipher Encode (Easy)

**Problem Statement:** Given a lowercase string **S** and integer **K**, shift every letter forward by **K** in the alphabet (wrapping around).


**Input Format:**

- Line 1: **S** (lowercase). Line 2: integer **K**.


**Output Format:**

- Print the encoded string.


**Sample Input:**
```text
xyz
3
```

**Sample Output:**
```text
abc
```

---

## 47. Caesar Cipher Decode (Easy)

**Problem Statement:** Given a lowercase string **S** encrypted with Caesar cipher shift **K**, decode it by shifting letters backward by **K**.


**Input Format:**

- Line 1: **S** (lowercase). Line 2: integer **K**.


**Output Format:**

- Print the decoded string.


**Sample Input:**
```text
abc
3
```

**Sample Output:**
```text
xyz
```

---

## 48. Add Two Binary Strings (Medium)

**Problem Statement:** Given two binary strings **A** and **B**, compute their sum and output it as a binary string.


**Input Format:**

- Two lines: **A**, **B** (each contains only `0` and `1`).


**Output Format:**

- Print the binary sum.


**Sample Input:**
```text
1011
110
```

**Sample Output:**
```text
10001
```

---

## 49. Compare Version Numbers (Medium)

**Problem Statement:** Given two version strings like `1.0.3` and `1.0`, compare them. Compare each dot-separated component as an integer. Output `-1` if A < B, `0` if equal, `1` if A > B.


**Input Format:**

- Two lines: version **A**, version **B**.


**Output Format:**

- Print `-1`, `0`, or `1`.


**Sample Input:**
```text
1.0.3
1.0
```

**Sample Output:**
```text
1
```

---

## 50. Validate IPv4 Address (Medium)

**Problem Statement:** Given a string **S**, determine if it is a valid IPv4 address. Valid IPv4 has 4 parts separated by dots, each part is 0..255 with no leading zeros unless the part is exactly '0'.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** if valid, else **"NO"**.


**Sample Input:**
```text
192.168.0.1
```

**Sample Output:**
```text
YES
```

---

## 51. Simplify Unix Path (Medium)

**Problem Statement:** Given a Unix-style absolute path, simplify it by resolving `.` and `..` and removing extra slashes.


**Input Format:**

- One line containing the path **P**.


**Output Format:**

- Print the simplified absolute path.


**Sample Input:**
```text
/a//b/./c/../d/
```

**Sample Output:**
```text
/a/b/d
```

---

## 52. Decode Nested Repetition (Medium)

**Problem Statement:** Given an encoded string like `3[a2[c]]`, decode it. The rule is `k[encoded]` means repeat `encoded` **k** times. Assume input is valid.


**Input Format:**

- One line containing the encoded string.


**Output Format:**

- Print the decoded string.


**Sample Input:**
```text
3[a2[c]]
```

**Sample Output:**
```text
accaccacc
```

---

## 53. Edit Distance (Medium)

**Problem Statement:** Given strings **A** and **B**, compute the minimum number of operations to convert **A** to **B**. Allowed operations: insert, delete, replace one character.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print the edit distance.


**Sample Input:**
```text
kitten
sitting
```

**Sample Output:**
```text
3
```

---

## 54. Longest Common Subsequence Length (Medium)

**Problem Statement:** Given strings **A** and **B**, compute the length of their longest common subsequence.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print the LCS length.


**Sample Input:**
```text
abcde
ace
```

**Sample Output:**
```text
3
```

---

## 55. Remove K Digits to Make Smallest Number (Medium)

**Problem Statement:** Given a numeric string **S** and integer **K**, remove exactly **K** digits to produce the smallest possible number (no leading zeros unless the result is zero).


**Input Format:**

- Line 1: **S**. Line 2: integer **K**.


**Output Format:**

- Print the smallest possible number as a string.


**Sample Input:**
```text
1432219
3
```

**Sample Output:**
```text
1219
```

---

## 56. Roman to Integer (Medium)

**Problem Statement:** Given a valid Roman numeral string **S**, convert it to an integer. Symbols: I(1), V(5), X(10), L(50), C(100), D(500), M(1000).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the integer value.


**Sample Input:**
```text
MCMXCIV
```

**Sample Output:**
```text
1994
```

---

## 57. Integer to Roman (Medium)

**Problem Statement:** Given an integer **N** (1 <= N <= 3999), convert it to Roman numerals using standard rules.


**Input Format:**

- One line containing integer **N**.


**Output Format:**

- Print Roman numeral string.


**Sample Input:**
```text
58
```

**Sample Output:**
```text
LVIII
```

---

## 58. Check If Any Permutation Exists as Substring (Medium)

**Problem Statement:** Given strings **S** and **P**, determine if any permutation (anagram) of **P** appears as a contiguous substring of **S**.


**Input Format:**

- Two lines: **S**, **P**.


**Output Format:**

- Print **"YES"** if exists, else **"NO"**.


**Sample Input:**
```text
oidbcaf
abc
```

**Sample Output:**
```text
YES
```

---

## 59. Longest Repeating Character Replacement (Medium)

**Problem Statement:** Given a string **S** of uppercase letters and integer **K**, you may replace at most **K** characters in a substring to make all characters equal. Find the maximum possible substring length.


**Input Format:**

- Line 1: **S**. Line 2: integer **K**.


**Output Format:**

- Print the maximum length.


**Sample Input:**
```text
AABABBA
1
```

**Sample Output:**
```text
4
```

---

## 60. Group Anagrams (Medium)

**Problem Statement:** Given **N** strings, group them by anagram. Output the number of groups, then for each group print its strings in input order (one group per line).


**Input Format:**

- First line integer **N**. Next **N** lines: strings.


**Output Format:**

- Line 1: groups count. Next lines: each group as space-separated strings.


**Sample Input:**
```text
6
eat
tea
tan
ate
nat
bat
```

**Sample Output:**
```text
3
eat tea ate
tan nat
bat
```

---

## 61. Convert snake_case to camelCase (Easy)

**Problem Statement:** Given a `snake_case` identifier **S**, convert it to `camelCase`. Assume **S** contains lowercase letters and underscores only.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the camelCase string.


**Sample Input:**
```text
my_variable_name
```

**Sample Output:**
```text
myVariableName
```

---

## 62. Convert camelCase to snake_case (Easy)

**Problem Statement:** Given a `camelCase` identifier **S**, convert it to `snake_case`. Assume **S** contains only English letters.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the snake_case string (lowercase).


**Sample Input:**
```text
myVariableName
```

**Sample Output:**
```text
my_variable_name
```

---

## 63. Split CamelCase Words (Easy)

**Problem Statement:** Given a camelCase or PascalCase string **S**, split it into words and print them separated by spaces.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print words separated by single spaces.


**Sample Input:**
```text
HTTPServerError
```

**Sample Output:**
```text
HTTP Server Error
```

---

## 64. Check One Edit Away (Medium)

**Problem Statement:** Given strings **A** and **B**, determine if they are at edit distance at most 1, where an edit is insert, delete, or replace a single character.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print **"YES"** if <= 1 edit away, else **"NO"**.


**Sample Input:**
```text
pale
ple
```

**Sample Output:**
```text
YES
```

---

## 65. Big Integer Addition (Medium)

**Problem Statement:** Given two non-negative integers **A** and **B** as strings (they may be too large for 64-bit), output their sum as a string.


**Input Format:**

- Two lines: **A**, **B** (digits only).


**Output Format:**

- Print the sum as a string.


**Sample Input:**
```text
999999999999999999
2
```

**Sample Output:**
```text
1000000000000000001
```

---

## 66. Big Integer Multiplication (Medium)

**Problem Statement:** Given two non-negative integers **A** and **B** as strings, output `A * B` as a string (without using big integer libraries).


**Input Format:**

- Two lines: **A**, **B** (digits only).


**Output Format:**

- Print the product.


**Sample Input:**
```text
123
45
```

**Sample Output:**
```text
5535
```

---

## 67. Validate Simple Email (Medium)

**Problem Statement:** Validate an email with these simplified rules: exactly one `@`, at least one character before `@`, domain contains at least one `.`, and no spaces. Print YES/NO.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** or **"NO"**.


**Sample Input:**
```text
john.doe@mail.com
```

**Sample Output:**
```text
YES
```

---

## 68. URL Decode (Percent Encoding) (Easy)

**Problem Statement:** Given a string **S** containing percent-encoded bytes like `%20`, decode it. Assume only `%` followed by two hex digits appears as encoding; other characters stay the same.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the decoded string.


**Sample Input:**
```text
hello%20world%21
```

**Sample Output:**
```text
hello world!
```

---

## 69. URL Encode Spaces (Easy)

**Problem Statement:** Given a string **S**, replace every space with `%20`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the encoded string.


**Sample Input:**
```text
hi there
```

**Sample Output:**
```text
hi%20there
```

---

## 70. Buddy Strings (Medium)

**Problem Statement:** Given strings **A** and **B** of equal length, determine if you can swap exactly two characters in **A** so that it becomes **B**.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print **"YES"** or **"NO"**.


**Sample Input:**
```text
ab
ba
```

**Sample Output:**
```text
YES
```

---

## 71. Longest Prefix Also Suffix (Medium)

**Problem Statement:** Given a string **S**, find the longest proper prefix of **S** which is also a suffix of **S**. Proper prefix means not equal to the whole string. Print the prefix, or `-1` if none.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the prefix or `-1`.


**Sample Input:**
```text
ababcab
```

**Sample Output:**
```text
ab
```

---

## 72. Count Substrings With Equal 0s and 1s (Medium)

**Problem Statement:** Given a binary string **S**, count the number of substrings that contain the same number of `0`s and `1`s.


**Input Format:**

- One line: binary string **S**.


**Output Format:**

- Print the count.


**Sample Input:**
```text
010
```

**Sample Output:**
```text
2
```

---

## 73. Maximum Consecutive 1s After Flipping K Zeros (Medium)

**Problem Statement:** Given a binary string **S** and integer **K**, you may flip at most **K** zeros to ones. Find the maximum length of a contiguous substring of all 1s after flips.


**Input Format:**

- Line 1: **S**. Line 2: integer **K**.


**Output Format:**

- Print the maximum length.


**Sample Input:**
```text
110100110
1
```

**Sample Output:**
```text
4
```

---

## 74. Minimum Flips to Make Alternating (Easy)

**Problem Statement:** Given a binary string **S**, compute the minimum number of flips needed to make it alternating (either `0101...` or `1010...`).


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the minimum flips.


**Sample Input:**
```text
010011
```

**Sample Output:**
```text
2
```

---

## 75. Longest Common Substring Length (Medium)

**Problem Statement:** Given strings **A** and **B**, compute the length of their longest common contiguous substring.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print the length.


**Sample Input:**
```text
ababc
babca
```

**Sample Output:**
```text
4
```

---

## 76. Minimum Repeats of A to Contain B (Medium)

**Problem Statement:** Given strings **A** and **B**, find the minimum number of times **A** must be repeated so that **B** becomes a substring of the repeated string. If impossible, print `-1`.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print the minimum repeats or `-1`.


**Sample Input:**
```text
abcd
cdabcdab
```

**Sample Output:**
```text
3
```

---

## 77. Reverse Characters in Each Word (Easy)

**Problem Statement:** Given a sentence **S**, reverse the characters of each word but keep word order the same.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the transformed sentence.


**Sample Input:**
```text
hello world
```

**Sample Output:**
```text
olleh dlrow
```

---

## 78. Rotate Words by K (Easy)

**Problem Statement:** Given a sentence **S** (single spaces) and integer **K**, rotate the words to the right by **K** positions.


**Input Format:**

- Line 1: **S**. Line 2: integer **K**.


**Output Format:**

- Print the rotated sentence.


**Sample Input:**
```text
a b c d e
2
```

**Sample Output:**
```text
d e a b c
```

---

## 79. Find the Missing Letter (Easy)

**Problem Statement:** Given a string **S** of distinct lowercase letters in sorted order, but missing exactly one letter in the range from first to last, find the missing letter.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the missing letter.


**Sample Input:**
```text
abdef
```

**Sample Output:**
```text
c
```

---

## 80. Find First Unique Word (Easy)

**Problem Statement:** Given a sentence **S** (single spaces), find the first word that appears exactly once. If none, print `-1`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the first unique word or `-1`.


**Sample Input:**
```text
red blue red green blue
```

**Sample Output:**
```text
green
```

---

## 81. Minimum Insertions to Make Palindrome (Medium)

**Problem Statement:** Given a string **S**, find the minimum number of characters you need to insert (anywhere) to make **S** a palindrome.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the minimum insertions.


**Sample Input:**
```text
leetcode
```

**Sample Output:**
```text
5
```

---

## 82. Find All Duplicate Characters (Easy)

**Problem Statement:** Given a string **S**, print all characters that appear more than once, in lexicographic order. If none, print `-1`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the duplicate characters as a string, or `-1`.


**Sample Input:**
```text
programming
```

**Sample Output:**
```text
gmr
```

---

## 83. Remove Characters Present in Another String (Easy)

**Problem Statement:** Given strings **S** and **R**, remove from **S** every character that appears in **R**. Preserve order of remaining characters.


**Input Format:**

- Two lines: **S**, **R**.


**Output Format:**

- Print the filtered string.


**Sample Input:**
```text
battle
bt
```

**Sample Output:**
```text
ale
```

---

## 84. Find the Extra Character (Easy)

**Problem Statement:** Given strings **A** and **B**, where **B** is **A** with one extra character added and then shuffled, find the extra character.


**Input Format:**

- Two lines: **A**, **B**.


**Output Format:**

- Print the extra character.


**Sample Input:**
```text
abcd
abcde
```

**Sample Output:**
```text
e
```

---

## 85. Count Reversed Pairs (Easy)

**Problem Statement:** Given **N** strings, count how many pairs `(i, j)` with `i < j` satisfy that `strings[j]` is exactly the reverse of `strings[i]`.


**Input Format:**

- First line integer **N**, then **N** strings (no spaces).


**Output Format:**

- Print the count of reversed pairs.


**Sample Input:**
```text
4
ab
ba
abc
cba
```

**Sample Output:**
```text
2
```

---

## 86. Sort Strings by Length Then Lexicographically (Easy)

**Problem Statement:** Given **N** strings, sort them by increasing length; if lengths tie, sort lexicographically. Print the sorted list.


**Input Format:**

- First line **N**. Next **N** lines: strings.


**Output Format:**

- Print **N** lines: sorted strings.


**Sample Input:**
```text
4
bb
a
ccc
b
```

**Sample Output:**
```text
a
b
bb
ccc
```

---

## 87. Minimum Deletions to Make All Characters Same (Easy)

**Problem Statement:** Given a string **S**, delete the minimum number of characters so that all remaining characters are the same.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the minimum deletions.


**Sample Input:**
```text
aabbbc
```

**Sample Output:**
```text
3
```

---

## 88. Longest Run of Same Character (Easy)

**Problem Statement:** Given a string **S**, find the length of the longest consecutive run of the same character.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the maximum run length.


**Sample Input:**
```text
aaabbaacccccc
```

**Sample Output:**
```text
6
```

---

## 89. Count Runs (Consecutive Groups) (Easy)

**Problem Statement:** Given a string **S**, count the number of consecutive groups (runs). For example, `aaabb` has runs: `aaa`, `bb` => 2.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the number of runs.


**Sample Input:**
```text
aaabbaac
```

**Sample Output:**
```text
4
```

---

## 90. Count Substrings Starting and Ending With Same Character (Medium)

**Problem Statement:** Given a string **S**, count the number of substrings that start and end with the same character.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the count.


**Sample Input:**
```text
abcab
```

**Sample Output:**
```text
7
```

---

## 91. Check If Palindrome After At Most One Deletion (Medium)

**Problem Statement:** Given a string **S**, determine if it can become a palindrome by deleting at most one character.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** or **"NO"**.


**Sample Input:**
```text
abca
```

**Sample Output:**
```text
YES
```

---

## 92. Reverse Substring Between Two Indices (Easy)

**Problem Statement:** Given a string **S** and two indices **L** and **R** (0-based, L <= R), reverse the substring `S[L..R]` and print the new string.


**Input Format:**

- Line 1: **S**. Line 2: **L R**.


**Output Format:**

- Print the resulting string.


**Sample Input:**
```text
abcdef
1 4
```

**Sample Output:**
```text
aedcbf
```

---

## 93. Count Substrings With Sum K (Binary String) (Medium)

**Problem Statement:** Given a binary string **S** and integer **K**, count the number of substrings whose number of `1`s equals **K**.


**Input Format:**

- Line 1: **S**. Line 2: integer **K**.


**Output Format:**

- Print the count.


**Sample Input:**
```text
10101
2
```

**Sample Output:**
```text
4
```

---

## 94. Minimum Window With All Vowels (Medium)

**Problem Statement:** Given a lowercase string **S**, find the length of the smallest substring that contains all five vowels `a,e,i,o,u` at least once. If none, print `-1`.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the minimum length or `-1`.


**Sample Input:**
```text
aabicodue
```

**Sample Output:**
```text
8
```

---

## 95. Count Good Splits (Medium)

**Problem Statement:** A split of **S** at position `i` divides it into `S[0..i-1]` and `S[i..]`. A split is **good** if both sides have the same number of distinct characters. Count good splits.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print the count of good splits.


**Sample Input:**
```text
aacaba
```

**Sample Output:**
```text
2
```

---

## 96. Smallest Subsequence of Distinct Characters (Medium)

**Problem Statement:** Given a string **S**, return the lexicographically smallest subsequence that contains all distinct characters of **S** exactly once.


**Input Format:**

- One line containing **S** (lowercase).


**Output Format:**

- Print the subsequence.


**Sample Input:**
```text
cbacdcbc
```

**Sample Output:**
```text
acdb
```

---

## 97. Evaluate Simple Expression (+ and -) (Medium)

**Problem Statement:** Given an expression string **E** containing non-negative integers, `+`, `-`, and spaces, evaluate it.


**Input Format:**

- One line containing **E**.


**Output Format:**

- Print the integer result.


**Sample Input:**
```text
12 + 3 - 4
```

**Sample Output:**
```text
11
```

---

## 98. Evaluate Expression with Parentheses (+ and -) (Medium)

**Problem Statement:** Given an expression string **E** containing integers, `+`, `-`, parentheses `()`, and spaces, evaluate it.


**Input Format:**

- One line containing **E**.


**Output Format:**

- Print the integer result.


**Sample Input:**
```text
(1 + (4 + 5 + 2) - 3) + (6 + 8)
```

**Sample Output:**
```text
23
```

---

## 99. Check Balanced Brackets with '*' Wildcard (Medium)

**Problem Statement:** Given a string **S** containing `'('`, `')'`, and `'*'`, where `'*'` can act as `'('`, `')'`, or empty, determine if the string can be valid parentheses.


**Input Format:**

- One line containing **S**.


**Output Format:**

- Print **"YES"** or **"NO"**.


**Sample Input:**
```text
(*))
```

**Sample Output:**
```text
YES
```

---

## 100. Minimum Characters to Remove to Make All Character Frequencies Unique (Medium)

**Problem Statement:** Given a string **S**, delete the minimum number of characters so that the frequencies of remaining characters are all unique.


**Input Format:**

- One line containing **S** (lowercase).


**Output Format:**

- Print the minimum deletions.


**Sample Input:**
```text
aaabbbcc
```

**Sample Output:**
```text
2
```
