---
layout: post
title:  "Docker Elasticsearch v2 Geolocation Sort and Filter"
date:   2015-10-24 12:00:00
categories: docker elasticsearch-v2 geolocation sort filter
tags: Docker Elasticsearch-v2 Geolocation Sort Filter
---

## Overview

Now that Elasticsearch v2.0.0 is out
lets take a quick look at how
quick and easy it is
to explore it's features using Docker.

This post will demonstrate
two Elasticsearch Geolocation features
using the official Docker image with Linux Ubuntu 14.04:

1. Sorting by Distance
2. Geo Distance Filter

All the following steps are commands that you will run from a terminal.

You can type them out or copy and paste them directly.

## Setup

Clone the code repository and cd into it:

- [https://github.com/rudijs/elasticsearch-geolocation-demo](https://github.com/rudijs/elasticsearch-geolocation-demo)
- `git clone https://github.com/rudijs/elasticsearch-geolocation-demo.git`
- `cd elasticsearch-geolocation-demo`

Start an Elasticsearch Docker container.

This command will download the official Elasticsearch v2.0.0 image from hub.docker.com and start a container instance:

- `sudo docker run -d --name elasticsearch -v "$PWD/esdata":/usr/share/elasticsearch/data -p 9200:9200 elasticsearch:2.0.0 -Des.network.bind_host=0.0.0.0`

<!-- todo: not working fully for elasticsearch v2
Let's install the *elasticsearch-head* plugin so we'll have a web admin tool to see our data:

- `sudo docker exec -it elasticsearch /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head`

Open a browser tab at [http://localhost:9200/_plugin/head/](http://localhost:9200/_plugin/head/)
-->

## Create

**Create a new Elasticsearch index**

- `curl -XPUT http://localhost:9200/geo`

<!-- curl -XDELETE http://localhost:9200/geo -->

**Create a mapping on the new index for our data**

- [mapping_place.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/mapping_place.json)
- `curl -XPUT localhost:9200/geo/_mapping/place --data-binary @mapping_place.json`

<!-- curl -XDELETE localhost:9200/geo/_mapping/place -->

## Geo Location Data

For this demostration we'll be using the East Coast of Australia from Cairns to Hobart.

![Australian East Coast](https://raw.githubusercontent.com/rudijs/elasticsearch-geolocation-demo/master/australian_east_coast.png "Australian East Coast")

**Add some 'places' to our index**

Click the links to the *place_* files below to view the JSON query objects being sent to the server.

- [place_brisbane.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/place_brisbane.json)
- `curl -XPOST http://localhost:9200/geo/place/  --data-binary @place_brisbane.json`
- [place_sydney.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/place_sydney.json)
- `curl -XPOST http://localhost:9200/geo/place/  --data-binary @place_sydney.json`
- [place_melbourne.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/place_melbourne.json)
- `curl -XPOST http://localhost:9200/geo/place/  --data-binary @place_melbourne.json`

## Search

Search all data (no sorting or filters):

- `curl -s -XGET http://localhost:9200/geo/place/_search`
- **Results**: Brisbane, Sydney, Melbourne

<!--
curl -s -XGET http://localhost:9200/geo/place/_search | jq '.hits.hits[]._source.place'
curl -s -XGET http://localhost:9200/geo/place/_search?q=melbourne | jq '.hits.hits[]._source.place'
-->

The next searches we'll do will be JSON objects that we POST to the server.

Click the links to the *query_* files below to view the JSON query objects being sent to the server.

Search and sort by distance:

- Search from Cairns in Far North Queensland (Top of the map): [query_distance_from_cairns.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/query_distance_from_cairns.json)
- `curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_cairns.json`
- **Results**: Brisbane, Sydney, Melbourne

<!--
- `curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_cairns.json | jq '.hits.hits[]._source.place'`
-->

- Search from Hobart in Tasmania (Bottom of the map): [query_distance_from_hobart.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/query_distance_from_hobart.json)
- `curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_hobart.json`
- **Results**: Melbourne, Sydney, Brisbane

<!--
curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_hobart.json | jq '.hits.hits[]._source.place'
-->

- Search from Canberra, the National Capital, which is nearer to Sydney: [query_distance_from_canberra.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/query_distance_from_canberra.json)
- `curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_canberra.json`
- **Results**: Sydney, Melbourne, Brisbane


<!--
curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_canberra.json | jq '.hits.hits[]._source.place'
-->

Search, sort and filter by distance:

- Search from Hobart in Tasmania (Bottom of the map) and limit the distance range to 1,500km: [query_distance_from_hobart_filter_1500km.json](https://github.com/rudijs/elasticsearch-geolocation-demo/blob/master/query_distance_from_hobart_filter_1500km.json)
- `curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_hobart_filter_1500km.json`
- **Results**: Melbourne, Sydney

<!--
curl -s -XPOST http://localhost:9200/geo/place/_search --data-binary @query_distance_from_hobart_filter_1500km.json | jq '.hits.hits[]._source.place`
-->

<!--
Cairns, Queensland, Australia
-16.917506, 145.760665

Hobart, Tasmania, Australia
-42.881856, 147.323999

Canberra, Australian Capital Territory, Australia
-35.282152, 149.125223
-->

## Summary

Deploying Elasticsearch v2.0.0 with Docker and using Elasticsearch's Geolocation features is clean, simple and very powerful.

For complete details on the tools and options available consult the [Official Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/query-dsl-filters.html)

I hope this post gets you up and running quickly and painlessly - ready to explore more of the power of the Elasticsearch.

Comments and feedback are very much welcomed.

If I've overlooked anything, if you can see room for improvement or if any errors please do let me know.

Thanks!
