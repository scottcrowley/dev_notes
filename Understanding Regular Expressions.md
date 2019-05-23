# Understanding Regular Expressions

## Notes from Laracast [Understanding Regular Expressions](https://laracasts.com/series/understanding-regular-expressions)

* ### Miscellaneous
    * Website to test regular expressions <https://regexr.com/>
* ### Matching symbols
    * expressions begin and end with `/`
    * `$` matches the end of a string or end of line if the multiline flag is being used.
        * `\w+$` matches "she sells `seashells`"
    * `.` specifies a character ( matches any character except a line break )
    * `+` one or more of. Depends on the previous character to test for matches.
    * `*` matches zero or more of the previous character
    * `\` escapes the next character so certain characters are considered literal and not a matching symbol
        * The following characters are special regex characters and should be escaped
            * `+*?^$\.[]{}()|/`
    * `[]` Character Sets - matches anything within the braces. i.e. `[A-Z]` matches any uppercase letter or `[a-z]` matches any lowercase letter
        * Each character within the braces is considered a potential match. i.e. `[eman]` will match either an `e` or `m` or `a` or `n`
        * `[aeiou]` matches "gl`i`b j`o`cks v`e`x dw`a`rv`e`s!"
        * `[^aeiou]` matches "`gl`i`b j`o`cks v`e`x dw`a`rv`e`s!`"
    * `^` not. i.e. `[^eman]` matches anything that isn't an `e`, `m`, `a` or `n`.
        * When `^` is used on it's own that means at the start of the string. i.e. `^abc` = match `abc` at the start of the string
            * `^\w+` matches "`she` sells seashells"
    * `\s` matches any whitespace character like a `space`, `tab` or `line breaks`
        * `\S` opposite
    * `\w` matches any word character (alphanumeric & underscore)
        * `\w` matches "`bonjour`, `mon` `frère`" not the spaces
        * `\W` opposite
            * `\W` matches "bonjour`, `mon frère" includes the spaces
    * `\d` matches any digit character (alphanumeric & underscore)
        * `\D` opposite
    * `\t` matches a tab
    * `\n` matches a line feed
    * `\r` matches a carriage return
    * `\b` matches a word boundary position such as whitespace, punctuation or the start/end of a string
        * `s\b` matches "she sell`s` seashell`s`"
        * `\B` not word boundary
            * `s\B` matches "`s`he `s`ells `s`ea`s`hells"
    * `{}` quantifier. Matches the specified number between the brackets.
        * `{3}` matches next `3`
        * `{3,5}` matches the next `3 to 5`
        * `{3,}` matches the next `3 or more`
    * `|` (Pipe character) is used like an `or`
    * `?` optional symbol ( zero or 1 match on the previous symbol). States that the previous character is optional. i.e. `/(https?://)/g` would match either `http://` or `https://` and assign the match to `$1`
    * `\1` back reference to `group 1`
    * `?:` Non-capturing Group - when used within a group states that the group does not need to be captured
        * `/((?:ht|f)tps?://([\s]+))/g` will match `https://`, `http://` `ftp://` followed by the `url`. 
            * `(?:ht|f)` Will be ignored as a captured group
            * The entire matching url will be captured to `group $1` and the url after the "`//`" will be captured to `group $2`
* ###  Grouping
    * Any pattern wrapped in parentheses is considered a matching group and can be referenced with a `$` followed by the group order digit
* ### Flags
    * `g` at the end of the expression, that specifies a `global` which retains the index of the search allowing for multiple matches
        * This flag is not needed when using php `preg_replace`
    * `m` multiline flag
    * `i` at the end of the expression, that specifies that the matches are `case insensitive`.
* ### Lookaheads and Lookbehinds
    * Lookaheads
        * `?=` Positive lookaheads match a group after the main expression without including it in the result
        * `/google(?=<)/ig` will match the only `google` that is followed by a "`<`" symbol and won't include it in the result.
            * Given: `<a href="google.com">Google</a>` Google.com is the best search engine.
                * The expression would only match `Google` that is between the anchor tags.
        * `?!` Negative lookahead specifies a group that can not match after the main expression (if it matches the result is discarded)
            * The example given for Positive lookaheads, the google in the href and the word Google in the following sentence would be matched and not the Google between the anchor tags.
    * Lookbehinds
        * `Javascript` does not support lookbehinds but `PHP` does
        * `(?<=)` Positive Lookbehind format.
            ```
            $string = '$name = "Quaid"; My name is Quaid.';
            //replace the found expression with VARIABLE
            $result = preg_replace('/(?<=\$)name/', 'VARIABLE', $string);
            var_dump($result);
            //$VARIABLE = "Quaid"; My name is Quaid.
            ```
        * `(?<!)` Negative Lookbehind
            * Change the regex in the above example to '`/(?<!\$)name/`'
                ```
                var_dump($result);
                //$name = "Quaid"; My VARIABLE is Quaid.
                ```
* ### PHP Functions
    * `preg_match(regex, $string)` if no third argument is provided then it will return a boolean on whether or not it found any matches. This function will only return the first match
        * if a variable name is provided as a third argument then the actual matches will be available within that variable. `preg_match(regex, $string, $matches)`
    * `preg_match_all(regex, $string)` same as `preg_match` except that it will return all matches if the `$matches` argument is passed or the number of matches found.
    * `preg_replace(regex, $replaceStr, $string)` assumes that you want a global check. This function will return the string with all the replacements performed that had matches
    * `preg_split(regex, $replaceStr, $string)` Used on a string when you want to split the string into an array based on the passed expression. Same as `explode` except it uses regular expression
    * `preg_grep(regex, $array)` Returns array entries that match the pattern. Same as running `array_filter` and performing a `preg_match` within the closure.
