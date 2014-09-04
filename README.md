OAI Provider for Fedora 4
=========================

Implements [Open Archives Protocol Version 2.0](http://www.openarchives.org/OAI/openarchivesprotocol.html) using Fedora 4 as the backend.

Installation
------------

1. Checkout the sources and build the fcrepo-oaiprovider jar:

```bash
#> git clone https://github.com/fasseg/fcrepo-oaiprovider.git
#> maven clean install
```

2. Drop the jar into Fedora's lib directory 

```bash
#> cp target/fcrepo-oaiprovider-{VERSION}.jar /path/to/webapp/WEB-INF/lib/
```

3. Edit Fedora's web.xml
  
```bash
#> vim /path/to/webapp/WEB-INF/web.xml
```

4. Add oaiprovider.xml context configuration. Simply adding oaiprovider.xml to Fedora's master.xml does not work it seems

```xml
<param-name>contextConfigLocation</param-name>
<param-value>WEB-INF/classes/spring/master.xml WEB-INF/classes/spring/oaiprovider.xml</param-value>
```

Links
-----

The provider depends on links to Datastreams and OAI Set objects to generate OAI responses
 
The graph should look like this:

                                                +----------+
                                                | MyObject | 
                                                +----------+
                                                /          \
                                               /            \
                                       hasOaiDCRecord   isPartOfOaiSet
                                             /                \
                                            /                  \
                               +-------------------+      +----------+
                               | MyOaiDCDatastream |      |   MySet  |
                               +-------------------+      +----------+


Additional Metadata record types
--------------------------------

The oaiprovider supports oai_dc out if the box, but users are able to add their own metadata format definitions to oaiprovider.xml.

Example
-------

1. Create a new Datastream containing an XML representation of an oai_dc record:

```bash
#> curl -X POST http://localhost:8080/fcrepo/rest -H "Slug:MyOAIDCDatastream" -H "Content-Type:application/octet-stream" --data @src/test/resources/test-data/oaidc.xml
```

2. Create a new OAI Set:

```bash
#> curl -X POST http://localhost:8080/fcrepo/rest/oai/sets -H "Content-Type:text/xml" --data @src/test/resources/test-data/set.xml
```

3. Create a new Fedora Object and link the oai_dc Datastream and the OAI Set created in the previous step to it:

```bash
#> curl -X POST http://localhost:8080/fcrepo/rest -H "Slug:MyObject" -H "Content-Type:application/sparql-update"  --data "INSERT {<> <http://fedora.info/definitions/v4/config#hasOaiDCRecord> <http://localhost:8080/fcrepo/rest/MyOAIDCDatastream> . <> <http://fedora.info/definitions/v4/config#isPartOfOAISet> \"MyOAISet\"} WHERE {}"
```

4. Try the various responses from the oai provider

```bash
#> curl http://localhost:8080/fcrepo/rest/oai?verb=ListMetadataFormats
#> curl "http://localhost:8080/fcrepo/rest/oai?verb=GetRecord&identifier=MyObject&metadataPrefix=oai_dc"
#> curl "http://localhost:8080/fcrepo/rest/oai?verb=ListRecords&metadataPrefix=oai_dc"
#> curl "http://localhost:8080/fcrepo/rest/oai?verb=ListIdentifiers&metadataPrefix=oai_dc"
#> curl "http://localhost:8080/fcrepo/rest/oai?verb=ListSets"
#> curl "http://localhost:8080/fcrepo/rest/oai?verb=ListIdentifiers&metadataPrefix=oai_dc&set=MyOAISet"

```



                               
                               
