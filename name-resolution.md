Name Resolution
---------------

Non-template functions:
-----------------

Degrees of freedom in name resolution:

* name --> name in definition
* argument types --> trivially convertible, or convertible via implicit conversion to parameter types

if more than one match is found, then the call is ambiguous -- an error.

Template functions:
-----------

Degrees of freedom in name resolution:
* name --> name in definition
* argument types --> matchable with template arguments

if more than one match is found, then all matches are ranked in terms of 'specific-ness', and if one match is more 'specific' than the others, then it is chosen.

Mixed template/non-template functions:
----------------

If a non-template match is found that requires no conversion of arguments, then choose that, even if a template version can be found that would match exactly.
If a non-template match requires some conversion to match, and a template version is found that can take the exact argument types, then choose that version.
Otherwise perform usual template function matching.

When you call a template function, there's a world of definitions that have been seen by the compiler at that point of 'instantiation'.


Quirk #1 of template functions
-----------

One quirk when working with template functions that does not affect non-template functions is that such function calls have the
potential to be seen by the compiler as not function calls at all!

This can happen when making a call with explicit template parameters:

```
make_thing<42>("hello");
```

The compiler has two ways of parsing the code above.

1) As a function call, with an explicit int template argument of 42, and a const char * argument of "hello"
2) As a strange attempt at a comparison between a variable 'make_thing', the number 42, and then the const char * "hello":
```
(make_thing < 42) > "hello"
```

We know that's highly unlikely, but it is possible. So, the rule is to treat it as 2), unless the compiler can prove to itself that
`make_thing` is definitely a function. To do that, it must find an instance of `make_thing` that *is* a template function, even if the 
rest of the parameter list does not match. Then it can safely assume you mean 1), and it can then proceed based on an
assumption that you mean to call a function template with an explicit template argument list.

In name resolution, just finding the name somewhere determines the rest of the treatment by the compiler.

If you think of the compiler as applying a set of 'passes' to find the matches for a function call, given the universe
of definitions it has already seen, then 

1. the first 'pass' eliminates everything that hasn't got the right name.
2. the second 'pass' eliminates everything that can't be seen in the current namespace
3. the third 'pass' *adds back* anything that can be seen in namespaces of argument types (Argument Dependent Lookup)

That's not the way a real compiler works, but it's *as if* it does. If at any stage above, no remaining matches can be found
then the compiler exits with an error.

Quirk #2 of Template Functions
-------

For the second pass above, template functions are resolved differently from non-template functions. At the point of
function call, the compiler will only consider functions that have been declared already in the universe of definitions,
even if just below the call, a better matching template is declared.
