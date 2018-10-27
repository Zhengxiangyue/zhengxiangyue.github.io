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
- Calculate the origin, +1, and -1, and the corresponding palindrome
  - 12345 for instance, first half 123, origin: 12321, +1: 12421, -1:12221
  - 1000 for instance , first half 10, origin: 1001, +1: 1111, -1: 999
- The result must be one of them



The first half is easy to calculate, use this function to get the palindrome where the second parameter is how many number we need to add to the back

```c++
    string getPalingrome(string s, int l) {
        
        if (s == "0") 
            return "9";
        
        if (l > s.size()) 
            return s + string(l, '9');
        
        string post_half(s.begin(), s.begin() + l);
        reverse(post_half.begin(), post_half.end());
        return s + post_half;
    }
```

Special conditions:

- If the number n <= 10, return n-1
- If the pre half is 0, return 9. When n like 11, 12, 13 and pre half could be 1, 0, 2 (if s == "0")
- If the length of the half decrease, we get some number like 10000, return 9999 (if l > s.size())



## 340. Longest Substring with At Most K Distinct Characters

> Given a string, find the length of the longest substring T that contains at most *k* distinct characters.
>
> **Example 1:**
>
> ```
> Input: s = "eceba", k = 2
> Output: 3
> Explanation: T is "ece" which its length is 3.
> ```
>
> **Example 2:**
>
> ```
> Input: s = "aa", k = 1
> Output: 2
> Explanation: T is "aa" which its length is 2.
> ```
>
>

Two pointers. At the beginning, left and right contain the first k characters, each time:

- Move the right pointer to the right by one position
  - If the range contains less or equal than k characters, continue, since it's a valid range
  - If not, move left pointer to right **until** the range contains k characters

In this process, use a hash map to record the occurrence of each character. Also update max length of the range