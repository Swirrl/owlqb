# OWL QB

This document/repo is an exploration and various thoughts on how OWL
might help describe data cubes and their dimension/dimension-values.

Due to the minor problems with the actual QB ontology I use a
simplified version for illustration - and assume the [existance of
workarounds](https://github.com/GSS-Cogs/cube-ontology/commit/b21b6af8ae2b1135efcaf62a778de61a0448575d)
for the various uninteresting issues, and instead try to focus on the
details.

There are two examples which both include OWL that works enough to
demonstrate the possibilities for modelling.

The most comprehensive example (and in my mind the better one) is
`sub-prop` and is desribed in the document on [materialising classes
on codes](/docs/sub-prop.md).

[on-obs](/docs/on-obs.md) is likely I think an evolutionary dead end,
but shows how we could materialise the classes onto observation
objects themselves.


## Running the database and OWL

1. Put the `bin` directory on your path and stardogs `bin` directory
   on your PATH.
2. Run from the root of this directory `create-db sub-prop &&
   create-db on-obs`.  This will build the stardog databases with OWL
   DL support.
3. You should now be able to run the queries and reason on the examples.

If you want to experiment with the data, you can run the command
`refresh-db sub-prop` which will drop and reload the data for you.

I used this crude workflow to develop and iterate the examples.
