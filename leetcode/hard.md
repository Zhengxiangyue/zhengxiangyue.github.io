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

The valid number is orderly consist of:

- 0 or more spaces `\s*`
- 0 or 1 of "-" or "+" `[-+]?`
- Some digits from eighter:
  - 1 or more digits `\d+`
  - 1 or more digits + "." + 1 or more digits `\d+.\d+`
- 0 or 1 of ("e" + 0 or 1 of "-" or "+" +  1 or more digits) `e[-+]?\d+`

`^\s*[-+]?(\d+|\d+.\d+)(e[-+]?\d+)+$`

