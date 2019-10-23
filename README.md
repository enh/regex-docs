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

<table>
  <tr> <td> ^ </td> <td>At beginning of line.</td> </tr>
  <tr> <td> $ </td> <td>At end of line.</td> </tr>
  <tr> <td> \A </td> <td>At beginning of input.</td> </tr>
  <tr> <td> \b </td> <td>At word boundary.</td> </tr>
  <tr> <td> \B </td> <td>At non-word boundary.</td> </tr>
  <tr> <td> \G </td> <td>At end of previous match.</td> </tr>
  <tr> <td> \z </td> <td>At end of input.</td> </tr>
  <tr> <td> \Z </td> <td>At end of input, or before newline at end.</td> </tr>
</table>

## Look-around assertions

Look-around assertions assert that the subpattern does (positive) or doesn't (negative) match after (look-ahead) or before (look-behind) the current position, without including the matched text in the containing match. The maximum length of possible matches for look-behind patterns must not be unbounded.

<table>
  <tr> <td> (?=<i>a</i>) </td> <td>Zero-width positive look-ahead.</td> </tr>
  <tr> <td> (?!<i>a</i>) </td> <td>Zero-width negative look-ahead.</td> </tr>
  <tr> <td> (?&lt;=<i>a</i>) </td> <td>Zero-width positive look-behind.</td> </tr>
  <tr> <td> (?&lt;!<i>a</i>) </td> <td>Zero-width negative look-behind.</td> </tr>
  </table>

## Groups

<table>
  <tr> <td> (<i>a</i>) </td> <td>A capturing group.</td> </tr>
  <tr> <td> (?:<i>a</i>) </td> <td>A non-capturing group.</td> </tr>
  <tr> <td> (?&gt;<i>a</i>) </td> <td>An independent non-capturing group. (The first match of the subgroup is the only match tried.)</td> </tr>
  <tr> <td> \<i>n</i> </td> <td>The text already matched by capturing group <i>n</i>.</td> </tr>
  </table>

## Operators

<table>
  <tr> <td> <i>ab</i> </td> <td>Expression <i>a</i> followed by expression <i>b</i>.</td> </tr>
  <tr> <td> <i>a</i>|<i>b</i> </td> <td>Either expression <i>a</i> or expression <i>b</i>.</td> </tr>
 </table>

## Flags

<table>
 <tr> <td> (?dimsux-dimsux:<i>a</i>) </td> <td>Evaluates the expression <i>a</i> with the given flags enabled/disabled.</td> </tr>
  <tr> <td> (?dimsux-dimsux) </td> <td>Evaluates the rest of the pattern with the given flags enabled/disabled.</td> </tr>
  </table>

The flags are:

<table>
  <tr><td>`i`</td> <td>{@link #CASE_INSENSITIVE}</td> <td>case insensitive matching</td></tr>
  <tr><td>`d`</td> <td>{@link #UNIX_LINES}</td>       <td>only accept {@code '\n'} as a line terminator</td></tr>
  <tr><td>`m`</td> <td>{@link #MULTILINE}</td>        <td>allow {@code ^} and {@code $} to match beginning/end of any line</td></tr>
  <tr><td>`s`</td> <td>{@link #DOTALL}</td>           <td>allow {@code .} to match {@code '\n'} ("s" for "single line")</td></tr>
  <tr><td>`u`</td> <td>{@link #UNICODE_CASE}</td>     <td>enable Unicode case folding</td></tr>
  <tr><td>`x`</td> <td>{@link #COMMENTS}</td>         <td>allow whitespace and comments</td></tr>
  </table>

Either set of flags may be empty. For example, `(?i-m)` would turn on case-insensitivity and turn off multiline mode, `(?i)` would just turn on case-insensitivity, and `(?-m)` would just turn off multiline mode.
