# Materializing Populations onto Observations

The obvious (and from a modelling perspective correct) place to
materialise the classes of statistical populations is onto the
observations themselves.

This can be achieved through owl property restrictions.

As a datacube uses skos:Concept's to describe codes, and concepts are
instances not classes, we first need a way to lift these individuals
into classes.  As we do so we can materialise those classes onto the
observation itself with property restriction like:

```turtle
:Male owl:equivalentClass [
        a owl:Restriction ;
        owl:onProperty :gender;
        owl:hasValue :male] .
```

This lifts the `:male` concept into a class `:Male`, that exists on
the domain of the `:gender` `qb:DimensionProperty`, i.e. here we
materialise the class `:Male` on obervations that have a `:gender`
dimension pointing to the `:male` concept.

In and of itself this doesn't achieve much, it simply allows a query
like:

```sparql
SELECT ?obs WHERE {
   ?obs :gender :male .
}
```

to be written as:

```sparql
SELECT ?obs WHERE {
   ?obs a :Male .
}
```

## PROs

However it does allow us to coin a class for skos concepts, and do so
on the observation which is the individual instance that describes the
statistical population and gives it a number.  So from a modelling
perspective it makes a lot of sense.

## Cons

The downside of this approach is that it requires a restriction to be
created for each distinct dimension value in the DSD.  Also
materialising classes onto observations will be both very noisy, and
also very expensive as Observations are the most numerous instance we
have.
