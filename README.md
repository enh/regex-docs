# Regular Expression Syntax Quick Reference

Java supports a subset of Perl 5 regular expression syntax. An important gotcha is that Java has no regular expression literals, and uses plain old string literals instead. This means that you need an extra level of escaping. For example, the regular expression `\s+` has to be represented as the string `"\\s+"`.

## Escape sequences

| Escape                 | Meaning                                                                   |
| ---------------------- | ------------------------------------------------------------------------- |
| `\`                    | Quote the following metacharacter (so `\.` matches a literal `.`).        |
| `\Q`                   | Quote all following metacharacters until `\E`.                            |
| `\E`                   | Stop quoting metacharacters (started by `\Q`).                            |
| `\\`                   | A literal backslash.                                                      |
| `&#x005c;u`<i>hhhh</i> | The Unicode character U+hhhh (in hex).                                    |
| `&#x005c;x`<i>hh</i>   | The Unicode character U+00hh (in hex).                                    |
| `\c`<i>x</i>           | The ASCII control character ^x (so `\cH` would be ^H, U+0008).            |
| `\a`                   | The ASCII bell character (U+0007).     |
| `\e`                   | The ASCII ESC character (U+001b).       |
| `\f`                   | The ASCII form feed character (U+000c). |
| `\n`                   | The ASCII newline character (U+000a).          |
| `\r`                   | The ASCII carriage return character (U+000d).  |
| `\t`                   | The ASCII tab character (U+0009). |

## Character classes

It's possible to construct arbitrary character classes using set operations:

| Class            | Meaning |
| ---------------- | --------------- |
| `[abc]`          | Any one of `a`, `b`, or `c`. (Enumeration.) |
| `[a-c]`          | Any one of `a`, `b`, or `c`. (Range.)       |
| `[^abc]`         | Any character _except_ `a`, `b`, or `c`. (Negation.) |
| `[[a-f][0-9]]`   | Any character in either range. (Union.) |
| `[[a-z]&&[jkl]]` | Any character in both ranges. (Intersection.) |

Most of the time, the built-in character classes are more useful:

| Class            | Meaning |
| ---------------- | --------------- |
| `\d` | Any digit character (see note below). |
| `\D` | Any non-digit character (see note below). |
| `\s` | Any whitespace character (see note below). |
| `\S` | Any non-whitespace character (see note below). |
| `\w` | Any word character (see note below). |
| `\W` | Any non-word character (see note below). |
| `\p{`NAME`}` | Any character in the class with the given _NAME_. |
| `\P{`NAME`}` | Any character _not_ in the named class. |

Note that these built-in classes don't just cover the traditional ASCII range. For example, `\w` is equivalent to the character class `[\p{Ll}\p{Lu}\p{Lt}\p{Lo}\p{Nd}]`.

