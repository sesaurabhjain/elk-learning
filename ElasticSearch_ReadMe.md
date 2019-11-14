
### ElasticSearch,
* It's a real time distributed search and analytics engine.
*  It' provide full text search (examle Google Search, search product name at amazon or flipkart)
* It provide NoSQL solution.
* It's document oriented
* Scalable
*  Real timeor can say ver very fast
* Logstash push the data into elasticsearch
* FileBean analyse the logs file and import the data into elasticsearch in real time.

### Basic terms used in Elasticsearch
* **Document**: Document is a json file, Each document has a unique Id, In elasticsearch we do store and retrieve the data in document form
*  **Type**: We devides diff document  document in type, Type is like Table in RDBMS.
* **Index**:   It's a collection of type, Can say Index is schema.
* **Inverted Index** : It's a mapping of terms in all document and very usefull while searching
**Note** : Ideally we are not support to compare it with RDBMS but just to understand
I am proving below terms

|Elastic Search| RDBMS|
|----------------|----|
|Index| Schema   |
|Type|Table|
|JSON doc| Table Row |

* Term Frequency : is how often a term occurs in a document.
* Document Frequency: how often a term occurs in all document.
* Term Frq/Doc Frq means the relavance of a term in that document
### Way to access ElasticSearch
1) RestAPI
2) Rich client library:  In many programming language we can access Elasticsearch by using client lib
3) analytics Tool like Kibana
### Elasticsearch Archetecture
* An index is splitted in shards, It is created at the time of creating index and number of shard can not be changed later. If we need then we have to create index again.
*  Each shard can be on any node in cluster
* Each shared maintain a replica also at diff node
* Read operation can be from either shard or replica
* Write Operation will always be on shard and then it will be automatically replicated to replica
#### Mapping
Mapping is schema definition.

Mapping controls the data type of document's fields stored in an index,It defines the data type like geo_point or string of data present in the documents and rules to control the mapping of dynamically added fields.
```json
PUT bankaccountdetails 
{  "mappings":
        {  "properties":
           {
            "name":{  "type":"text"},
            "date":{  "type":"date"},
            "balance":{  "type":"double"},
            "liability":{  "type":"double"}  
          }
       }
}
```
and the response is
```json
{
   "acknowledged" : true,
   "shards_acknowledged" : true,
   "index" : "bankaccountdetails"
}
```
We can set data type for specific doc type also.
```json
PUT bankaccountdetails 
{  "mappings":
     "externalbank": {
        "properties":{
              "_all": {"enabled": false}  //set true if we need full text search otherwise false
              "date": { "type":"date"} }
        }
     }    
}
```
**Note** : **_all** is added automatically that holds the data of all fields, and used while full text search but it takes extra space so do not set if full text serach is not req.
 
 Mapping can be used for below stupff.

 1. fields type : we can set fileds type like byte, short, interger, log, float, double, boolean, data, string etc
       ```json
      "properties": {
          user_id: {type:string}
       }
    ```
 2. field index: The index attribute controls how the string will be indexed. It can contain one of three values.
    ```json
      "properties": {
          user_id: {index: "not_analyzed"}
       }
    ```
     -   **analyzed:** First analyze the string, and then index it. In other words, index this field as full text.
     -   **not_analyzed:** Index this field so it is searchable, but index the value exactly as specified. Do not analyze it.
       -   **no:** Don’t index this field at all. This field will not be searchable.

3. field analyzer : string fields are passed through an analyzer to convert the string into a stream of tokens or terms
```json

 ```json
      "properties": {
          address: {
              type:"text",
              index: "not_analyzed".
              analyzer: "english"
          }
       }
    ```
```
Question: `What if we need to to search/sort based on not_analysed`
Solution: cretae a replica field 
```json
 "properties": {
          address: {
              type:"text",
              fields:{
                   row:{
                      type:string, 
                      index: 'not_analysed'
                    }
               }
          }
       }
       ?sort=address.row&pretty
```

### Analyzers
When a query is processed during a search operation, the content in any index is analyzed by analyser, We can create custom analyser
List of provided analyser
1) standard: splict on words boundries, remove pantuation, lowercase, 
2) simple : split on anythign that is not latter and lowercase
3) whilespace : split based on whitespace but does not lowercase
4) language (ie english) , work like standard and extra capability to understand the text such as stop words , synonums, etc
```json
POST _analyze 
{  "analyzer":  "standard",
   "text":  "Today's weather is beautiful"  
}
```

