<style>
	h1 {
        display: none;
	}
</style>
## 65. Valid Number

> Validate if a given string can be interpreted as a decimal number.
>
> Some examples:
> `"0"` => `true`
> `" 0.1 "` => `true`
> `"abc"` => `false`
> `"1 a"` => `false`
> `"2e10"` => `true`
> `" -90e3   "` => `true`
> `" 1e"` => `false`
> `"e3"` => `false`
> `" 6e-1"` => `true`
> `" 99e2.5 "` => `false`
> `"53.5e93"` => `true`
> `" --6 "` => `false`
> `"-+3"` => `false`
> `"95a54e53"` => `false`
>
> **Note:** It is intended for the problem statement to be ambiguous. You should gather all requirements up front before implementing one. However, here is a list of characters that can be in a valid decimal number:
>
> - Numbers 0-9
> - Exponent - "e"
> - Positive/negative sign - "+"/"-"
> - Decimal point - "."
>
> Of course, the context of these characters also matters in the input.

### Intuition

The valid number orderly consists of:

- 0 or more spaces `\s*`
- 0 or 1 of "-" or "+" `[-+]?`
- Some digits from eighter:
  - 1 or more digits `\d+`
  - 1 or more digits + "." + 1 or more digits `\d+.\d+`
- 0 or 1 of ("e" + 0 or 1 of "-" or "+" +  1 or more digits) `e[-+]?\d+`

`^\s*[-+]?(\d+|\d+.\d+)(e[-+]?\d+)?$`

### Some Changes

Actually, the OJ treate ".3" or "1." as valid numbers too, but "." Is not valid

 so

The valid number orderly consists of:

- 0 or more spaces `\s*`
- 0 or 1 of "-" or "+" `[-+]?`
- Some digits from eighter:
  - <span style="color:red">0</span> or more digits `\d+`
  - <span style="color:red">0</span> or more digits + "." + 1 or more digits `\d+.\d+`
  - 1 or more digits + "." + <span style="color:red">0</span>  or more digits `\d+.\d+`
- 0 or 1 of ("e" + 0 or 1 of "-" or "+" +  1 or more digits) `e[-+]?\d+`

`^\s*[-+]?(?:\d+|\d*\.\d+|\d+\.\d*)(?:e[-+]?\d+)?\s*$`



## 564. Find the Closest Palindrome

> Given an integer n, find the closest integer (not including itself), which is a palindrome.
>
> The 'closest' is defined as absolute difference minimized between two integers.
>
> **Example 1:**
>
> ```
> Input: "123"
> Output: "121"
> ```
>
>
>
> **Note:**
>
> 1. The input **n** is a positive integer represented by string, whose length will not exceed 18.
> 2. If there is a tie, return the smaller one as answer.

- Get first half of the number "L"
  - if the number is of size 6, get leading 3 numbers 123456 => 123
  - if the number is if size 9, get leading 5 numbers 123456789 => 12345

- The result must be one of the number leading by either
  - L
  - L+1
  - L-1
- We have to 