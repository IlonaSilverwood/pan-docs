---
author:
- Albert Krewinkel
- John MacFarlane
date: 'January 10, 2020'
title: Pandoc Lua Filters
---
<!-- introductory guide, explain what filters are and how to run Lua filters -->
# Pandoc overview

<!-- assumes reader already knows what pandoc is/does -->
Pandoc consists of a set of readers and writers. When converting a
document from one format to another, text is first parsed by a
reader into pandoc’s intermediate representation of the document,
known as an "abstract syntax tree" or AST. The AST is then
converted by the writer into the target format. <!-- rewrite this paragraph, link to more information about AST -->

The pandoc AST format is defined in the module
[`Text.Pandoc.Definition` in the `pandoc-types` package
](https://hackage.haskell.org/package/pandoc-types/docs/Text-Pandoc-Definition.html).

# Filters explained

Filters are programs that modify the AST between the reader
and writer.

    INPUT --reader--> AST --filter--> AST --writer--> OUTPUT

Pandoc supports two kinds of filters:

- **Lua filters** use the Lua language to define transformations
  on the pandoc AST. 

- **JSON filters** are pipes that read from standard input and
  write to standard output, consuming and producing a JSON
  representation of the pandoc AST:

                             source format
                                  ↓
                               (pandoc)
                                  ↓
                          JSON-formatted AST
                                  ↓
                            (JSON filter)
                                  ↓
                          JSON-formatted AST
                                  ↓
                               (pandoc)
                                  ↓
                            target format

We recommend using Lua filters. The Lua interpreter is embedded in
pandoc, so no external software is required.  They are also
usually faster than JSON filters.  

Use a JSON filter if you wish to write your filter in any language
other than Lua. More information on JSON filters for pandoc can be
found [here](link to JSON section).

# Lua filters overview

Starting with pandoc version 2.0, a Lua interpreter (version 5.4)
and a Lua library for creating pandoc filters is built into the
pandoc executable. Pandoc data types are marshaled to Lua
directly, avoiding the overhead of writing JSON to stdout and
reading it from stdin. This means that no external dependencies
are needed when writing Lua filters. 

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
<!-- what is walking the AST?
"This tells the X to walk the AST..."? -->

Run the filter using the following steps:
1. save as a .lua file
   *  Example: `smallcaps.lua`
2. invoke pandoc with `--lua-filter=smallcaps.lua`

<!-- add more examples, link to tutorial? -->
 For more information on Lua filters, please see [in depth write up on Lua](Lua-advanced.md)

For a list of available Lua filters, see