# Materialising classes on codes

This document corresponds to the example in sub-prop.  Which
demonstrates how we can coin classes on the codes themselves, add
further meaning by subPropertying, dimension values, and also model
MECEness and other things in OWL.

The file `sub-prop/skos-bindings.ttl` shows how we can trivially lift
skos concepts into classes, by creating a direct mapping into
owl:classes with statements like:

`skvoc:male a :Male .`

## MECEness

The concept of MECE (Mutually Exclusive and Collectively Exhaustive)
for us means that within a dimension, each concept (dimension value)
is non-overlapping, i.e. it does not include members of another set,
and that the set of dimension values covers every possible value for
that dimension.

We believe MECE may be important to model in our datasets and concepts
as it will allow us to know whether numbers within the same dataset
are aggregatable.  Currently we can't assume they are, because of the
presence of intermediate "totalling" observations, or potentially
overlapping concepts.  It's worth stating that not all dimension
values can be MECE as it's an overly restrictive for some dimensions,
and [datasets](https://statistics.gov.scot/resource?uri=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Fneighbourhood-safety-when-walking-alone---scottish-household-survey) such as questionnaire results that include a "Don't know"
option.

Lets first take a look at a simplified `:gender` dimension:

```turtle
:Gender rdfs:subClassOf :StatisticalPopulation .

:AnyMECEGender owl:disjointUnionOf (:Male :Female) ;
               rdfs:subClassOf :Gender .

:GenderAll rdfs:subClassOf :Gender, :Aggregate .

:gender rdfs:subPropertyOf qb:dimension .
```

`:StatisticalPopulation` is proposed as the root class for all
concepts we use in cubes that describe a population of people.
`:Gender`, like all other dimension values in this example is a
subclass of this class.

We next express the concept of `:AnyMECEGender` as
`owl:disjointUnionOf (:Male :Female)`.  This OWL declaration states
that the classes in the rdf List are pairwise disjoint from each other
(i.e. they don't overlap in anyway, and there is no person in the
female population that is also in the male population), and that
`:Male` and `:Female` enterily cover the class `:AnyMECEGender`.

`:Male` and `:Female` effectively then become an an `rdfs:subClassOf`
of `:AnyMECEGender`.  This then enables aggregating queries such as
this:

```sparql
prefix : <http://example.org/v/>

select (SUM(?count)AS ?sum) WHERE {
    ?obs a :Observation ;
         :gender ?gender ;
         :refArea ?area ;
         :count ?count .

    ?gender a :AnyMECEGender .
    ?area a :Liverpool .
}
```

You'll note also that I define the class `:GenderAll` as another
class.  This is a bit of a hack, and I suspect it can be better
modelled, however it's an attempt to implicitly try and model the
distinction between an aggregated "All" dimension value, that
identifies a total over gender, and the class `:AnyMECEGender` which
we use to find those observations.  The reason we need something like
this is because `:Male a :AnyMECEGender` and `:Female a
:AnyMECEGender` and `:AnyMECEGender a :AnyMECEGender .` (i.e. all
classes are subclasses of themselves.).  We may not need to explicitly
model this as we can use special properties such as
`sp:directSubClassOf`.  However I got around this here by modelling
the class `:GenderAll` as being outside of the MECE set, but still a
`:Gender`.

## Gotchas

There are some unexpected gotchas with OWL I ran into:

1. Here we express OWL in RDF triples which we load into stardog.
   However not all the triples you use to define owl axioms are
   themselves allowed in the graph of triples OWL will reason about.
   Presumably this is to avoid stuff like the Russel Paradox, and
   halting problem.  However the upshot of this is that when reasoning
   is on you can no longer see triples like this:

   `:AnyMECEGender owl:disjointUnionOf (:Male :Female) ;`

   Which means that you can't write meta-queries to find MECE children
   like this with reasoning on:

```sparql
   select ?class where {

    :Ward1 owl:disjointUnionOf ?disj .

    ?disj rdf:rest*/rdf:first ?class  .
   }
```

   Instead you need to write something more baroque like:

```sparql
prefix : <http://example.org/v/>
prefix skvoc: <http://example.org/skos-vocab-term/>
prefix sp: <tag:stardog:api:property:>

# finding mece codes for a dimension class
select distinct ?mece WHERE {

    BIND(:Gender AS ?parent_code)

    ?dc sp:directSubClassOf ?parent_code .

    ?mece sp:directSubClassOf ?dc .
    ?mece_2 sp:directSubClassOf ?dc .

    ?mece owl:disjointWith ?mece_2 .

    # Thanks OWL for reminding me that the class
    # of "no things" is disjoint with the class
    # of things I want.
    FILTER(?mece != owl:Nothing)
}
   ```

   This query will return the distinct set of MECE `:Gender`s
   i.e. `:Male` `:Female`.  It is proposed meta queries such as these
   could be used to display checkboxes on the interface for Total that
   might allow aggregating cubes across many dimensions, even where
   aggregations don't exist.  Or this information may be used to
   materialise these observations in the first place.

   Similarly marker classes may be defined like so to explicitly
   represent the meta-model of MECEness in the model:

```turtle
:MECE owl:intersectionOf (:ME :CE) .

:AnyMECEGender owl:disjointUnionOf (:Male :Female) ;
               rdfs:subClassOf :MECE ; # marker class to identify mece superclasses
               rdfs:subClassOf :Gender .

   ```

this would allow queries like this to aggregate over a dimension:

   ```sparql
   prefix : <http://example.org/v/>
   prefix skvoc: <http://example.org/skos-vocab-term/>
   prefix sp: <tag:stardog:api:property:>

   # finding mece codes for a dimension class
   select *  WHERE {
     BIND(:Gender AS ?dim)
       ?obs :gender ?gen .
       ?gen a :MECE .
   }
   ```

## Ontologies as DSDs


Having engaged in this exercise I have come from a purely theoretical
standpoint to share a view of Alex's that datasets may want to coin
codes as they use them, and then integrate them with them OWL later.

A logical consequence and extension of this idea is that the real
meaning of a code is only understood in the context of its dataset.
For example on Scotland we see the use of the code `Other` to describe
ethnicity.  Unlike concrete concepts of ethnicity like `Asian`,
`Black` or `White` the code `Other` often seems to mean "other
ethnicities not included" in this dataset; which is often a different
set of ethnicities than `Other` in a different datasets context.

For example in this
[dataset](https://statistics.gov.scot/resource?uri=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Fneighbourhood-safety-when-walking-alone---scottish-household-survey)
Other means "Not white".  There's another interesting point
illustrated by this dataset whichs is that the ethnicities `White` and
`Other` (not white) are MECE within this dataset.  Though `Other` (not
white) if used in a different dataset may not be MECE, depending on
what other codes it is mixed with.

Lets assume this dataset coins codes for the ethnicity dimension:

```turtle
prefix scot: <http://stats.scot/concepts/>
prefix waeth: <http://stats.scot/data/walking-alone/ethnicity/> .

waeth:Other a scot:Ethnicity .

```

Then later we discover "Other" means not white...

```turtle
waeth:Other owl:disjointWith scot:White ;
            owl:unionOf (scot:Black, scot:Asian, scot:Chinese scot:Other) .
```

Then a notional observation finder can join in a way it previously
couldn't on the dimension value of Other.
