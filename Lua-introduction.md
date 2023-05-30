# Introduction

Pandoc has long supported filters, which allow the pandoc
abstract syntax tree (AST) to be manipulated between the parsing
and the writing phase. [Traditional pandoc
filters](https://pandoc.org/filters.html) accept a JSON
representation of the pandoc AST and produce an altered JSON
representation of the AST. They may be written in any
programming language, and invoked from pandoc using the
`--filter` option.

Although traditional filters are very flexible, they have a
couple of disadvantages. First, there is some overhead in
writing JSON to stdout and reading it from stdin (twice, once on
each side of the filter). Second, whether a filter will work
will depend on details of the user's environment. A filter may
require an interpreter for a certain programming language to be
available, as well as a library for manipulating the pandoc AST
in JSON form. One cannot simply provide a filter that can be
used by anyone who has a certain version of the pandoc
executable.

Starting with version 2.0, pandoc makes it possible to write
filters in Lua without any external dependencies at all. A Lua
interpreter (version 5.4) and a Lua library for creating pandoc
filters is built into the pandoc executable. Pandoc data types
are marshaled to Lua directly, avoiding the overhead of writing
JSON to stdout and reading it from stdin.

Here is an example of a Lua filter that converts strong emphasis
to small caps:

``` lua
return {
  {
    Strong = function (elem)
      return pandoc.SmallCaps(elem.c)
    end,
  }
}
```

or equivalently,

``` lua
function Strong(elem)
  return pandoc.SmallCaps(elem.c)
end
```

This says: walk the AST, and when you find a Strong element,
replace it with a SmallCaps element with the same content.

To run it, save it in a file, say `smallcaps.lua`, and invoke
pandoc with `--lua-filter=smallcaps.lua`.

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
