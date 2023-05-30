# Lua filter structure

Lua filters are tables with element names as keys and values
consisting of functions acting on those elements.

Filters are expected to be put into separate files and are
passed via the `--lua-filter` command-line argument. For
example, if a filter is defined in a file `current-date.lua`,
then it would be applied like this:

    pandoc --lua-filter=current-date.lua -f markdown MANUAL.txt

The `--lua-filter` option may be supplied multiple times. Pandoc
applies all filters (including JSON filters specified via
`--filter` and Lua filters specified via `--lua-filter`) in the
order they appear on the command line.

Pandoc expects each Lua file to return a list of filters. The
filters in that list are called sequentially, each on the result
of the previous filter. If there is no value returned by the
filter script, then pandoc will try to generate a single filter
by collecting all top-level functions whose names correspond to
those of pandoc elements (e.g., `Str`, `Para`, `Meta`, or
`Pandoc`). (That is why the two examples above are equivalent.)

For each filter, the document is traversed and each element
subjected to the filter. Elements for which the filter contains
an entry (i.e. a function of the same name) are passed to Lua
element filtering function. In other words, filter entries will
be called for each corresponding element in the document,
getting the respective element as input.

The return value of a filter function must be one of the
following:

-   nil: this means that the object should remain unchanged.
-   a pandoc object: this must be of the same type as the input
    and will replace the original object.
-   a list of pandoc objects: these will replace the original
    object; the list is merged with the neighbors of the
    original objects (spliced into the list the original object
    belongs to); returning an empty list deletes the object.

The function's output must result in an element of the same type
as the input. This means a filter function acting on an inline
element must return either nil, an inline, or a list of inlines,
and a function filtering a block element must return one of nil,
a block, or a list of block elements. Pandoc will throw an error
if this condition is violated.

If there is no function matching the element's node type, then
the filtering system will look for a more general fallback
function. Two fallback functions are supported, `Inline` and
`Block`. Each matches elements of the respective type.

Elements without matching functions are left untouched.

