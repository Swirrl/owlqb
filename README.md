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

## Related Work

- [Managing Codelists and Classifications with OWL](https://docs.google.com/document/d/1n3liKicPcQwNgkok5T2WrRoFREPxBbek_t-j-TvM3E4/edit#)
   - [Worked example](https://gss-cogs.github.io/ref_migration/alignment/)
   - [Worked example code](https://github.com/GSS-Cogs/ref_migration)
- [3 sector model](https://github.com/Swirrl/cogs-issues/blob/master/modelling/3-sector-model.md)
  - [Various](https://github.com/Swirrl/cogs-issues/discussions/331) [problems](https://github.com/Swirrl/cogs-issues/discussions/332) [doing this](https://github.com/Swirrl/cogs-issues/discussions/275)
  - An existence proof for why modelling MECEness in OWL for statistical classifications might be hard/impossible; is that Ian Horrocks [avoided doing it here](https://www.cs.ox.ac.uk/people/ian.horrocks/Publications/download/2021/GermanoSHL21.pdf)
