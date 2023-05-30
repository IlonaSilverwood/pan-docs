# Pandoc Module

The `pandoc` Lua module is loaded into the filter's Lua
environment and provides a set of functions and constants to
make creation and manipulation of elements easier. The global
variable `pandoc` is bound to the module and should generally
not be overwritten for this reason.

Two major functionalities are provided by the module: element
creator functions and access to some of pandoc's main
functionalities.

## Element creation

Element creator functions like `Str`, `Para`, and `Pandoc` are
designed to allow easy creation of new elements that are simple
to use and can be read back from the Lua environment.
Internally, pandoc uses these functions to create the Lua
objects which are passed to element filter functions. This means
that elements created via this module will behave exactly as
those elements accessible through the filter function parameter.

## Exposed pandoc functionality

Some pandoc functions have been made available in Lua:

-   [`walk_block`](#pandoc.walk_block) and
    [`walk_inline`](#pandoc.walk_inline) allow filters to be applied
    inside specific block or inline elements;
-   [`read`](#pandoc.read) allows filters to parse strings into pandoc
    documents;
-   [`pipe`](#pandoc.pipe) runs an external command with input from and
    output to strings;
-   the [`pandoc.mediabag`](#module-pandoc.mediabag) module
    allows access to the "mediabag," which stores binary content
    such as images that may be included in the final document;
-   the [`pandoc.utils`](#module-pandoc.utils) module contains
    various utility functions.
