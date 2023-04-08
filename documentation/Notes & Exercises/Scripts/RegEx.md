## Regular Expressions

| Syntax  | Meaning                                                      |
| ------- | ------------------------------------------------------------ |
| `.`     | Wildcard:  Matches any single character                      |
| `[]`    | Set marker.  Matches only characters inside the brackets to a single character. |
| `[^]`   | Negate:  Used inside brackets to negate the set match        |
| `[x-y]` | Range:  Specify a sequential range of case-sensitive alpha-numeric characters.  ex. `[1-4], [^A-M]`.  Multiple ranges can be included in the same brackets `[1-47-9a-d]` |
| `\`     | Escape character:  Define a literal character follows or to specify metacharacters |
| `{}`    | Repetition marker:  Specifies the number of times the preceding character or group repeats.  A comma separated range can be specified.  ex.  Exactly three w's `w{3}`, 1-5 digits `\d{1,5}`,  3-4 of any character `.{3,4}`.  <br /><br />Note:  This is not supported with all regex implementations. |
| `*`     | Kleene Star:  Matches zero or more repetitions of the previous character or group |
| `+`     | Kleene Plus:  Matches one or more repetitions of the previous character or group |
| `?`     | Optional:  Matches on zero or one occurrence of the preceding character or group |
| `()`    | Capture group:  Captures text matching expression inside parenthesis.  Can be used multiple times and also can be nested.  These are captured in the order of the open parenthesis character. |
| `(|)`   | Logical OR:  Can match on left OR right character.  Used inside parenthesis.  More than 2 criteria can be defined, each separated by the pipe `|`.  Ex. `1|2|3` |
|         |                                                              |

#### Special Characters

| Syntax | Meaning                   |
| ------ | ------------------------- |
| `␣`    | Space                     |
| `\t`   | Tab                       |
| `\n`   | New line                  |
| `\r`   | Carriage return (Windows) |
| `^`    | Start of a line           |
| `$`    | End of a line             |


#### Metacharacters

| Syntax | Meaning                                                      |
| ------ | ------------------------------------------------------------ |
| `\d`   | any digit 0 to 9                                             |
| `\D`   | any non-digit character                                      |
| `\w`   | character range *[A-Za-z0-9_]*                               |
| `\W`   | any non-alphanumeric character                               |
| `\s`   | Matches space (`␣`),  the tab (`\t`), the new line (`\n`) and the carriage return (`\r`) |
| `\S`   | any non-whitespace character                                 |
| `\b`   | boundary between word and non-word character                 |

#### Back Referencing

This is largely dependent on the implementation of regex, but you can reference captured groups using the characters below.  This is particularly useful for tasks such as swapped text, modifying and replacing text, and so on.

| Syntax | Meaning                     |
| ------ | --------------------------- |
| `\0`   | Full matched text           |
| `\1`   | Matched group 1             |
| `\2`   | Matched group 2 (and so on) |
|        |                             |



### RegEx Testing

Use this tool to test your expressions on a string.

https://regex101.com/

### Further Reading

This is a great basic tutorial site with inline practice exercises:  https://regexone.com/

This is a good reference for all things RegEx and shows examples specific to different languages and scripts:  http://www.regular-expressions.info/reference.html

This is a regex game that gives you more advanced practice: http://play.inginf.units.it/#/

This site goes <u>deep</u> on regular expressions:  http://www.rexegg.com/