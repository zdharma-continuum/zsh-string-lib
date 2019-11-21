<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [zsh-string-lib](#zsh-string-lib)
  - [List Of The Functions](#list-of-the-functions)
    - [@str-parse-json](#str-parse-json)
    - [@str-read-all](#str-read-all)
    - [@str-ng-match](#str-ng-match)
    - [@str-ng-matches](#str-ng-matches)
    - [@str-read-ini](#str-read-ini)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# zsh-string-lib

A string library for Zsh. Its founding function was parsing of JSON.

## List Of The Functions

### @str-parse-json

Parses the buffer (`$1`) with JSON and returns:

1. Fields for the given key (`$2`) in the given hash (`$3`).
2. The hash looks like follows:

    ```
    1/1 → strings at the level 1 of the 1st object
    1/2 → strings at the level 1 of the 2nd object
    …
    2/1 → strings at 2nd level of the 1st object
    …
    ```

    The strings are parseable with `"${(@Q)${(@z)value}"`, i.e.:
    they're concatenated and quoted strings found in the JSON.

Example:

```json
{
    "zplugin-ices":{
        "default":{
            "wait":"1",
            "lucid":"",
            "as":"program",
            "pick":"fzy",
            "make":"",
        },
        "bgn":{
            "wait":"1",
            "lucid":"",
            "as":"null",
            "make":"",
            "sbin":"fzy;contrib/fzy-*"
        }
    }
}
```

Will result in:

```zsh
local -A Strings
Strings[1/1]='zplugin-ices'
Strings[2/1]='default bgn'
Strings[3/1]='wait 1 lucid \  as program pick fzy make \ '
Strings[3/2]='wait 1 lucid \  as null make \  sbin fzy\;contrib/fzy-\*'
```

So that when you e.g.: expect a key `bgn` but don't know at which
position, you can do:

```zsh
local -A Strings
@str-parse-json "$json" "zplugin-ices" Strings

integer pos
pos=${${(@Q)${(@z)Strings[2/1]}}[(I)bgn]}
if (( pos )) {
  local -A ices
  ices=( "${(@Q)${(@z)Strings[3/$pos]}}" )
  # Use the `ices' hash holding the values of the `bgn' object
  …
}
```

Arguments:

1. The buffer with JSON.
2. The key in the JSON that should be mapped to the result (i.e.: it's possible
   to map only a subset of the input).
3. The name of the output hash parameter.

### @str-read-all

Consumes whole data from given file descriptor and stores the string under the
given (`$2`) parameter, which is `REPLY` by default.

It can try hard to read the whole data by retrying multiple times (`10` by
default) and sleeping before each retry (not done by default).

Arguments:

1. File descriptor (a number; use `1` for stdin) to be read from.
2. Name of output variable (default: `REPLY`).
3. Numer of retries (default: `10`).
4. Sleep time after each retry (a float; default: `0`).

Example:

```zsh
exec {FD}< =( cat /etc/motd )
@str-read-all $FD
print -r -- $REPLY
…
```

### @str-ng-match

Returns a non-greedy match of the given pattern (`$2`) in the given string
(`$1`).

1. The string to match in.
2. The pattern to match in the string.

Example:

```zsh
if @str-ng-match "abb" "a*b"; then
  print $REPLY
fi
Output: ab
```

### @str-ng-matches


Returns all non-greedy matches of the given pattern ($2) in the given string
($1).

Input:

- `$1` … `$n-1` - the strings to match in
- `$n`         - the pattern to match in the strings

Return value:

- `$reply` – contains all the matches
- `$REPLY` - holds the first match
- return code: 0 if there was any match found, otherwise 1

Example:

```zsh
arr=( a1xx ayy a2xx )
if @str-ng-matches ${arr[@]} "a*x"; then
   print -rl $reply
fi

Outout:
a1x
a2x
```

### @str-read-ini

Reads an INI file.

Arguments:

1. Path to the ini file to parse.
2. Name of output hash (`INI` by default).
3. Prefix for keys in the hash (can be empty).

Writes to given hash under keys built in following way: `${3}<section>_field`.
Values are the values from the ini file.

<!-- vim:set ft=markdown tw=80 fo+=an1 autoindent: -->
