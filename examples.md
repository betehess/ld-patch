## Initial document 

The examples for the proposed syntax is given for the following document:

~~~
@prefix schema: <http://schema.org/> .
@prefix profile: <http://ogp.me/ns/profile#> .

<http://example.org/timbl#me> a schema:Person ;
  schema:alternateName "TimBL" ;
  profile:first_name "Tim" ;
  profile:last_name "Berners-Lee" ;
  schema:workLocation [ schema:name "W3C/MIT" ] ;
  schema:attendee _:b1, _:b2 .

_:b1  ;
  schema:name "F2F5 - Linked Data Platform" ;
  schema:url <https://www.w3.org/2012/ldp/wiki/F2F5> .

_:b2 a schema:Event ;
  schema:name "TED 2009" ;
  schema:startDate "2009-02-04" .
  schema:url <http://conferences.ted.com/TED2009/> .
~~~

## Custom syntax

This was our first attempt, which evolved from [Pierre-Antoine's solution](https://github.com/pchampin/rdfpatch).

```
PATCH /timbl HTTP/1.1
Host: example.org
Content-Length: 326@@
Content-Type: text/ldpatch
If-Match: "abc123"

@prefix profile: <http://ogp.me/ns/profile#> .

Update <#me> profile:first_name "Tim" "Timothy" .

Bind <#me>/schema:attendee ?event .
Add schema:Event -rdf:type ?event .

Bind <http://conferences.ted.com/TED2009/>\schema:url! ?ted .
Delete ?ted schema:startDate .
Add ?ted schema:location [
  schema:name "Long Beach, California";
  schema:geo [ schema:latitude "33.7817" ; schema:longitude "-118.2054" ] .
] .

Cut <#>/schema:workLocation .
```

## RDF-encoded LD Patch

Here LD Patch is defined in RDF. We show here the output as JSON-LD and Turtle (they would be completely equivalent).

### JSON-LD

```
{
  "@context": {
    "@base": "http://example.org/timbl",
    "@vocab": "http://www.w3.org/ns/ldpatch#",
    "patch": "http://www.w3.org/ns/ldpatch#",
    "ops": { "@container": "@list" },
    "predicate": { "@type": "@id" },
    "path": { "@type": "patch:Path" },
    
    "profile": "http://ogp.me/ns/profile#",
    "schema": "http://schema.org/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  },
  "@type": "patch:Patch",
  "ops": [
    { "@type": "patch:Update", "start": { "@id": "#me" }, "predicate": "profile:first_name", "existingValue": "Tim", "value": "Timothy" },
    { "@type": "patch:Bind", "start": { "@id": "#me" }, "path": "/schema:attendee", "value": { "@id": "_:event", "@type": "patch:Variable" } },
    { "@type": "patch:Add", "start": { "@id": "schema:Event" }, "inversePredicate": "rdf:type", "value": { "@id": "_:event" } },
    { "@type": "patch:Bind", "start": { "@id": "http://conferences.ted.com/TED2009/" }, "path": "\\schema:url!", "value": { "@id": "_:ted", "@type": "patch:Variable" } },
    { "@type": "patch:Delete", "start": { "@id": "_:ted" }, "value": { "@id": "schema:startDate" } },
    { "@type": "patch:Add", "start": { "@id": "_:ted" }, "predicate": "schema:location",
      "value": {
        "schema:name": "Long Beach, California",
        "schema:geo": { "schema:latitude": "33.7817", "schema:longitude": "-118.2054" }
      }
    },
    { "@type": "patch:Cut", "start": { "@id": "#me" }, "predicate": "schema:workLocation" }
  ]
}
```

## Turtle

```
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix patch: <http://www.w3.org/ns/ldpatch#> .

[]
    a patch:Patch ;
    patch:ops ([
            a patch:Update ;
            patch:existingValue "Tim" ;
            patch:predicate <http://ogp.me/ns/profile#first_name> ;
            patch:start <http://example.org/timbl#me> ;
            patch:value "Timothy"
        ]
        [
            a patch:Bind ;
            patch:path "/schema:attendee"^^patch:Path ;
            patch:start <http://example.org/timbl#me> ;
            patch:value [ _:b3 a patch:Variable ]
        ]
        [
            a patch:Add ;
            patch:inversePredicate "rdf:type" ;
            patch:start <http://schema.org/Event> ;
            patch:value _:b3
        ]
        [
            a patch:Bind ;
            patch:path "\\schema:url!"^^patch:Path ;
            patch:start <http://conferences.ted.com/TED2009/> ;
            patch:value [ _:b6 a patch:Variable ]
        ]
        [
            a patch:Delete ;
            patch:start _:b6 ;
            patch:value <http://schema.org/startDate>
        ]
        [
            a patch:Add ;
            patch:predicate <http://schema.org/location> ;
            patch:start _:b6 ;
            patch:value [
                <http://schema.org/geo> [
                    <http://schema.org/latitude> "33.7817" ;
                    <http://schema.org/longitude> "-118.2054"
                ] ;
                <http://schema.org/name> "Long Beach, California"
            ]
        ]
        [
            a patch:Cut ;
            patch:predicate <http://schema.org/workLocation> ;
            patch:start <http://example.org/timbl#me>
        ]
    ) .

```
