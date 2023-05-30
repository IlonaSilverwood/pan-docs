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

## Lua filters overview

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

## Using Lua filters

Lua filters are tables with element names as keys and values
consisting of functions acting on those elements.

Filters are expected to be put into separate files and are passed
via the `--lua-filter` command-line argument. For example, if a
filter is defined in a file `current-date.lua`, then it would be
applied like this:

    pandoc --lua-filter=current-date.lua -f markdown MANUAL.txt

The `--lua-filter` option may be supplied multiple times. Pandoc
applies all filters (including JSON filters specified via
`--filter` and Lua filters specified via `--lua-filter`) in the
order they appear on the command line.

Pandoc expects each Lua file to return a list of filters. The
filters in that list are called sequentially, each on the result
of the previous filter. If there is no value returned by the
filter script, then pandoc will try to generate a single filter by
collecting all top-level functions whose names correspond to those
of pandoc elements (e.g., `Str`, `Para`, `Meta`, or `Pandoc`).
(That is why the two examples above are equivalent.)

For each filter, the document is traversed and each element
subjected to the filter. Elements for which the filter contains an
entry (i.e. a function of the same name) are passed to Lua element
filtering function. In other words, filter entries will be called
for each corresponding element in the document, getting the
respective element as input.

The return value of a filter function must be one of the
following:

-   nil: this means that the object should remain unchanged.
-   a pandoc object: this must be of the same type as the input
    and will replace the original object.
-   a list of pandoc objects: these will replace the original
    object; the list is merged with the neighbors of the original
    objects (spliced into the list the original object belongs
    to); returning an empty list deletes the object.

The function's output must result in an element of the same type
as the input. This means a filter function acting on an inline
element must return either nil, an inline, or a list of inlines,
and a function filtering a block element must return one of nil, a
block, or a list of block elements. Pandoc will throw an error if
this condition is violated.

If there is no function matching the element's node type, then the
filtering system will look for a more general fallback function.
Two fallback functions are supported, `Inline` and `Block`. Each
matches elements of the respective type.

Elements without matching functions are left untouched.

See [module documentation](#module-pandoc) for a list of pandoc
elements.

