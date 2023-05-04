# Lua filters structure

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

