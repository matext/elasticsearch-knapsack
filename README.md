![Knapsack](https://github.com/jprante/elasticsearch-knapsack/raw/master/src/site/resources/hitchhiker-691581_1280.jpg)

Image taken from Pixabay.com https://pixabay.com/de/tramper-daumen-hoodie-rucksack-691581/ 
without license (CC0 Public domain)

# Knapsack plugin for Elasticsearch

Knapsack is an "swiss knife" export/import plugin for [Elasticsearch](http://github.com/elasticsearch/elasticsearch).
It uses archive formats (tar, zip, cpio) and also Elasticsearch bulk format with 
compression algorithms (gzip, bzip2, lzf, xz).

A pull or push of indexes or search hits with stored fields across clusters is also supported.

The knapsack actions can be executed via HTTP REST, or in Java using the Java API.

In archive files, the following index information is encoded:

- index settings
- index mappings
- index aliases

When importing archive files again, this information is reapplied.

## Compatibility matrix

![Travis](https://travis-ci.org/jprante/elasticsearch-knapsack.png)

| Elasticsearch  |   Plugin       | Release date |
| -------------- | -------------- | ------------ |
| 2.0.0-beta2    | 2.0.0-beta2.1  | Sep 27, 2015 |
| 2.0.0-beta2    | 2.0.0-beta2.0  | Sep 22, 2015 |
| 1.6.2          | 1.6.2.0        | Sep 27, 2015 |
| 1.5.2          | 1.5.2.1        | Sep 27, 2015 |
| 1.5.2          | 1.5.2.0        | Apr 30, 2015 |
| 1.4.5          | 1.4.5.0        | Apr 30, 2015 |
| 1.5.1          | 1.5.1.0        | Apr 22, 2015 |
| 1.4.4          | 1.4.4.1        | Apr 22, 2015 |
| 1.3.4          | 1.3.4.0        | Apr 16, 2015 |
| 1.4.0          | 1.4.0.0        | Feb 19, 2015 |
| 1.2.4          | 1.2.4.1        | Sep 30, 2014 |
| 1.1.2          | 1.1.2.1        | Sep 30, 2014 |
| 1.0.3          | 1.0.3.1        | Sep 30, 2014 |
| 1.2.1          | 1.2.1.0        | Jun 13, 2014 |
| 1.2.0          | 1.2.0.0        | May 23, 2014 |
| 1.1.0          | 1.1.0.0        | May 25, 2014 |
| 1.0.0          | 1.0.0.1        | Feb 16, 2014 |
| 0.90.11        | 0.90.11.2      | Feb 16, 2014 |
| 0.20.6         | 0.20.6.2       | Feb 16, 2014 |
| 0.19.11        | 0.19.11.3      | Feb 15, 2014 |
| 0.19.8         | 0.19.8.1       | Feb 15, 2014 |

## Installation 1.x

    ./bin/plugin -install knapsack -url http://xbib.org/repository/org/xbib/elasticsearch/plugin/elasticsearch-knapsack/1.7.2.0/elasticsearch-knapsack-1.7.2.0-plugin.zip

## Installation 2.x

    ./bin/plugin install http://xbib.org/repository/org/xbib/elasticsearch/plugin/elasticsearch-knapsack/2.0.0-beta2.0/elasticsearch-knapsack-2.0.0-beta2.0-plugin.zip

Do not forget to restart the node after installation.

## Project docs

The Maven project site is available at [Github](http://jprante.github.io/elasticsearch-knapsack)

## Overview

![Diagram](https://github.com/jprante/elasticsearch-knapsack/raw/master/src/site/resources/knapsack-diagram-2.png)

# Example

Let's go through a simple example:

    curl -XDELETE localhost:9200/test
    curl -XPUT localhost:9200/test/test/1 -d '{"key":"value 1"}'
    curl -XPUT localhost:9200/test/test/2 -d '{"key":"value 2"}'

# Export

You can export this Elasticsearch index with

    curl -XPOST localhost:9200/test/test/_export
    {"running":true,"state":{"mode":"export","started":"2014-09-28T19:18:27.447Z","path":"file:///Users/es/elasticsearch-1.3.2/test_test.tar.gz","node_name":"Slither"}}

The result is a file in the Elasticsearch folder

    -rw-r--r--   1 joerg  staff          343 28 Sep 21:18 test_test.tar.gz
   
Check with tar utility, the settings and the mapping is also exported

    tar ztvf test_test.tar.gz 
    -rw-r--r--  0 joerg  0         133 28 Sep 21:18 test/_settings/null/null
    -rw-r--r--  0 joerg  0          49 28 Sep 21:18 test/test/_mapping/null
    -rw-r--r--  0 joerg  0          17 28 Sep 21:18 test/test/1/_source
    -rw-r--r--  0 joerg  0          17 28 Sep 21:18 test/test/2/_source

Also, you can export a whole index with

    curl -XPOST localhost:9200/test/_export

with the archive file `test.tar.gz`, or even all cluster indices with

    curl -XPOST 'localhost:9200/_export'

to the file `_all.tar.gz`

## Available suffixes for archive formats

    .tar
    .zip
    .cpio
    .bulk

## Available suffixes for compression

    .gz
    .bzip2
    .xz
    .lzf

By default, the archive format is `tar` with compression `gz` (gzip). 

You can also export to `zip`, `cpio` or `bulk` archive format.

Available compression codecs are `bz2` (bzip2), `xz` (Xz), or `lzf` (LZF)

Note: if you use the `bulk` format, you create Elasticsearch bulk format.

## Export search results

You can add a query to the `_export` endpoint just like you would do for searching in Elasticsearch.

    curl -XPOST 'localhost:9200/test/test/_export' -d '{
       "query" : {
           "match_phrase" : {
               "key" : "value 1"
           }
       },
       "fields" : [ "_parent", "_source" ]
    }'

## Export to an archive with a given path name

You can configure an archive path with the parameter `path`

    curl -XPOST 'localhost:9200/test/_export?path=/tmp/myarchive.zip'

If Elasticsearch can not write to the path, an error message will appear, and no export will take place.
You can force overwrite with the parameter `overwrite=true`

## Export split by byte size

You can create multiple archive files with the parameter `bytes`

    curl -XPOST 'localhost:9200/test/_export?path=/tmp/myindex.bulk&bytes=10m'

This creates `myindex.bulk`, `1.myindex.bulk`, `2.myindex.bulk` ... where all archive files are around 10 megabytes.

## Renaming indexes and index types

You can rename indexes and index types by adding a `map` parameter that contains a JSON
object with old and new index (and index/type) names.

    curl -XPOST 'localhost:9200/test/type/_export?map=\{"test":"testcopy","test/type":"testcopy/typecopy"\}'

Note the backslash, which is required to escape shell interpretation of curly braces.

## Push or pull indices from one cluster to another

If you want tp push or pull indices from one cluster to another, Knapsack is your friend.

You can copy an index in the local cluster or to a remote cluster with the `_push` or the `_pull` endpoint.
This works if you have the same Java JVM version and the same Elasticsearch version.

Example for a local cluster copy of the index `test` to `testcopy`

    curl -XPOST 'localhost:9200/test/_push?map=\{"test":"testcopy"\}'

Example for a remote cluster copy of the index `test` by using the parameters `cluster`, `host`, and `port`

    curl -XPOST 'localhost:9200/test/_push?&cluster=remote&host=127.0.0.1&port=9201'

This is a complete example that illustrates how to filter an index by timestamp and copy this part to
another index

    curl -XDELETE 'localhost:9200/test'
    curl -XDELETE 'localhost:9200/testcopy'
    curl -XPUT 'localhost:9200/test/' -d '
    {
        "mappings" : {
            "_default_": {
                "_timestamp" : { "enabled" : true, "store" : true, "path" : "date" }
            }
        }
    }
    '
    curl -XPUT 'localhost:9200/test/doc/1' -d '
    {
        "date" : "2014-01-01T00:00:00",
        "sentence" : "Hi!",
        "value" : 1
    }
    '
    curl -XPUT 'localhost:9200/test/doc/2' -d '
    {
        "date" : "2014-01-02T00:00:00",
        "sentence" : "Hello World!",
        "value" : 2
    }
    '
    curl -XPUT 'localhost:9200/test/doc/3' -d '
    {
        "date" : "2014-01-03T00:00:00",
        "sentence" : "Welcome!",
        "value" : 3
    }
    '
    curl 'localhost:9200/test/_refresh'
    curl -XPOST 'localhost:9200/test/_push?map=\{"test":"testcopy"\}' -d '
    {
        "fields" : [ "_timestamp", "_source" ],
        "query" : {
             "filtered" : {
                 "query" : {
                     "match_all" : {
                     }
                 },
                 "filter" : {
                    "range": {
                       "_timestamp" : {
                           "from" : "2014-01-02"
                       }
                    }
                 }
             }
         }
    }
    '
    curl '0:9200/test/_search?fields=_timestamp&pretty'
    curl '0:9200/testcopy/_search?fields=_timestamp&pretty'

# Import

You can import the file with the `_import` endpoint

    curl -XPOST 'localhost:9200/test/test/_import'

Knapsack does not delete or overwrite data by default.
But you can use the parameter `createIndex` with the value `false` to allow indexing to indexes that exist.

When importing, you can map your indexes or index/types to your favorite ones.

    curl -XPOST 'localhost:9200/test/_import?map=\{"test":"testcopy"\}'

## Modifying settings and mappings

You can overwrite the settings and mapping when importing by using parameters in the form 
`<index>_settings=<filename>` or `<index>_<type>_mapping=<filename>`. 

General example::

    curl -XPOST 'localhost:9200/myindex/mytype/_import?myindex_settings=/my/new/mysettings.json&myindex_mytype_mapping=/my/new/mapping.json'

The following statements demonstrate how you can change the number of shards from the default `5` to `1` 
and replica from `1` to `0` for an index `test`

    curl -XDELETE localhost:9200/test
    curl -XPUT 'localhost:9200/test/test/1' -d '{"key":"value 1"}'
    curl -XPUT 'localhost:9200/test/test/2' -d '{"key":"value 2"}'
    curl -XPUT 'localhost:9200/test2/foo/1' -d '{"key":"value 1"}'
    curl -XPUT 'localhost:9200/test2/bar/1' -d '{"key":"value 1"}'
    curl -XPOST 'localhost:9200/test/_export'
    tar zxvf test.tar.gz test/_settings
    echo '{"index.number_of_shards":"1","index.number_of_replicas":"0"}' > test/_settings/null/null
    curl -XDELETE 'localhost:9200/test'
    curl -XPOST 'localhost:9200/test/_import?test_settings=test/_settings/null/null'
    curl -XGET 'localhost:9200/test/_settings?pretty'
    curl -XPOST 'localhost:9200/test/_search?q=*&pretty'

The result is a search on an index with just one shard.

    {
      "took" : 19,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "failed" : 0
      },
      "hits" : {
        "total" : 2,
        "max_score" : 1.0,
        "hits" : [ {
          "_index" : "test",
          "_type" : "test",
          "_id" : "1",
          "_score" : 1.0,
          "_source":{"key":"value 1"}
        }, {
          "_index" : "test",
          "_type" : "test",
          "_id" : "2",
          "_score" : 1.0,
          "_source":{"key":"value 2"}
        } ]
      }
    }

## State of knapsack import/export actions

While exports or imports or running, you can check the state with

    curl -XPOST 'localhost:9200/_export/state'

or

    curl -XPOST 'localhost:9200/_import/state'

## Aborting knapsack actions

If you want to abort all running knapsack exports/import, you can do this by

    curl -XPOST 'localhost:9200/_export/abort'

or

    curl -XPOST 'localhost:9200/_import/abort'

# Java API

Knapsack implements all actions as Java transport actions in ELasticsearch.

You can consult the junit tests for finding out how to use the API. To give you an impression, 
here is just an example for a very minimal export/import cycle using the `bulk` archive format.

        
        client.index(new IndexRequest().index("index1").type("test1").id("doc1")
             .source("content","Hello World").refresh(true)).actionGet();
        
        File exportFile = File.createTempFile("minimal-import-", ".bulk");
        Path exportPath = Paths.get(URI.create("file:" + exportFile.getAbsolutePath()));
        KnapsackExportRequestBuilder requestBuilder = new KnapsackExportRequestBuilder(client.admin().indices())
                .setPath(exportPath)
                .setOverwriteAllowed(false);
        KnapsackExportResponse knapsackExportResponse = requestBuilder.execute().actionGet();

        KnapsackStateRequestBuilder knapsackStateRequestBuilder =
               new KnapsackStateRequestBuilder(client.admin().indices());
        KnapsackStateResponse knapsackStateResponse = knapsackStateRequestBuilder.execute().actionGet();

        Thread.sleep(1000L);

        client.admin().indices().delete(new DeleteIndexRequest("index1")).actionGet();

        KnapsackImportRequestBuilder knapsackImportRequestBuilder = new KnapsackImportRequestBuilder(client.admin().indices())
                .setPath(exportPath);
        KnapsackImportResponse knapsackImportResponse = knapsackImportRequestBuilder.execute().actionGet();



# Caution

Knapsack is very simple and works without locks or snapshots. This means, if Elasticsearch is
allowed to write to the part of your data in the export while it runs, you may lose data in the export.
So it is up to you to organize the safe export and import with this plugin.

If you want a more advanced feature, please use the snapshot/restore which is the standard
procedure for saving/restoring data in Elasticsearch:

http://www.elasticsearch.org/blog/introducing-snapshot-restore/

# Credits

Knapsack contains derived work of Apache Common Compress
http://commons.apache.org/proper/commons-compress/

The code in this component has many origins:
The bzip2, tar and zip support came from Avalon's Excalibur, but originally
from Ant, as far as life in Apache goes. The tar package is originally Tim Endres'
public domain package. The bzip2 package is based on the work done by Keiron Liddle as
 well as Julian Seward's libbzip2. It has migrated via:
Ant -> Avalon-Excalibur -> Commons-IO -> Commons-Compress.
The cpio package has been contributed by Michael Kuss and the jRPM project.

Thanks to `nicktgr15 <https://github.com/nicktgr15>` for extending Knapsack to support Amazon S3.

# License

Knapsack Plugin for Elasticsearch

Copyright (C) 2012 Jörg Prante

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
