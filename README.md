# OWL QB

This document/repo is an exploration and various thoughts on how OWL
might help describe data cubes and their dimension/dimension-values.

Due to the minor problems with the actual QB ontology I use a
simplified version for illustration - and assume the [existance of
workarounds](https://github.com/GSS-Cogs/cube-ontology/commit/b21b6af8ae2b1135efcaf62a778de61a0448575d)
for the various uninteresting issues, and instead try to focus on the
details.

## Alex's insight

An owl class can be used to describe a statistical population.
i.e. for us the class `Female` is not actually describing gender or
sex, but a statistical population.

Taking this as a starting point we can explore what this means in the
context of a data cube and how it might help us model things that
support new features.

### Converting owl individuals to classes

As a datacube uses skos:Concept's to describe codes, and concepts are
instances not classes, we first need a way to lift these individuals
into classes.

We can do this with property restrictions like so:

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

However it does allow us to coin a class for skos concepts.  There are
likely many other ways we might do this.  The hope is however that
once you have a class you can do interesting things that open up OWL
tooling.  All observations returned from the above query are then
observations about the :Male population.

### Trying to represent MECEness

The concept of MECE (Mutually Exclusive and Collectively Exhaustive)
for us means that within a dimension, each concept (dimension value)
is non-overlapping, i.e. it does not include members of another set,
and that the set of dimension values covers every possible value for
that dimension.

We believe MECE may be important to model in our datasets and concepts
as it will allow us to know whether numbers within the same dataset
are aggregatable.  Currently we can't assume they are, because of the
presence of intermediate "totalling" observations, or potentially
overlapping concepts.  For example a dataset may contain an
observation that sums all male and female values:

```
:obs-2 a :Observation ;
    :refArea :liverpool ;
    :count 20 ;
    :gender :gender-all .
```

If we wanted to check this count, or perhaps derive this observation
from more granular observations from wards beneath liverpool, we would
need to know that the dimension values for wards within liverpool are
non-overlapping (to ensure we don't double count).

An attempt to represent this is made with `owl:disjointUnionOf`:

```turtle
:AnyGender owl:disjointUnionOf (:Male :Female) ;
           rdfs:subClassOf :Gender .

:GenderAll owl:disjointWith :AnyGender ;
           rdfs:subClassOf :Gender, :Aggregate .

:Liverpool owl:disjointUnionOf (:Ward1 :Ward2) .

:Ward1 owl:disjointUnionOf (:LSOA1A :LSOA1B ) .
:Ward2 owl:disjointUnionOf (:LSOA2A :LSOA2B ) .
```

This then allows things like:


```sparql
prefix : <http://example.org/v/>

select (SUM(?count)AS ?sum) WHERE {
    ?obs a :Liverpool ; #, :Ward1, :Female ;
        :count ?count .

}
```

the council area of `:liverpool` where we have
observations for arbitrary other




Converting this into OWL involves a shift in terminology, to that of
sets/classes.  OWL provides a primitive that means
