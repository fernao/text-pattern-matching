# About

This document is a compilation of commands and procedures in order to remove/replace text patterns and clean git repositories of sensitive information.

## Grep for finding patterns

Usages of grep for text pattern search:

Recursive search (-r) that outputs also the number of the line (-n) and performs a perl regular expression (-P), in the case, numbers of 11 digits:

```grep -rnP '\d{11}' .```

Grep output is thrown to another grep, that keeps only the lines where 'pattern' is present:

```grep -rnP '\d{11}' . | grep 'pattern' ```

Grep output is filtered to remove unwanted pattern

```grep -rnP '\d{11}' . | grep 'pattern' ``` | grep -v 'this data no!'

## Using sed to find a text pattern

Searching for a pattern of 3 digits (\{3\}) and output (/p) it on the screen (-n suppress other output except the match)

```sed -n '{/[[:digit:]]\{3\}/p}' filename```

Searching for a pattern of 3 digits and output it on the screen only at line 6

```sed -n '6{/[[:digit:]]\{3\}/p;}' filename```

Applies a regular expression replace (-e) at the same file (-i) that searches (s/from/to/g) at line 20, replacing numbers ([[:digit:]]) at size of 11 (\{11\}) and replaces then for XXXX, greedy (g). Note that inside sed you must escape the brackets

```sed -i -e 20's/[[:digit:]]\{11\}/XXXX/g' filename```

## Using cut and awk to process some output

When using shell programming, you can use different programs combinations in order to achieve your goals. For example, let's observe the 'cut' program. It's used to split a string by some char.

This will split a line by ':', using a ':' as separator. In this case, we're getting only the first match; second line, the second match; third, both of them:

```
grep -rnP '\d{11}' . | cut -d ':' -f1
grep -rnP '\d{11}' . | cut -d ':' -f2
grep -rnP '\d{11}' . | cut -d ':' -f1,2
```

If you want to efectivelly use the match data as a variable, you can use awk! So, you can get the same response doind the following:

```grep -rnP '\d{11}' . | awk -F ':' '{ print $1 ":" $2 }'```

... but if needed, you can use the $1 or $2 or whatelse as variables to your expression, using awk 'system' function. For example, if you want to find a pattern and replace it using sed. Let's take our sed output example.

As we're creating an sed expression inside an awk. So we've a lot of escaping... chars like (){} needs to be escaped. For more complex cases we can use octal codes for the chars: \47 for ", \173 for { and \175 for }. For more about char codes, see http://www.asciitable.com/. Here we're performing a search for a 3 char digit in a folder and output the matches to somewhere.

```grep -rnP '\d{3}' . | awk -F ':' '{ system("sed -ne "$2"\47\173s/\\([[:digit:]]\\{3\\}\\)/\\1/p; \175 \47 " $1) }' > somefile```

Woops, we've found some binary files that matched our result! Let's skip then by adding the -I option:

```grep -rnPI '\d{3}' . | awk -F ':' '{ system("sed -ne "$2"\47\173s/\\([[:digit:]]\\{3\\}\\)/\\1/p; \175 \47 " $1) }' > somefile```

Now we're gonna replace this result for empty value. We're gonna use the -i option in our sed, in order to make the changes infile (and not output then); also, we're using s////g.

```grep -rnP '\d{3}' . | awk -F ':' '{ system("sed -i "$2"'\''s/\[[:digit:]]\\{3\\}//g'\'' " $1) }'```


