Hacky McHacker is the son of the most renowned programmers in Lambda Land. He has just started picking programming up and his ambitious parents have given him the task to output coloured text on the terminal of his Linux box. Hacky, confused and at his wits' end, has decided to ask for a solution in a well established programmers' forum. Following a kind question in InfiniteRecursion, you have seen it. You've passed through the usual concerns whether to downvote him for asking the obvious, or being a student looking for an easy solution to their homework problem. Now you are convinced Hacky has given a fair try, because he has read the pertinent pages in WikiLambdia and as a result has already defined some constants to be used as colours and several ([stubs](https://en.wikipedia.org/wiki/Stub)) for functions like `colorize` and `bleach`. The young McHacker has also divided the problem into subproblems, but unfortunately, is not able to implement the functions.


### Exercise #1

From Hacky's description and the [link to WikiLambdia](https://en.wikipedia.org/wiki/ANSI_escape_code), it becomes apparent that printing coloured text on the terminal is not a hard task to do. All styling is done by a special escape symbol (`Escape`) followed by a numeric code of the style. "Commands" formatted in this way are interleaved through the normal text, instructing the terminal to style everything following them.

As a beginner in Haskell, Hacky doesn't even know the function for converting an `Int` into a `String`. That's why he has declared the following two functions:

```hs
getLastDigit  :: Int -> Int
dropLastDigit :: Int -> Int
```

`getLastDigit` returns the last digit of a given number. Or, in a mathemetical sense, the remainder after division by 10.

```hs
getLastDigit 12345
> 5

getLastDigit 7
> 7

getLastDigit 0
> 0
```

Hint: Haskell doesn't have an operator for the modulo operation as the family of C-like languages do. Just use the function `mod`.

`dropLastDigit` is exactly the opposite of `getLastDigit` - it removes the last digit off of a number. Or again, in a mathematical sense, the whole part after division by 10.

Hint: The usual division operator `/` would not work here, because in Haskell it is defined to accept only rational numbers. Please refer to the lecture for other options.

```hs
dropLastDigit 12345
> 1234

dropLastDigit 7
> 0

dropLastDigit 0
> 0
```


### Exercise #2

Now that we already have the means of splitting a number into its parts, we have to assemble a list of them. We'll use it later for character conversion. The natural recursive solution yields a list with the digits in reverse order. That's fine for now.

The algorithm is as follows:
 - if the number (`x`) is less than zero, then apply the algorithm to `(-x)`
 - if `x` is less than 10, return a singleton list containing just `x`
 - or else, take the last digit of `x` and prepend it to the result of the recursive invocation with the rest of `x`'s digits

Expected results:
```hs
getReverseDigits 12345
> [5, 4, 3, 2, 1]

getReverseDigits (-12345)
> [5, 4, 3, 2, 1]

getReverseDigits 7
> [7]

getReverseDigits 0
> [0]
```


### Exercise #3

A function mapping a digit to its character is what we need next. The solution is not very elegant, but Haskell is very stubborn here and doesn't allow us to add a number to a character as we would usually do. That's why we'll settle for an easier solution - doing pattern matching on the digits and returning the needed result. If we define the function in this manner it would not be valid for all input numbers. Such a property is not a good property and is referred to as the function being _partial_. Haskell issues a warning, making us cautious of the dangers. The solution is to make the function _total_ by adding a pattern to admit all other values and throw an error. It is worth noting that the function hasn't transformed into a _total_ function in an operational sense (i.e. it still errs for inputs different from a digit) but it is _total_ from the compiler's perspective.

Use the following pattern as a final option:
```hs
toChar _ = error "Not a digit"
```

Expected results:
```hs
toChar 1
> '1'

toChar 9
> '9'

toChar 10
> *** Exception: Not a digit
```


### Exercise #4

We finally have all the needed sub-functions to write

```hs
itoa :: Int -> String
```

This is the time and place to deal with the reverse ordered digit list. We'll define the following helper function:

```hs
itoaLoop :: String -> [Int] -> String
```

The first argument is of type `String` - we'll keep the running result here. Such a parameter is called an _accumulator_, because the result is accumulated in it. The algorithm is very simple:
 - if the list of digits is empty - return the accumultor
 - or else, invoke the algorithm recursively with the first argument being the first digit prepended to the accumulator and the second argument being the rest of the digits

`itoa` is defined trivially via `itoaLopp` (just initialize the accumulator). This technique is very commonly used in functional programming. However, not every helper function needs an accumulator (they are more widely useful, e.g. for emulating a loop) and sometimes we use accumulators outside of helper functions.

```hs
itoaLoop "" [3, 2, 1]
  -> itoaLoop "3" [2, 1]
    -> itoaLoop "23" [1]
      -> itoaLoop "123" []
> "123"
```

