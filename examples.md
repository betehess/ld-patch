## Initial document 
 
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

```
PATCH /timbl HTTP/1.1
Host: example.org
Content-Length: 326@@
Content-Type: text/ldpatch
If-Match: "abc123"

@prefix profile: <http://ogp.me/ns/profile#> .

Update <#me>/profile:first_name "Tim" "Timothy" .

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
