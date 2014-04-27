## Initial document 

The examples for the proposed syntax is given for the following document:

```
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
```

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
Add ?event rdf:type schema:Event .

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
    "op": { "@type": "@id" },
    "predicate": { "@type": "@id" },
    "var": { "@type": "@id" },
    "path": { "@type": "patch:path" },
    
    "profile": "http://ogp.me/ns/profile#",
    "schema": "http://schema.org/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  },
  "ops": [
    { "op": "patch:update", "startingNode": { "@id": "#me" }, "predicate": "profile:first_name", "matchingNode": "Tim", "value": "Timothy" },
    { "op": "patch:bind", "startingNode": { "@id": "#me" }, "path": "/schema:attendee", "var": "event" },
    { "op": "patch:add", "startingNode": { "var": "event" }, "predicate": "rdf:type", "value": { "@id": "schema:Event" } },
    { "op": "patch:bind", "startingNode": { "@id": "http://conferences.ted.com/TED2009/" }, "path": "\\schema:url!", "var": "ted" },
    { "op": "patch:delete", "startingNode": { "var": "ted" }, "value": { "@id": "schema:startDate" } },
    { "op": "patch:add", "startingNode": { "var": "ted" }, "predicate": "schema:location",
      "value": {
        "schema:name": "Long Beach, California",
        "schema:geo": { "schema:latitude": "33.7817", "schema:longitude": "-118.2054" }
      }
    }
  ]
}
```

## Turtle

```
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix patch: <http://www.w3.org/ns/ldpatch#> .

[]
    patch:ops ([
            patch:matchingNode "Tim" ;
            patch:op patch:update ;
            patch:predicate <http://ogp.me/ns/profile#first_name> ;
            patch:startingNode <http://example.org/timbl#> ;
            patch:value "Timothy"
        ]
        [
            patch:op patch:bind ;
            patch:path "/schema:attendee"^^patch:path ;
            patch:startingNode <http://example.org/timbl#> ;
            patch:var <http://example.org/event>
        ]
        [
            patch:op patch:add ;
            patch:predicate rdf:type ;
            patch:startingNode [
                patch:var <http://example.org/event>
            ] ;
            patch:value <http://schema.org/Event>
        ]
        [
            patch:op patch:bind ;
            patch:path "\\schema:url!"^^patch:path ;
            patch:startingNode <http://conferences.ted.com/TED2009/> ;
            patch:var <http://example.org/ted>
        ]
        [
            patch:op patch:delete ;
            patch:startingNode [
                patch:var <http://example.org/ted>
            ] ;
            patch:value <http://schema.org/startDate>
        ]
        [
            patch:op patch:add ;
            patch:predicate <http://schema.org/location> ;
            patch:startingNode [
                patch:var <http://example.org/ted>
            ] ;
            patch:value [
                <http://schema.org/geo> [
                    <http://schema.org/latitude> "33.7817" ;
                    <http://schema.org/longitude> "-118.2054"
                ] ;
                <http://schema.org/name> "Long Beach, California"
            ]
        ]
    ) .
```
