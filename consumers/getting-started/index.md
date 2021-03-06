---
layout: default
title: Getting started with the SODA Consumer API
---

<div id="tryit-visor">
  <div id="tryit-target"><!-- This space intentionally left blank --></div>
</div>

<img src="/img/soda-small.png" class="right" alt="SODA Consumer API" />

# Getting Started with the SODA Consumer API

In addition to our [code samples and SDKs](/libraries/), we also provide simple runnable examples inline with the documentation. In order to make the examples as language-agnostic as possible, and to show how you can use the SODA API with the simplest tools possible, examples are provided in-line live sample queries. Wherever you see a link like <http://soda.demo.socrata.com/resource/earthquakes.json?$limit=5>, you can click it to dynamically run that request in the example "visor". You can examine the URL that was constructed, see what results were returned, and experiment, all without ever leaving the page you're currently on.

A few examples:

* Retrieve the last five earthquakes: <http://soda.demo.socrata.com/resource/earthquakes.json?$limit=5&$order=datetime DESC>
* Retrieve only quakes with a `source` of "uw": <http://soda.demo.socrata.com/resource/earthquakes.json?source=uw>
* Retrieve only quakes with a `magnitude` less than 5: <http://soda.demo.socrata.com/resource/earthquakes.json?$where=magnitude < 5>

## SODA Datasets and Custom APIs

On the Socrata platform, every `dataset` is automatically provided with a simple API. That ensures that every developer has at least programmatic access to every dataset on our platform, regardless of the actions of the dataset owner.