```json
PUT my_index
{
   "settings":{
      "analysis":{
         "analyzer":{
            "my_analyzer":{ 
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase" 
               ]
            },
            "my_stop_analyzer":{ 
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase",
                  "english_stop"
               ]
            }
         },
         "filter":{
            "english_stop":{
               "type":"stop",
               "stopwords":"_english_"
            }
         }
      }
   },
   "mappings":{
       "properties":{
          "title": {
             "type":"text",
             "analyzer":"my_analyzer", 
             "search_analyzer":"my_stop_analyzer", 
             "search_quote_analyzer":"my_analyzer" 
         }
      }
   }
}

PUT my_index/_doc/1
{
   "title":"The Quick Brown Fox"
}

PUT my_index/_doc/2
{
   "title":"A Quick Brown Fox"
}

GET my_index/_search
{
   "query":{
      "query_string":{
         "query":"\"the quick brown fox\"" 
      }
   }
}
```
### Tokenizer and filter:  
 * Tokenizer : spilt the words,
 * filter: used while tokenize like lowercase, stopwords

### Insert data into index
1) Inserting single record
```json
PUT localhost:9200/movie/109486
{
   "genre" : ["IMAx","Sci-Fi"],
   "title":"A Quick Brown Fox",
   "year" : 2014
}
```
2) Inserting multiple record
We can run multiple queryes in a single post by _buil command
    1) `POST /_bulk`
` 2) POST /<index>/_bulk`
```json
POST  _bulk 
 {  "index"  :  {  "_index"  :  "test",  "_id"  :  "1"  }  }
 {  "field1"  :  "value1"  } 
 {  "delete"  :  {  "_index"  :  "test",  "_id"  :  "2"  }  }
 {  "create"  :  {  "_index"  :  "test",  "_id"  :  "3"  }  }
 {  "field1"  :  "value3"  }
 {  "update"  :  {"_id"  :  "1",  "_index"  :  "test"}  }
 {  "doc"  :  {"field2"  :  "value2"}  }
```
### Update doc 
```json
PUT  test/_doc/1  //create
 {  "counter"  :  1,  "tags"  :  ["red"]  }
or 
POST  test/_update/1 //and then update existing counter by 4
{  "script"  : 
 {  "source":  "ctx._source.counter += params.count",  "lang":  "painless",  
 "params"  :  {  "count"  :  4  }  }  }
 POST  test/_update/1 //add new field name
  {  "doc"  :  {  "name"  :  "new_name"  }  }
```
### upsert 
if document exist then it will be updated otherwise new one will be created
```json
POST  test/_update/1  
{  "doc"  :  {  "name"  :  "new_name"  }, 
    "doc_as_upsert"  :  true
  }
```
##   
### Delete 
Removes a JSON document from the specified index.
```json
DELETE /<index>/_doc/<_id>
```

### Dealing with concurancy
`optimistic concurancy management`
* It's ok in case of get req.
* for post and upate we do send the update with `?if_seq_no=7&if_primary_term=1` number so if any other client has updated then record will not be updated then send appropriate error msg like invalid sen_no or primary_term number.
## Search
1`Query line search`

   1 GET /index/_search
   2 POST /index/_search
   3 GET /_search
   4 POST /_search

### Search with Query String
```json
GET  /twitter/_search?q=tag:wow
```
but it's not good because each time we have to encode the url.
### Search JSON as in request body
 type of queries
 * query_string
*  match
*  match_phrase
* match_all
*  multiple_match
*  bool
```json
GET  /_search  
 {  "query":{
        "query_string"  : {
            "fields"  :  ["content",  "name"],
            "query"  :  "this AND that"
         }
     }
 }
```
OR
```json
GET  /_search 
 {  "query":  { 
          "match"  :  { 
                 "description"  :  "this is a test"  
           }
     }
  }
```  
```json
GET _search
{
    "query": {
        "range" : {
            "age" : {
                "gte" : 10,
                "lte" : 20,
                "boost" : 2.0
            }
        }
    }
}
```
#### Boolean query 
bool apply and among the options like must, must_not, filter
```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "match" : { "title" : "kimchy" }
      },
      "filter": {
         "range" : {
              "age" : { "gte" : 10, "lte" : 20 }
          }
      },
      "must_not" : {
        "term" : { "tag" : "tech" }        
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```
* Term filer by exact values 
  `{"term : {"year": 2014}"}`
