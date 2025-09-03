# let's start solve daily one problem in js 
### String
##### 1. reverse a string 

```js
let stringname ="hello"
function reverseString(str) {
    let reversedString = "";
    for (let i = str.length - 1; i >= 0; i--) {
        reversedString += str[i];
    }
    return reversedString;
};
function reverseString(str) {
    let arr = str.split("");
    let start = 0;
    let end = arr.length - 1;
    while (start < end) {
        [arr[start], arr[end]] = [arr[end], arr[start]];
        start++;
        end--;
    }

    return arr.join("");
}
```