Some data publishers may opt to use [Socrata API Foundry](http://www.socrata.com/api-foundry) product to create `custom APIs` for their datasets. When creating a custom API, data publishers can customize the schema, functionality, and endpoint of their API to their needs. Custom APIs also are provided automatically generated, customizable documentation pages and entry in that site's API catalog, like [this example for the `earthquakes` API](https://soda.demo.socrata.com/developers/docs/earthquakes).

## SODA Endpoints

In SODA 2.0, the endpoint for every `dataset` and `custom API` is by default hosted under `/resources`. Datasets are addressable by that dataset's unique identifier, which you can locate either in the "API" sidebar on the dataset or as the 4x4 code at the end of any dataset URL. Custom APIs have a custom address under `/resources` that you can discover in that data site's `/developer` portal.

For example, the [USGS Earthquake Reports](https://soda.demo.socrata.com/dataset/USGS-Earthquake-Reports/4tka-6guv)
set on [soda.demo.socrata.com](https://soda.demo.socrata.com) has a unique identifier of `4tka-6guv`, so its API endpoint is `https://soda.demo.socrata.com/resource/4tka-6guv.json`. However, a `custom API` has also been created for it and, as stated in its [API documentation](https://soda.demo.socrata.com/developers/docs/earthquakes), its custom endpoint is at `https://soda.demo.socrata.com/resource/earthquakes.json`.

<div class="more-info">
  Details on SODA endpoints can be found their <a href="/docs/endpoints.html">detailed documentation</a>.
</div>

## API Schemas

The schema of the output of SODA APIs is determined either by the structure of the dataset itself, or by the configuration that the data owner applied using the [API Foundry](http://www.socrata.com/api-foundry/). In either case, each column in the dataset is represented as a `field` within the output that is keyed by its `field name`. Field names can be distinct from the human-readable name for a column, but are designed to be best used with the API.

The actual format differs depending on the output type you have selected (see the Available Data Formats section below), but take, for example, the JSON output for the [USGS Earthquakes API](https://soda.demo.socrata.com/developers/docs/earthquakes) we've been working with:

<pre class="prettyprint">
[
    {
        "region": "British Columbia, Canada",
        "source": "uw",
        "location": {
            "longitude": -120.5032,
            "latitude": 49.4012
        },
        "magnitude": 2.3,
        "number_of_stations": 11,
        "datetime": "2012-09-13T23:06:29",
        "earthquake_id": "60451507",
        "depth": 0.00,
        "version": "2"
    },
]
</pre>

Each column is represented by a unique field name. Conveniently, the schema details for the API are included in the HTTP response headers with every request as `X-SODA2-Fields` and `X-SODA2-Types`:

<pre class="prettyprint">
X-SODA2-Fields: ["region",":updated_at","source","location","magnitude ","number_of_stations",":id","datetime",":created_at","earthquake_id","d epth","version"]
X-SODA2-Types: ["text","meta_data","text","location","number","number" ,"meta_data","calendar_date","meta_data","text","number","text"]
</pre>

<div class="more-info">
  Details on SODA datatypes can be found their <a href="/docs/datatypes.html">detailed documentation</a>.
</div>

## Querying with SoQL

If you perform a `GET` request on an API endpoint using an HTTP client (such as a web browser, a simple utility like [cURL](http://curl.haxx.se/), or a REST client like [Postman](https://chrome.google.com/webstore/detail/fdmmgilgnpjigdojojpjoooidkmcomcm)), you will automatically get the first 1000, unfiltered records.

To perform simple equality queries, simply use the field name as a GET parameter with the your chosen filter as the value. For example, to query the [USGS Earthquakes API](https://soda.demo.socrata.com/developers/docs/earthquakes) for quakes with a `region` of `Washington`:

[https://soda.demo.socrata.com/resource/earthquakes.json?region=Washington](https://soda.demo.socrata.com/resource/earthquakes.json?region=Washington)

In addition to this base functionality, SODA adds a rich query language to let you build  expressive queries.
Here is set of progressively more complex queries that could be added to the endpoint for more interesting results:

* Quakes with a `magnitude` of greater than 5: [http://soda.demo.socrata.com/resource/earthquakes.json?$where=magnitude > 5](http://soda.demo.socrata.com/resource/earthquakes.json?$where=magnitude%3E5)
* Return only the `magnitude`, `location`, and `datetime` of the above query: [http://soda.demo.socrata.com/resource/earthquakes.json?$where=magnitude > 5&$select=magnitude,location,datetime](http://soda.demo.socrata.com/resource/earthquakes.json?$where=magnitude+>+5&$select=magnitude,location,datetime)
* Group by `region` and return the count of earthquakes with a `magnitude` of greater than 5: [http://soda.demo.socrata.com/resource/earthquakes.json?$select=region,count(earthquake_id)&$where=magnitude > 5&$group=region](http://soda.demo.socrata.com/resource/earthquakes.json?$select=region,count%28earthquake_id%29&$where=magnitude+>+5&$group=region)

For those looking for a little more in the way of geo-queries, we offer those as well. You can perform range queries using the `within_circle` function, and you can perform bounding-box queries using `within_box`:

* Quakes within 5 kilometers of the Socrata offices in Pioneer Square, Seattle, WA: [http://soda.demo.socrata.com/resource/earthquakes.json?$where=within_circle(location, 47.598178, -122.334526, 5000)](http://soda.demo.socrata.com/resource/earthquakes.json?$where=within_circle%28location,+47.598178,+-122.334526,+5000%29)
* Quakes within the City of Seattle, WA: [http://soda.demo.socrata.com/resource/earthquakes.json?$where=within_box(location, 47.712585, -122.464676, 47.510759, -122.249756)](http://soda.demo.socrata.com/resource/earthquakes.json?$where=within_box%28location,+47.712585,+-122.464676,+47.510759,+-122.249756%29)

<div class="more-info">
  Details on SODA queries can be found in their <a href="/docs/queries.html">detailed documentation</a>.
</div>

## Available Data Formats

The Socrata Open Data API supports a number of different formats, including [JSON](/docs/formats/json.html), [XML](/docs/formats/xml.html), [CSV](/docs/formats/csv.html), and [RDF](/docs/formats/rdf.html). To change the output format of a request, simply change the extension you use on the resource you are trying to receive.

Most of the API examples use the JSON format, as it is the most compact and efficient format we provide.

## Authentication, Application Tokens, and Throttling

If you only wish to consume data from the Socrata Open Data API, you don't need to worry about authentication. However, all requests are subject to throttling to protect one application, malicious or otherwise, from affecting the availability of the API for everyone else using it. If you're going to be making a large number of requests, [register your application](http://opendata.socrata.com/profile/app_tokens) for an application token. You'll receive higher request quotas, and you'll also receive free publicity for your app. What's not to love?

If you wish to act on behalf of a Socrata user, however, you will need to use one of the SODA API's two authentication methods. The API is built to authenticate primarily with [OAuth 2](http://oauth.net/2/). We support only the server-based flow with a callback URL for security reasons. Please see the [API Authentication and Application Tokens](/authentication/) section for further details on authenticating using OAuth 2.

If you're only writing a quick application to authenticate as yourself, or you are unable for some other reason to use OAuth 2, the SODA API provides a fall-back authentication method using [HTTP Basic](http://en.wikipedia.org/wiki/Basic_access_authentication) authentication. Simply use the email address and password you registered with Socrata. When making authenticated requests, please remember to *always* use HTTPS connections, and to supply an application token.

<div class="more-info">
  For more information, see the <a href="/authentication">API Authentication and Application Tokens</a> section.
</div>

{% include try.html %}
{% include prettifier.html %}
