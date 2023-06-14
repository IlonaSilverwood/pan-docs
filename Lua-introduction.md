Create custom outputs with pandoc's embedded Lua engine.

## Introduction

<!--- Intro to address "why should I care?" --->
With Lua filters, you can write Pandoc filters  without any external dependencies.
Besides the simpler set-up, Lua filters are generally faster and can access utility functions to manipulate document elements. 

<!---quick technical overview-->
Since Pandoc 2.0, the pandoc executable has a built-in Lua interpreter (version 5.4) and a Lua library for creating pandoc filters.
Pandoc data types are marshaled to Lua directly, avoiding the overhead of writing JSON to stdout and reading it from stdin.

Here is an example of a Lua filter that converts strong emphasis to small caps:

``` lua
return {
  {
    Strong = function (elem)
      return pandoc.SmallCaps(elem.content)
    end,
  }
}
```

or equivalently,

``` lua
function Strong(elem)
  return pandoc.SmallCaps(elem.content)
end
```

This says: walk the AST, and when you find a Strong element,
replace it with a SmallCaps element with the same content.

To run it, save it in a file, say `smallcaps.lua`, and invoke
pandoc with `--lua-filter=smallcaps.lua`.

## Why Lua filters over JSON?

[JSONfilters](https://pandoc.org/filters.html) accept a JSON
representation of the pandoc AST and produce an altered JSON
representation of the AST. They may be written in any
programming language, and invoked from pandoc using the
`--filter` option.

 However, JSON filters have limitations:
 
 - Writing JSON to stdout and reading it from stdin (twice, once on
each side of the filter) is inefficient.
- External dependencies vary between users, and universal JSON filters are not possible.

Here's a quick performance comparison, converting the pandoc
manual (MANUAL.txt) to HTML, with versions of the same JSON
filter written in compiled Haskell (`smallcaps`) and interpreted
Python (`smallcaps.py`): 

    Command                                 Time
    --------------------------------------- -------
    `pandoc`                                1.01s
    `pandoc --filter ./smallcaps`           1.36s
    `pandoc --filter ./smallcaps.py`        1.40s
    `pandoc --lua-filter ./smallcaps.lua`   1.03s

As you can see, the Lua filter avoids the substantial overhead
associated with marshaling to and from JSON over a pipe.
