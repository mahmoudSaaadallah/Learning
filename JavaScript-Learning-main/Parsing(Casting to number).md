## Casting to integer
* To cast a string in Js to integer we could use parseint() method that accept any data type and convert it to integer if it possible.

```JavaScript
var x = parseInt("258.90");
console.log(x, typeof x); //258 'number'
```

* We have to know that even if the string that we want to parse to integer has a characters the parseInt() method will work.

```JavaScript
var str = parseInt("23.3abc");
console.log(str, typeof str); // 23 'number'
```

* As we saw in the previous example we it also work even with string with character.
* But we have to know how the parseInt() method work when the string contains characters?

- _The pareseInt() will start by checking the first character in the parameter, if it a number it goes to the next number and so on, until it find a decimal point or a character then it will stop and return the result._

- <span style="color:rgb(255, 255, 0)">What if the first char in the string was a letter or a decimal point?</span>
- Then it will stop directly and return NaN(Not a Number);

```JavaScript
var str = parseInt("a23.3abc");
console.log(str, typeof str); // NaN 'number'
```

- As we saw in this example as the string started with letter it returns NaN, because the parseInt() method didn't find number.
- We also have to know that NaN is a type of number, even if it Not a Number, but it's under the Number class. [[NaN(Not a Number)]].


------- 

## Casting to float
* To cast a string in Js to float we could use parseFloat() method that accept any data type and convert it to float if it possible.

```JavaScript
var x = parseFloat("258.90");
console.log(x, typeof x); //258.90 'number'
```

* We have to know that even if the string that we want to parse to float has a characters the parseFloat() method will work.

```JavaScript
var str = parseFloat("23.3abc");
console.log(str, typeof str); // 23.3 'number'
```

* As we saw in the previous example we it also work even with string with character.
* But we have to know how the parseFloat() method work when the string contains characters?

- <mark style="background: #FFB86CA6;">The parseFloat() work exactly as the parseInt() work.</mark>

```JavaScript
var str = parseFloat("a23.3abc");
console.log(str, typeof str); // NaN 'number'
```

- As we saw in this example as the string started with letter it returns NaN, because the parseFloat() method didn't find number.
- We also have to know that NaN is a type of number, even if it Not a Number, but it's under the Number class.


---

## Casting With Number()
- In addition to cast using parsing we could cast using the Number class.
- We have to know that casting using Number will cast to integer and float depend on the value that we want to cast.
- If it contains decimal point the Number will cast it to float, else it will cast it to int.
- <span style="color:rgb(255, 255, 0)">What is the different between casting using parsing and Number class?</span>
- _Casting with Number() is more restricted than parsing, as if the string that we want to cast contains any character or any thing except numbers it will return NaN._

```JavaScript
var str = Number("23.3");
console.log(str, typeof str); // 23.3 'number'
```

```JavaScript
var str = Number("23.3abc");
console.log(str, typeof str); // NaN 'number'
```

- Here it return NaN, because the string contains characters not only numbers.

- learn How to check NaN? [[NaN(Not a Number)]].