```hs
itoa 12345
> "12345"

itoa 7
> "7"

itoa 0
> "0"
```


### Exercise #5

`mkStyle` and `mkTextStyle` are already implemented by another forum member.

`mkStyle` accepts a single argument - style code and returns a formatted string according to the terminal's specification. In this function the operator `++` is used for glueing (concatanating) two lists.

`mkTextStyle` accepts a colour code and returns the corresponding style. The colours are defined as one of eight predefined options. The constant `textStyle` is an offset such that when added to a color results in a style that modifies the text colour according to the ASCII standard (text styles are in the 30~37 range).

```hs
mkStyle :: Int -> String
mkStyle style = "\x1B[" ++ itoa style ++ "m"


mkTextStyle :: Int -> String
mkTextStyle color = mkStyle (color + textStyle)
```

You have to implement the function
```hs
getStyle :: String -> String
```

It accepts one of the following `String` constants as input and returns the corresponding style string.

| String | Color   |
| ------ | ------- |
| "blk"  | black   |
| "red"  | red     |
| "grn"  | green   |
| "ylw"  | yellow  |
| "blu"  | blue    |
| "mgt"  | magenta |
| "cyn"  | cyan    |
| "wht"  | white   |
| "clr"  | clear<sup>[*](#getStyle-clear)   |

<a name="getStyle-clear">*</a> `clr` will be used to cleanse all styles. Take a look at the appropriate constant and the function for creating generic (non-offseted) style.

If the input is not one of the above constants, the function should return the input unchanged surrounded by angle brackets `<input>`.

Example results:
```hs
getStyle "bkl"
> "\x1B[30m"

getStyle "blu"
> "\x1B[34m"

getStyle "clr"
> "\x1B[0m"

getStyle "other"
> "<other>"
```


### Exercise #6

All parts of the puzzle are already present, they just need to be assembled. One of the senior forum members has implemented the function for removing styles - `bleach`, but not `colorize`. The predominant understanding is that Hacky should walk the last mile alone. A hint is given that `colorize` is very alike `bleach` with just a few minor (but logically significant) differences. `colorize` accepts a single `String` argument and replaces all occurences of "keywords" in it with their values. The keywords are surrounded by angle brackets `<>` and are the constants in the table from above. For exmaple `"<red>hello"` gets transformed into `"\x1B[30mhello"`. If the keyword is not one of the known ones it should be left intact. Notice that all keywords are of fixed length and are surrounded by predefined markers. Colorize does not need any extra auxiliary functions - everything needed is already defined.

```hs
colorize "<red>hello<clr>"
> "\x1B[30mhello"

colorize "<red>hello <blue>world<clr>"
> "\x1B[30mhello \x1B[34mworld\x1B[0m"

colorize "<haskell><hsk>"
> "<haskell><hsk>"
```

Printing escape symbols is not very fun. To see the real colours use `putStrLn` as in:
```hs
putStrLn (colorize "<red>hello<clr>")
```

What happens if you omit `<clr>`?

### Bonus exercise

This exercise adds further improvements and ideas for extensions. You'll add an option to change the background in addition to the colour of the text. You can continue adding more features such as font-weight modifiers, helper functions that generate the _markup_ (_\<keyword\>_) and others. 

The following functions should be your guide:

```hs
mkBackgroundStyle :: Int -> String
dropMarkup :: String -> String
getMarkup :: String -> String
colorize2 :: String -> String
```

`mkBackgroundStyle` is a function that accepts a colour code and returns a style. Implement it in the spirit of `mkTextStyle`.

`dropMarkup` - this function skips (drops) everything up to (and including) the closing markup symbol (`>`). Take a look at `removeStyle`. `dropMarkup`'s implementation should have the same code but only with a single minimal change. To further broaden your knowledge search for the documentation of `drop` and `dropWhile`. `dropWhile` takes a condition (predicate) and skips elements while the condition holds. `drop` is much simpler - it skips the next `n` elements (e.g. `1` in the case of `removeStyle`).

`getMarkup` - this function extracts the contents locked inbetween `<>`. The intended usage is by `colorize2` when you already know that a keyword follows (pattern matching?). This function is the complement of `dropMarkup`. Instead of `dropWhile` use `takeWhile` - the alternative for collecting elements while a condition holds.

`colorize2` uses the above auxiliary functions in order to extract the keywords in a much more lenient manner. Don't forget to add new background keywords to `getStyle`. One way to do that is to use the prefix `bgr-` for the new keywords.

Example results:
```hs
getStyle "bgr-red"
> "\x1B[41m"

putStrLn (colorize2 "<bgr-wht><blk>black <bgr-blk><wht> white<clr>")
```

PS: What bug the new implementation of `colorize2` has? Can you fix it?