*  terms: match if any exact values in a list
`{"terms : {"year": [2014,2015]}}`
* range: find the value in range
`{"range" :{"age":{gt:18,lt<20}}}`
* exists: find doc where field exist
`{ "exists": {"field": "tag"}`
* missing: find doc where field is missing
`{ "missing": {"field": "tag"}`
* bool : combine filters with boolean logic (like must , must not, shoud etc)*
### Paginnation
```json
GET /_search
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
### Sorting
```json
GET /my_index/_search
{
    "sort" : [
        { "date" : {"order" : "asc"}}
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### Fuzzy sarch
```json
GET /_search
{
    "query": {
        "fuzzy": {
            "user": {
                "value": "ki",
                "fuzziness": "AUTO",// AUTO-depends on string size, we can set it like 1 or 2
                "max_expansions": 50,
                "prefix_length": 0,
                "transpositions": true,
                "rewrite": "constant_score"
            }
        }
    }
}
```
### Partial Matching by prefix, wild card and reg exp
```json
GET /my_index/_search
{
    "query" : {
        "prefix" :{year: 201}
    }
}

GET /my_index/_search //wild card is better the prefix
{
    "query" : {
        "wildcard" :{year: 20*} 
    }
}
GET /my_index/_search //wild card is better the prefix
```json
{
    "query" : {
        "regexp" :{year: "^20"} 
    }
}
```
### N-Grams, You tube like sarch
It actually devides the text into 1 to n size part , basically we create analyser by using this and then those analyser are used to auto completion kind of search. 
### Importing data into elastic-search

As we have seen importing data into elasticsearch from Json by _bulk but is is not sufficient to import only json data, in real world we may need to import data from logs, db or csv etc.

* Traditional way read csv or any other file construct the json and make a _butl rest call to import it into elasticsearch.
* There are languages like java which provide api to to read csv and convert it inot well formatted JSON and import import it into elasticsearch.



### Logstash
Logstash is a server-side data processing pipeline that get the data from a multitude of sources simultaneously, filter, transforms it, and then sends it to your favorite "stash." (Ours is Elasticsearch, naturally.).
### Aggregation
It provide _aggregated_ data based on a search query
```json
curl -X POST "localhost:9200/exams/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{
    "aggs" : {
        "avg_grade" : { "avg" : { "field" : "grade" } }
    }
}
//
curl -X POST "localhost:9200/ledger/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {"title": "Ken"}
    },
    "aggs" : {
        "avg_grade" : { "avg" : { "field" : "grade" } }
    }
}
```
`Hitograms` bar chart
```json
curl -X POST "localhost:9200/sales/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{   "query":{
          "match":{"title":"super"}
     },
    "aggs" : {
        "prices" : {
            "histogram" : {
                "field" : "price",
                "interval" : 50
            }
        }
    }
}
```
`Agreegation Time series data`
```json
curl -X POST "localhost:9200/sales/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            }
        }
    }
}
```
`Nested Agreegation`:

### Kibana
Kibana is an open source data visualization  tool for Elasticsearch.
It provides visualization capabilities on top of the content indexed on an Elasticsearch.
Users can create bar, line and scatter plots, or pie charts and maps on top of large volumes of data

### Elasticsearch Operations

1) Request works in round robin way among all nodes
2) Write requests are routed to the primary shard, then replicated
3) Read request are routed any one primary/replica
4) Basically each index has two primary shards and tow replicas
5) we can not add new shards later, if we need we have to re-create index.
6) at time time of creating index we do define number of shards and replicas
7) we can also create template for automatically allocating number of shards, replica, analyzers etc.
8) By default number of shards are 5 but we can change this setting in conf file.
9) create more indices to manage more data like `logstash` create a separate indices for each day log data and if we need to data of last 3 months of we can combine like logs_June_*,logs_july_*,logs_august_* .
10) generally we do select 64 GB RAM machine to setup elastic search, so that we can  allocate 32GB RAM to elasticsearch itself.
11) allocating more then 32 GB RAM is also not a good idea. Because elasticsearch has been made on the top of JAVA and in JAVA heap memory is cleaned up by gc and it works faster when heap size is less.
12) Use SSD hard disk,
### X-pack
It's actually a plugin for elasticsearch and kibana for monitoring, security and machine learning.
It's paid but we have some limited free feature.

### Fail-over in action
1) Setup a second elasticsearch node 
2) observe how elasticsearch expand to this node automatically
3) stop first (master) node and observe how things are moved to second node automatically 
4)  start first node observe how thing are moved back to first node automatically

### Backup 
snapshot let you backup your indices to local file system or any other places over protocol ex we can backup it at Amazon S3
`steps for backup`
set the backup path in elasticsearch.yml
1) path.repo: /home/<user>/backups/
2) 
```json
PUT _snapshot/backup-repo/snapshot-1
{
type : "fs",
settings: {location: '/home/user/backups/backup-repo'}
}
```
### Restore
POST /_all/_close
POST _snapshot/backup-repo/snapshot-1/_restore
### Maintenance work
1) stop adding new data into cluster
2) disable all shards /close all your indices
3) shut down one node
4) perform the maintenance activity
5) restart the node re-enable shards and 
6 ) wait for cluster to go in green state
7) repeat above steps to all ndoes.
8) and all done :)
