**In Java Script we have different methods to get sub string:**
## 1. substring(start, end?)
- The `substring(start, end?)` is a build-in method that use to get part of string.
- The `substring()` method used the indices to get the substring as it accepts:
	- `start`: the index of the start point which will be included.
	- `end?`: the index of the end point which _will not be included_.
- If we didn't provide the end point, it will continue till the last index of the string.
- If the start is bigger than the end the `substring()` will automatically swap between them to make the start point the smallest value.
- Any negative value will be treated as _zero_.


```JavaScript
var mystr = "Mahmoud Saadallah";
console.log(mystr.substring(0, 3)); // Mah
console.log(mystr.substring(3, 9)); // moud S
console.log(mystr.substring(5)); // ud Saadallah

console.log(mystr.substring(5, 1)); // ahmo  // Swap(5, 1) => (1, 5) .
console.log(mystr.substring(-5, 3)); // ahmo  // negative number will be as zero.
console.log(mystr.substring(-19)); // Mahmoud Saadallah
```

---
## 2. substr(from, length?)
- As the `substring()` the `substr(from, lenght?)` uses indices to get the sub string.
	- `from`: the index of the start point which will be included.
	- `length?`: the length of the sub string.
- If we didn't provide the length, it will continue till the last index of the string.
- Any negative value will make `substr()` start count from the end.

```JavaScript
var mystr = "Mahmoud Saadallah";
console.log(mystr.substr(0, 3)); // 'Mah'
console.log(mystr.substr(3, 9)); // 'moud Saad'
console.log(mystr.substr(5)); // 'ud Saadallah'

console.log(mystr.substr(5, 1)); // 'u' 
console.log(mystr.substr(-4, 3)); // 'lla'  // negative start count from the end.
console.log(mystr.substr(-9)); // 'Saadallah'
```

---
## 3. slice(start, end?)
- `slice(start, end?)` work like `substring()` in most of situations.
	- `start`: the index of the start point which will be included.
	- `end?`: the index of the end point which _will not be included_.
- If we didn't provide the end point, it will continue till the last index of the string.
- If the start is bigger than the end the `slice()` will return `''` empty string.
- Any negative value will make `substr()` start count from the end in condition of the `abs(start)` less than `end?` and the `end?` is greater than `start`

```JavaScript
var mystr = "Mahmoud Saadallah";
console.log(mystr.slice(0, 3)); // Mah
console.log(mystr.slice(3, 9)); // moud S
console.log(mystr.slice(5)); // ud Saadallah

console.log(mystr.slice(5, 1)); // ahmo  // Swap(5, 1) => (1, 5) .
console.log(mystr.slice(-5, 3)); // ''
console.log(mystr.slice(-5, 8)); // '' // as the index 7 which is 'S' is before -5 which is 'a'
console.log(mystr.slice(-5, 14)); // 'al'
console.log(mystr.slice(-19)); // Mahmoud Saadallah
```