See [module documentation](#module-pandoc) for a list of pandoc
elements.

## Filters on element sequences

For some filtering tasks, it is necessary to know the order
in which elements occur in the document. It is not enough then to
inspect a single element at a time.

There are two special function names, which can be used to define
filters on lists of blocks or lists of inlines.

[`Inlines (inlines)`]{#inlines-filter}
:   If present in a filter, this function will be called on all
    lists of inline elements, like the content of a [Para]
    (paragraph) block, or the description of an [Image]. The
    `inlines` argument passed to the function will be a [List] of
    [Inline] elements for each call.

[`Blocks (blocks)`]{#blocks-filter}
:   If present in a filter, this function will be called on all
    lists of block elements, like the content of a [MetaBlocks]
    meta element block, on each item of a list, and the main
    content of the [Pandoc] document. The `blocks` argument
    passed to the function will be a [List] of [Block] elements
    for each call.

These filter functions are special in that the result must either
be nil, in which case the list is left unchanged, or must be a
list of the correct type, i.e., the same type as the input
argument. Single elements are **not** allowed as return values,
as a single element in this context usually hints at a bug.

See ["Remove spaces before normal citations"][Inlines filter
example] for an example.

This functionality has been added in pandoc 2.9.2.

[Inlines filter example]: #remove-spaces-before-citations

## Traversal order

The traversal order of filters can be selected by setting the key
`traverse` to either `'topdown'` or `'typewise'`; the default is
`'typewise'`.

Example:

``` lua
local filter = {
  traverse = 'topdown',
  -- ... filter functions ...
}
return {filter}
```

Support for this was added in pandoc 2.17; previous versions
ignore the `traverse` setting.

### Typewise traversal

Element filter functions within a filter set are called in a
fixed order, skipping any which are not present:

  1. functions for [*Inline* elements](#type-inline),
  2. the [`Inlines`](#inlines-filter) filter function,
  2. functions for [*Block* elements](#type-block) ,
  2. the [`Blocks`](#inlines-filter) filter function,
  3. the [`Meta`](#type-meta) filter function, and last
  4. the [`Pandoc`](#type-pandoc) filter function.

It is still possible to force a different order by explicitly
returning multiple filter sets. For example, if the filter for
*Meta* is to be run before that for *Str*, one can write

``` lua
-- ... filter definitions ...

return {
  { Meta = Meta },  -- (1)
  { Str = Str }     -- (2)
}
```

Filter sets are applied in the order in which they are returned.
All functions in set (1) are thus run before those in (2),
causing the filter function for *Meta* to be run before the
filtering of *Str* elements is started.

### Topdown traversal

It is sometimes more natural to traverse the document tree
depth-first from the root towards the leaves, and all in a single
run.

For example, a block list `[Plain [Str "a"], Para [Str
"b"]]`{.haskell} will try the following filter functions, in
order: `Blocks`, `Plain`, `Inlines`, `Str`, `Para`, `Inlines`,
`Str`.

Topdown traversals can be cut short by returning `false` as a
second value from the filter function. No child-element of
the returned element is processed in that case.

For example, to exclude the contents of a footnote from being
processed, one might write

``` lua
traverse = 'topdown'
function Note (n)
  return n, false
end
```

## Global variables

Pandoc passes additional data to Lua filters by setting global
variables.

`FORMAT`
:   The global `FORMAT` is set to the format of the pandoc
    writer being used (`html5`, `latex`, etc.), so the behavior
    of a filter can be made conditional on the eventual output
    format.

`PANDOC_READER_OPTIONS`
:   Table of the options which were provided to the parser.
    ([ReaderOptions](#type-readeroptions))

`PANDOC_WRITER_OPTIONS`
:   Table of the options that will be passed to the writer.
    While the object can be modified, the changes will **not**
    be picked up by pandoc.
    ([WriterOptions](#type-writeroptions))

    Accessing this variable in **custom writers** is
    **deprecated**. Starting with pandoc 3.0, it is set to a
    placeholder value (the default options) in custom writers.
    Access to the actual writer options is provided via the
    `Writer` or `ByteStringWriter` function, to which the options
    are passed as the second function argument.

    *Since: pandoc 2.17*

`PANDOC_VERSION`
:   Contains the pandoc version as a [Version] object which
    behaves like a numerically indexed table, most significant
    number first. E.g., for pandoc 2.7.3, the value of the
    variable is equivalent to a table `{2, 7, 3}`. Use
    `tostring(PANDOC_VERSION)` to produce a version string. This
    variable is also set in custom writers.

`PANDOC_API_VERSION`
:   Contains the version of the pandoc-types API against which
    pandoc was compiled. It is given as a numerically indexed
    table, most significant number first. E.g., if pandoc was
    compiled against pandoc-types 1.17.3, then the value of the
    variable will behave like the table `{1, 17, 3}`. Use
    `tostring(PANDOC_API_VERSION)` to produce a version string.
    This variable is also set in custom writers.

`PANDOC_SCRIPT_FILE`
:   The name used to involve the filter. This value can be used
    to find files relative to the script file. This variable is
    also set in custom writers.

`PANDOC_STATE`
:   The state shared by all readers and writers. It is used by
    pandoc to collect and pass information. The value of this
    variable is of type [CommonState] and
    is read-only.

`pandoc`
:   The *pandoc* module, described in the next section, is
    available through the global `pandoc`. The other modules
    described herein are loaded as subfields under their
    respective name.

`lpeg`
:   This variable holds the `lpeg` module, a package based on
    Parsing Expression Grammars (PEG).  It provides excellent
    parsing utilities and is documented on the official [LPeg
    homepage].  Pandoc uses a built-in version of the library,
    unless it has been configured by the package maintainer to
    rely on a system-wide installation.

    Note that the result of `require 'lpeg'` is not necessarily
    equal to this value; the `require` mechanism prefers the
    system's lpeg library over the built-in version.

`re`
:   Contains the LPeg.re module, which is built on top of LPeg
    and offers an implementation of a [regex engine].  Pandoc
    uses a built-in version of the library, unless it has been
    configured by the package maintainer to rely on a system-wide
    installation.

    Note that the result of `require 're` is not necessarily
    equal to this value; the `require` mechanism prefers the
    system's lpeg library over the built-in version.

[LPeg homepage]: http://www.inf.puc-rio.br/~roberto/lpeg/
[regex engine]: http://www.inf.puc-rio.br/~roberto/lpeg/re.html
