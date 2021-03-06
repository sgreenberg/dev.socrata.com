---
layout: default
title: Accessing Endpoints in SODA 2.0
---

# {{ page.title }}

## Creating an Endpoint

An endpoint in SODA 2.0 is a URL that references an object.  In Socrata, every Dataset, View or Row has it's own endpoint without
the creater doing any additional work.  However, the Socrata platform also has several ways to customize your endpoints
that refer to your objects.

All resources are accessed through the `resource` path element.  In addition, all datasets and views have unique identifiers
that are used to identify them.  By default, Socrata will create an identifier sometimes referred to as a 4x4.  It is called
that because it normally has two groups of four characters.  For example,  `6cpn-3h7n` would be a valid 4x4.

The endpoint for a Dataset or View will be:

    https://[your domain]/resource/[unique identifier]

E.g.

    https://sandbox.demo.socrata.com/resource/6cpn-3h7n

Although, these endpoints now point to the right thing, you will notice that hitting them with a browser redirects you
to the HTML version of these endpoints.  This is because we didn't specify what format to return it as.  That will be
discussed below.

#### Naming an endpoint

Often times, a 4x4 is not a satisfactory name for your endpoint.  What you would prefer is something that has a name
more appropos to your object.  Socrata allows you do do this by setting the endpoint name on any dataset or view you own.
For exmple, the above dataset `6cpn-3h7n` is also named nominationsCopy.  This allows us to access it as:

    https://sandbox.demo.socrata.com/resource/nominationsCopy

#### Endpoint for a specific row

In addition to having an endpoint for a dataset, each row also has an endpoint based on it's unique identifier.  By default, a
row's unique identifier is provided automatically by Socrata.  It is called `:id` and is not returned unless asked for.  A dataset
can also specifically set a unique identifier in it's metadata, in which case that one is used.

    https://sandbox.demo.socrata.com/resource/nominationsCopy/1.json

NOTE:  If you want to actually see the `:id` values, you can put it in a select statement.  E.g. [https://sandbox.demo.socrata.com/resource/nominationsCopy.json?$select=:id,name,agency_name](https://sandbox.demo.socrata.com/resource/nominationsCopy.json?$select=:id,name,agency_name)

## Response Formats

SODA 2.0 deliberately separates the core API and data structures from the response types returned.  This allows it to be
very flexible in terms of providing multiple formats as well as adding new ones in the future.  Currently, it supports
JSON, CSV, XML and RDF.  In the future many other formats or more sophisticated mappings can be added without having to
affect the core capabilities of the system.

There are two ways to specify what format you want returned:
* file extension
* the `accept` HTTP header

The main reason for offering both mechanisms it make it easier to use tools such as the browser and curl for accessing the
API, while still providing the more "correct" `accept` header.  To use a browser to get the JSON version of a resource, you
 can now access:

    https://sandbox.demo.socrata.com/resource/nominationsCopy.json

The full list of currently supported formats are:

|Format|Extension|Accept Type|
|---|---|---|
|[JSON](/docs/formats/json)|`.json`|`application/json`|
|[CSV](/docs/formats/csv)|`.csv`|???|
|[XML](/docs/formats/xml)|`.xml`|`application/xml`|
|[RDF](/docs/formats/rdf)|`.rdf`|???|

## Authentication and Authorization

The authentication and authorization for SODA 2.0 is exactly the same as for the rest of the Socrata platform.  You can
 read more about it [here](/authentication)

## Errors

SODA 2.0 follows the HTTP and REST standards for returning error codes, and in addition has a standard structure to return
additional error responses.  The breakdown of error codes is:

|Code|Explaination|
|---|---|
|200 - 299|The request was successful (or at least did not fail yet).|
|300 - 399|The request requires the client to do additional work (like follow a redirect).|
|400 - 499|The request failed because there was something wrong with the request.  There is probably a bug in your code.|
|500 - 599|The request failed because there is a problem on the server.  There is probably a bug in Socrata's code.|

Each error will have a consistent error body returned with it.  This error body contains the following fields:

|Field|Explaination|
|---|---|
|code|Canonical error definition used by API implementations to distinguish errors at a finer level than HTTP codes.|
|message|An error message in the language specified by the request’s Accept-Language header, or in English if we don’t have a translation for that language.|
|data|Data specific to the context of this error.  This data may be a JSON structure instead of simply a string based on the errorCode field.  This may be useful for displaying to the user or for intelligent agent recovery from an error.|

An example of a JSON error response would be:

    {
    "code" : "query.execution.queryTooComplex",
    "error" : true,
    "message" : "Only simple comparison filters are allowed",
    "data" : {
        "reason" : "validation.complex-filter"
        }
    }

####Long Running Operations

There are some cases, where a particularly long running operation causes Socrata to move the operation to be asynchronous.  If
 this happens, the client will get back a 202 error code along with two headers to tell them how to get the correct results:

|Header|Explaination|
|---|---|
|Location|The URL to go to find the results (or get another 202).|
|Retry-After|How long the client should wait before retry-ing.|