For more details see [Unicode TR-18](http://www.unicode.org/reports/tr18/#Compatibility_Properties), and bear in mind that the set of characters in each class can vary between Unicode releases. If you actually want to match only ASCII characters, specify the explicit characters you want; if you mean 0-9 use `[0-9]` rather than `\d`, which would also include Gurmukhi digits and so forth.

There are also a variety of named classes:

* Unicode category names (TODO: list) prefixed by `Is`. For example `\p{IsLu}` for all uppercase letters.
* POSIX class names. These are 'Alnum', 'Alpha', 'ASCII', 'Blank', 'Cntrl', 'Digit', 'Graph', 'Lower', 'Print', 'Punct', 'Upper', 'XDigit'.
* Unicode block names, as accepted as input to {@link java.lang.Character.UnicodeBlock#forName}, prefixed by `In`. For example `\p{InHebrew}` for all characters in the Hebrew block.
* Character method names. These are all non-deprecated methods from {@link java.lang.Character} whose name starts with `is`, but with the `is` replaced by `java`. For example, `\p{javaLowerCase}`.

## Quantifiers

Quantifiers match some number of instances of the preceding regular expression.

| Quantifier  | Meaning |
| ----------- | --------------- |
| `*`         | Zero or more. |
| `?`         | Zero or one.  |
| `+`         | One or more.  |
| `{`n`}`     | Exactly _n_ repetitions.  |
| `{`n`,}`    | At least _n_ repetitions. |
| `{`n`,`m`}` | At least _n_ and not more than _m_. |

Quantifiers are "greedy" by default, meaning that they will match the longest possible input sequence. There are also non-greedy quantifiers that match the shortest possible input sequence.

They're same as the greedy ones but with a trailing `?`:

| Quantifier   | Meaning |
| ------------ | --------------- |
| `*?`         | Zero or more (non-greedy). |
| `??`         | Zero or one (non-greedy).  |
| `+?`         | One or more (non-greedy).  |
| `{`n`}?`     | Exactly _n_ repetitions (non-greedy).  |
| `{`n`,}?`    | At least _n_ repetitions (non-greedy). |
| `{`n`,`m`}?` | At least _n_ and not more than _m_ (non-greedy). |

Quantifiers allow backtracking by default. There are also possessive quantifiers to prevent backtracking. They're same as the greedy ones but with a trailing `+`:

| Quantifier   | Meaning |
| ------------ | --------------- |
| `*+`         | Zero or more (possessive). |
| `?+`         | Zero or one (possessive).  |
| `++`         | One or more (possessive).  |
| `{`n`}+`     | Exactly _n_ repetitions (possessive).  |
| `{`n`,}+`    | At least _n_ repetitions (possessive). |
| `{`n`,`m`}+` | At least _n_ and not more than _m_ (possessive). |

## Zero-width assertions

| Assertion    | Meaning |
| ------------ | --------------- |
| `^`          | At beginning of line. |
| `$`          | At end of line. |
| `\A`         | At beginning of input. |
| `\b`         | At word boundary. |
| `\B`         | At non-word boundary. |
| `\G`         | At end of previous match. |
| `\z`         | At end of input. |
| `\Z`         | At end of input, or before newline at end. |

## Look-around assertions

Look-around assertions assert that the subpattern does (positive) or doesn't (negative) match after (look-ahead) or before (look-behind) the current position, without including the matched text in the containing match. The maximum length of possible matches for look-behind patterns must not be unbounded.

| Assertion    | Meaning |
| ------------ | --------------- |
| `(?=`e`)`    | Zero-width positive look-ahead. |
| `(?!`e`)`    | Zero-width negative look-ahead. |
| `(?<=`e`)`   | Zero-width positive look-behind. |
| `(?<!`e`)`   | Zero-width negative look-behind. |

## Groups

| Grouping     | Meaning |
| ------------ | --------------- |
| `(`e`)`      | A capturing group. |
| `(?:`e`)`    | A non-capturing group. |
| `(?>`e`)`    | An independent non-capturing group. (The first match of the subgroup is the only match tried.) |
| `\`n         | The text already matched by capturing group _n_. |

## Operators

| Operator     | Meaning |
| ------------ | --------------- |
| ab           | Expression _a_ followed by expression _b_. |
| a`|`b`       | Either expression _a_ or expression _b_. |

## Flags

| Expression             | Meaning |
| ---------------------- | --------------- |
| `(?dimsux-dimsux:`e`)` | Evaluates the expression _e_ with the given flags enabled/disabled. |
| `(?dimsux-dimsux)`     | Evaluates the rest of the pattern with the given flags enabled/disabled. |

The flags are:

| Flag  | Meaning |
| ----- | --------------- |
| `i`   | Case insensitive matching. |
| `d`   | Only accept `\n` as a line terminator. |
| `m`   | Multiline: allow `^` and `$` to match beginning/end of any line. |
| `s`   | Single line: allow `.` to match `\n`. |
| `u`   | Unicode case folding. |
| `x`   | Allow whitespace and comments. |

Either set of flags may be empty. For example, `(?i-m)` would turn on case-insensitivity and turn off multiline mode, `(?i)` would just turn on case-insensitivity, and `(?-m)` would just turn off multiline mode.
