---
layout: post
title: "Frequently Used Awk Commands"
subtitle: "For fast text stream manipulations"
date: 2022-09-16 09:26:47
header-style: text
catalog: true
author: "Yuan"
tags: [awk, shell, scripts, text stream]
---
{% include linksref.html %}
> 运用之妙，存乎一心

# Introduction
Awk is a program that you can use to select particular records in a file and perform operations upon them. It is installed on Linux and Mac by default.

Awk is an interpreting program language with a simple programming paradigm: <b> find a pattern in the input and then perform an action</b>, which often reduce complex or tedious data manipulations to a few lines of code. Since it's easy and powerful, why not using it to avoid tedious openning/searching/matching/changings using one or few lines of awk codes?

This blog is for my own notes and records, mainly from [Dougspeed](https://dougspeed.com/awk/) and [The GNU Awk User’s Guide](https://www.gnu.org/software/gawk/manual/gawk.html#One_002dshot). The awk is far more powerful than the content/examples listed in this blog.

# Basic composition of an awk program: rule-action
When you run awk, you specify an awk program that tells awk what to do. The program consists of a series of rules (it may also contain user defined functions). Each rule specifies one pattern to search for and one action to perform upon finding the pattern. <b>By default, the action of awk will be the <code>{print}</code> (or equally <code>{print $0}</code>) if not specified</b>.\
Syntactically, a rule consists of <b>a pattern</b> followed by <b>an action</b>. The action is <b>enclosed in braces</b> to separate it from the pattern. <b>Newlines</b> usually separate rules. Therefore, an awk program looks like this:\
```bash
pattern1 { action1 }
pattern2 { action2 }
pattern3 { action3 }
pattern4 { action4 }
…
```

{{warning}}
Note that awk scanns your file <b>line by line</b>. For each line, it excute <i>pattern1{action1}</i> first, and then <i>pattern2{action2}</i> <b>separately</b> unless you <i>exit</i> in <i>action1</i>.<br/>
Same rule applys to pattern3, pattern4, etc.
{{end}}

# Running awk

## No input files
awk applies the program to the standard input, which usually means whatever you type on the keyboard. This continues until you indicate end-of-file by typing <i>Ctrl-d</i>. \
```bash
#No input files
awk 'program'
```

## Input with files
```bash
# Input with single file
awk 'program' input-file1
awk <input-file1 'program'
zcat xxx.gz | awk 'program'

# Input with multiple files
# The 'dash' argument indicate awk to read its standard input from stream '|'
awk 'program' input-file1 input-file2 …
zcat xxx.gz | awk 'program' input-file2 -

```

## Read 'program' from a script
```bash
#When the program is long,
awk -f program-file input-file1 input-file2 …
```

# System Variables and Functions
## System variables in awk
The awk program defines a number of special variables that can be referenced or reset inside a program.\

| Variable Name | Definition                                                        | Example                                                                                  | Meaning                                                                                                                            |
| ------------: | :---------------------------------------------------------------- | :--------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
|      FILENAME | Current filename                                                  | <code>awk 'NR==1{print FILENAME} input-file'</code>                                      | print filename                                                                                                                     |
|            FS | Field separator(a blank by default)                               | <code>awk 'BEGIN{FS=","}NR==1' xxx.csv</code>                                            | Set separator to ','                                                                                                               |
|            NR | Total number of lines/records processed                           | <code>awk '1;NR == 11{exit}' inputfile</code>                                            | Print the first 11 lines                                                                                                           |
|           FNR | the record number (typically the line number) in the current file | <code>awk '{if(NR==FNR){arr[$1];next}}($1 in arr){print $1}' file1 file2</code>          | first store Column 1 of the first file (in the variable arr), then test whether elements in Column 1 of the second file are in arr |
|            NF | Number of field in the current record                             | <code>awk NF</code>                                                                      | Delete all blank lines from a file: If NF==0, not print; Else print $0;                                                            |
|           OFS | Output file separator                                             | <code>awk ‘BEGIN{FS=",";OFS=";"}NR==4, NR==8 {print NR,$1,$2}’ sample_summary.csv</code> | print line#,\\$1,\\$2 from line4 to line8,separated by ";"                                                                         |
|            RS | record separator                                                  | <code>awk 'BEGIN{RS="\n"};1;NR==2{exit}' file1</code>                                    | set record separator to "\n"                                                                                                       |
|           ORS | output record separator                                           | <code>awk 'BEGIN{ORS="\n\n"};1;NR==4{exit}' file1</code>                                 | set output record separator to "\n\n"                                                                                              |

## System functions in awk
The awk program defines a number of special variables that can be referenced or reset inside a program.\

|                  Function Name | Definition                                                                                                                                                    | Example                                                                                                                                           | Meaning                                                                         |
| -----------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------ |
|                       int(num) | get int of number                                                                                                                                             | <code>awk 'BEGIN{print int(3.534);print int(4);print int(-5.223);print int(-5);}'</code>                                                          | print int(num)                                                                  |
|                       log(num) | get natural logarithmic(with base e) of given amount                                                                                                          | <code>awk 'BEGIN{print log(3.534);print log(4);print log(0);print log(-5);print log(-1);}'</code>                                                 | Returns -inf when given zero and gives nan error when negative number is given. |
|                         exp(x) | the exponential of x (e ^ x) or report an error if x is out of range                                                                                          | <code>awk 'BEGIN{print exp(2.1)}'</code>                                                                                                          | get e^2.1                                                                       |
|                      sqrt(num) | gives the positive root for the given number                                                                                                                  | <code>awk 'BEGIN{print sqrt(16);print sqrt(1.21);print sqrt(0);print sqrt(-12);}'</code>                                                          | returns nan error if we give negative number as argument.                       |
|                       sin(num) | gives sine value of num, with num in radians                                                                                                                  | <code>awk 'BEGIN{print sin(-60);print sin(90);print sin(45);}'</code>                                                                             | get sin(num)                                                                    |
|                       cos(num) | gives cosine value of n, with n in radians                                                                                                                    | <code>awk 'BEGIN{print cos(-60);print cos(90);print cos(45);}'</code>                                                                             | get cos(num)                                                                    |
|                 length(string) | alculates the length of a string                                                                                                                              | <code>awk 'BEGIN{print length("AAAA  BBB\tCC")}'</code>                                                                                           | Length of the string also includes spaces                                       |
|                substr(s, p, n) | Returns substring of string s at beginning position p up to a maximum length of n.                                                                            | <code>awk 'BEGIN{print substr("example aa bb cc", 4)}'</code>                                                                                     | If n is not supplied, the rest of the string from p is used.                    |
|              index(str1, str2) | searches the string str1 for the first occurrences of the string str2, and returns the position in characters where that occurrence begins in the string str1 | <code>awk 'BEGIN{print index("Graphic", "ph"); print index("University", "abc")}'</code>                                                          | String indices in awk starts from 1.                                            |
|                     tolower(s) | all uppercase characters in string s to lowercase                                                                                                             | <code>awk 'BEGIN{print tolower("GEEKSFORGEEKS")}'</code>                                                                                          |                                                                                 |
|                     toupper(s) | all lowercase characters in string s to uppercase                                                                                                             | <code>awk 'BEGIN{print toupper("geeksforgeeks")}'</code>                                                                                          |                                                                                 |
| split(string, array, fieldsep) | divides string into pieces separated by fieldsep, and stores the pieces in array, return length of array                                                      | <code>awk 'BEGIN{string="Hi world Hi AZ"; fieldsep=" "; n=split(string, array, fieldsep); for(i=1; i<=n; i++){printf("%s\n", array[i]);}}'</code> | Split string, store in array, get n, and print                                  |

# Regular Expressions in awk
AWK is very powerful and efficient in handling regular expressions.
```bash

echo -e "cat\nbat\nfun\nfin\nfan" | awk '/f.n/'

echo -e "This\nThat\nThere\nTheir\nthese" | awk '/^The/'

echo -e "knife\nknow\nfun\nfin\nfan\nnine" | awk '/n$/'

echo -e "Call\nTall\nBall" | awk '/[CT]all/'

echo -e "Call\nTall\nBall" | awk '/[^CT]all/'

echo -e "Call\nTall\nBall\nSmall\nShall" | awk '/Call|Ball/'

echo -e "Colour\nColor" | awk '/Colou?r/'

echo -e "ca\ncat\ncatt" | awk '/cat*/'

echo -e "111\n22\n123\n234\n456\n222"  | awk '/2+/'
#Grouping
echo -e "Apple Juice\nApple Pie\nApple Tart\nApple Cake" | awk '/Apple (Juice|Cake)/'

#substitute
echo "apple apple\npineapple apple\n" | awk 'sub(/apple/, "nut")'
echo "apple apple\npineapple apple\n" | awk 'gsub(/apple/, "nut")'

# Regx with variables
## awk can match against a variable if you don't use the // regex markers
## need to build up the required regex as a string
echo "apple apple\npineapple apple\n" | awk 'BEGIN{r = "eapp"} $0~r'
# Using variables from bash terminal
rex=eapp
echo "apple apple\npineapple apple\n" | awk -v r="$rex" '$0~r'
```

# Examples by Practice
## Find the intersect (overlap) of two files
```bash
awk '{if(NR==FNR){arr[$1];next}}($1 in arr){print $1}' file1.txt file2.txt
# Or 
awk '(NR==FNR){arr[$1];next}($1 in arr){print $1}' file1.txt file2.txt
```
## Get details of the overlaps
```bash
awk '(NR==FNR){arr[$1]=$2;next}($1 in arr){print "SNP:",$1, "P-Value1:",arr[$1], "P-Value2:", $2}' file1.txt file2.txt
```


## Remove duplicates
Remove duplicates in file1.$1, keep only the first record.<br/>
Step by step intepretations:<br/>
1. check \\$1 in seen, if not: seen[\\$1]=0;
2. check !seen[\\$1], if TRUE: {print \\$0};
3. seen[\\$1]++;

```bash
awk '(!seen[$1]++)' file1.txt >file1_rmDuplicate.txt
```

## An example of awk script
From [Steve](https://stackoverflow.com/questions/11534173/how-to-use-awk-variables-in-regular-expressions)\
```awk
BEGIN {
    FS = "[. ]"
    OFS = "."
}

FNR == NR {
    domain[$1] = $0
    next
}

FNR < NR {
    if ($2 in domain) {
        for ( i = 2; i < NF; i++ ) {
            if ($i != "") {
                line = (line ? line OFS : "") $i
            }
        }
        total[line] += $NF
        line = ""
    }
}

END {
    for (i in total) {
        printf "%s\t%s\n", i, total[i]
    }
}
```


---
