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

<!-- vim:set ft=markdown tw=80 fo+=an1 autoindent: -->
