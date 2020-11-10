---
title: regex
toc: true
date: 2020-10-20 15:20:09
tags:
	- regex
---

[learn source link](https://www.sololearn.com/play/python)

# MetaCharacters

`.`: matches **any character**, other than a new line.

`^`: match the **start** of a string. It means **not in** when used in character class, see [character classes](#character-classes). In some other languages, e.g. js, it will treat `^` as the start of a line, which is different from `\A` (see [special sequences](#special-sequences) below), see [stackoverflow](https://stackoverflow.com/questions/577653/difference-between-a-z-and-in-ruby-regular-expressions) for detail.

`$`: match the **end** of a string.

`|`: means or, e.g. `red|blue` matches `red` or `blue`



MetaCharacters that **match count of** previous thing (The "previous thing" can be a single character, a class, or a [group](#group) of characters in parentheses. eg. `(abc)*def` matches `def` or `abcdef` or `abcabcdef`...):

`*`: zero or more repetitions of the previous thing. 

`+`: one or more repetitions

`?`: zero or one repetitions

`{x,y}`: between x and y repetitions of something. `?` is the same thing as `{0,1}`.  If the first number is missing, it is taken to be zero. If the second number is missing, it is taken to be infinity.



# Character Classes<a name="character-classes" />

**Character classes** provide a way to match only one of a specific set of characters.
A character class is created by putting the characters it matches inside **square brackets**.

e.g. 

* `a[123]`: matches any of a1,a2,a3
* `a[\d.]`: matches any number or dot, eg. a1,a.,a2



Character classes can also match **ranges of characters.**
e.g.

* `[a-z]` matches any lowercase alphabetic character.
*  `[G-P]` matches any uppercase character from G to P.
*  `[0-9]` matches any digit.

Multiple ranges can be included in one class. For example, `[A-Za-z]` matches a letter of any case.



Place a **^** at the start of a character class to **invert** it.

e.g.

* `[^A-Z]`: matches any not A-Z

# Group<a name= "group" />

A group can be created by surrounding **part of a regular expression** with **parentheses**.

In python re module:
A call of **group(0)** or **group()** returns the whole match.
A call of **group(n),** where **n** is greater than 0, returns the **n**th group from the left. The order of group is the same as the order of left parenthese.
The method **groups()** returns all groups up from 1.

## Special Groups

Two useful special groups are **named groups** and **non-capturing groups**.
**Named groups** have the format **(?P<name>...)**, where **name** is the name of the group, and **...** is the content. They behave exactly the same as normal groups, except they can be accessed by **group(name)** in addition to its number.
**Non-capturing groups** have the format **(?:...)**. They are not accessible by the group method, so they can be added to an existing regular expression without breaking the numbering.

e.g.:

```python
import re

pattern = r"(?P<first>abc)(?:def)(ghi)"

match = re.match(pattern, "abcdefghi")
if match:
   print(match.group("first"))
   print(match.groups())
```

# Special Sequences<a name="special-sequences" />

 **Special sequences** are written as a backslash followed by another character.

* a backslash and a number between 1 and 99, e.g., \1 or \17. This matches the expression of the group of that number. 
  *  `ab(abc)((cd)ef)\2`: matches `ababccdefcdef`, the `\2` match the 2nd group, that is the (cdef) group
* **`\d`**: matches digits. In ASCII mode they are equivalent to **[0-9]** (In Unicode mode it may match certain other characters, as well.)
  * `\D`: opsite of `\d`, matches anything that isn't digit
* **`\s`**: matches whitespace. In ASCII mode they are equivalent to **[ \t\n\r\f\v]** (In Unicode mode it may match certain other characters, as well.)
  * `\S`: opsite of `\s`, matches anything that isn't whitespace
* **`\w`**: matches word characters. In ASCII mode they are equivalent to **[a-zA-Z0-9_]**  (In Unicode mode it matches letters with accents, as well.)
  * `\W`: opsite of `\w`, matches anything that isn't word characters
* **`\A`** & **`\Z`**: matches the beginning and end of a string.  In some other languages, e.g. js, it will treat `^` as the start of a line, while `\A` as the start of a string. And there's `\z` and `\Z`, see [stackoverflow](https://stackoverflow.com/questions/577653/difference-between-a-z-and-in-ruby-regular-expressions) for detail.
* **`\b`**:  matches a word boundary. That is defined as the transition from a word character (`\w`) to a non-word character (`\W`), or vice versa. It's a zero-width match (just like \A and\Z and various other constructs) which means it matches the gap between two characters.
  * e.g. `\AS...\b.\Z` matches `SPAM!`, because M is a word character, and ! is not a word character, so the gap between them is a word break, which \b matches. The final . matches the !.
  * `\B`: opsite of `\b`, matches any transition that is not between `\w` & `\W`, or `\w` & the beginning/end of the string.




