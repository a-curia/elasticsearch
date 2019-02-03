# Elasticsearch

### Updating Whole and Partial Documents

	curl -XGET 'localhost:9200/products/mobiles/3?pretty'

	curl -XPUT 'localhost:9200/products/mobiles/3?pretty' -H 'Content-Type: application/json' -d'{
	"name" : "iPhoneX",
	"camera" : "12MP",
	"storage" : "256GB",
	"display" : "4.7inch",
	"battery" : "1,960mAh",
	"reviews" : [
	  "incredible",
	  "good price quality",
	  "very bad"
	]
	}'

	curl -XPOST 'localhost:9200/products/mobiles/3/_update?pretty' -H 'Content-Type: application/json' -d'{ "doc": { "color": "black" } }'

_update command can be used to update current fields or create new fields inside the document

	curl -XPUT 'localhost:9200/myshoes/shoes/1?pretty' -H 'Content-Type: application/json' -d'{
	"name": "Adidas",
	"size": 9,
	"color": "black"
	}'

curl -XPOST 'localhost:9200/myshoes/shoes/1/_update?pretty' -H 'Content-Type: application/json' -d'{
"script": "ctx._source.size += 2"
}'

### Deleting Documents and Indices
	curl -XDELETE 'localhost:9200/products/mobiles/3?pretty'

check if the document exists in a specific type

	curl -i -XHEAD 'localhost:9200/products/mobiles/3?pretty'

delete the entire index

	curl -XDELETE 'localhost:9200/products?pretty'

### Performing Bulk Operations on Documents

bulk operations are not possible but we can retrieve multiple documents, index multiple document and do multiple operations on one command

	curl -XGET 'localhost:9200/_mget?pretty' -H 'Content-Type: application/json' -d'{ "docs":[
        {
            "_index":"products",
            "_type":"mobiles",
            "_id":"1"
        },
        {
            "_index":"products",
            "_type":"mobiles",
            "_id":"2"
        }
    ]}'

	curl -XGET 'localhost:9200/products/_mget?pretty' -H 'Content-Type: application/json' -d'{ "docs":[
        {
            "_type":"mobiles",
            "_id":"1"
        },
        {
            "_type":"mobiles",
            "_id":"2"
        }
    ]}'

	curl -XGET 'localhost:9200/products/mobiles/_mget?pretty' -H 'Content-Type: application/json' -d'{ "docs":[
        {
            "_id":"1"
        },
        {
            "_id":"2"
        }
    ]}'

bulk operations - add new document to the index

	curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' -d'{ "index": { "_index": "products", "_type": "shoes", "_id": "3" } } { "name": "Puma", "size": 9, "color": "black" } { "index": { "_index": "products", "_type": "shoes", "_id": "4" } } { "name": "Nike", "size": 9, "color": "black" }'

first add a new document to the index; the second line specifies the document to be added

	curl -XPOST 'localhost:9200/products/_bulk?pretty' -H 'Content-Type: application/json' -d'{ "index": { "_type": "shoes", "_id": "3" } } { "name": "Puma", "size": 9, "color": "black" } { "index": { "_type": "shoes", "_id": "4" } } { "name": "Nike", "size": 9, "color": "black" }'

	curl -XPOST 'localhost:9200/products/shoes/_bulk?pretty' -H 'Content-Type: application/json' -d'{ "index": { "_id": "3" } } { "name": "Puma", "size": 9, "color": "black" } { "index": { "_id": "4" } } { "name": "Nike", "size": 9, "color": "black" }'

to auto generated the IDs of these documents leave out the index as empty document

	curl -XPOST 'localhost:9200/products/shoes/_bulk?pretty' -H 'Content-Type: application/json' -d'{ "index": {} { "name": "Puma", "size": 9, "color": "black" } { "index": {} } { "name": "Nike", "size": 9, "color": "black" }'

	curl -XPOST 'localhost:9200/products/shoes/_bulk?pretty' -H 'Content-Type: application/json' -d'{ "index": { "_id": "3" } } { "name": "Puma", "size": 9, "color": "black" } { "index": { "_id": "4" } } { "name": "Nike", "size": 9, "color": "black" } { "delete": {"_id" : "2"}} {"create": {"_id":"5"} } {"name": "Nike Power", "size": 12, "color": "black"} {"update": {"_id":"1"}} {"doc": {"color": "orange"}}'

### Bulk Indexing of Documents from a JSON File
the customers.json file

{"index": {}}
{"name": "Carlson Barnes", "age": 34}
{"index": {}}
{"name": "Sheppard Stein", "age": 39}
{"index": {}}
{"name": "Nixon Singleton", "age": 36}
{"index": {}}
{"name": "Sharron Sosa", "age": 33}
{"index": {}}
{"name": "Kendra Cabrera", "age": 24}
{"index": {}}
{"name": "Young Robinson", "age": 20}

you can chose to have and index specifying the id, type and index name like this
{"index": {"_id":"4", "_type":"personal", "_index":"customers"}}

bulk on the json file

	curl -H 'Content-Type: application/json' -XPOST 'localhost:9200/customers/personal/_bulk?pretty&refresh' --data-binary @"customers.json"
you don't need to create the index and type upfront

### Setting up Fake Data for Queries

https://www.json-generator.com/

[
'{{repeat(1000, 1000)}}',
{
name: '{{firstName()}} {{surname()}}',
age: '{{integer(18,75)}}',
gender: '{{gender()}}',
email: '{{email()}}',
phone: '+1 {{phone()}}',
street: '{{integer(100, 999)}} {{street()}}',
city: '{{city()}}',
state: '{{state()}}, {{integer(100.1000)}}'
}
]

delete and recreate the customer index from the customers_full.json file

curl -XGET 'localhost:9200/_cat/indices?v&pretty'

curl -XDELETE 'localhost:9200/customers'

curl -H 'Content-Type: application/json' -XPOST 'localhost:9200/customers/personal/_bulk?pretty&refresh' --data-binary @"customers_full.json"

the are 2 Contexts: Query Context(based on score) and Filter Context(exact or range matches)

you can specify the query terms as:

- search terms as URL query parameters
- search terms within the URL request body

Search Using Query Params

curl -XGET 'localhost:9200/customers/_search?q=wyoming&pretty'

curl -XGET 'localhost:9200/customers/_search?q=wyoming&sort=age:desc&pretty'

curl -XGET 'localhost:9200/customers/_search?q=state:kentucky&sort=age:asc&pretty'

from index 10 get only 2 documents

curl -XGET 'localhost:9200/customers/_search?q=state:kentucky&from=10&size=2&pretty'

curl -XGET 'localhost:9200/customers/_search?q=state:kentucky&explain&pretty'

Search Using the Request Body

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match_all": {} }
}
'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match_all": {} },
"size": 3
}
'

elasticsearch does stateless searches - maintains no open cursor or session for your search

only document from index 5

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match_all": {} },
"from": 5,
"size": 3
}
'

search multiple indices

curl -XGET 'localhost:9200/customers,products/_search?pretty'

search and restrict multiple Doc Types

curl -XGET 'localhost:9200/products/shoes,laptops/_search?pretty'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match_all": {} },
"sort": { "age": {"order": "desc"}},
"size": 20
}

Query params options are a subset of options available in the Request Body

Source filtering to include only those fields that we're interested in

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"term": { "name": "gates" } }
}

Term Searches - term should have an exact match in the inverted index

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"_source": false,
"query": {"term": { "name": "gates" } }
}

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"_source": "st*",
"query": {"term": { "name": "gates" } }
}

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"_source": ["st*", "*n*"],
"query": {"term": { "name": "gates" } }
}

No Change to Relevance Rankings - source filtering does not affect the relevance of documents

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"_source": {"includes":["st*", "*n*"], "excludes": ["*der"]},
"query": {"term": { "name": "gates" } }
}

### Full Text Searches - full text queries using:
- match
- match_phrase
- match_phrase_prefix

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match": {"name":"webster"}}
}'

Not an Exact Term Match - tweak other parameters to specify how the match is to be performed

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match": {"name":{"query":"frank norris", "operator":"or"}}}
}'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match_phrase": {"street":"tompkins place"}}
}'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"match_phrase_prefix": {"name":"no"}}
}'


Query clause to Relevance score : each document has a different relevance score based on the query clause

Fuzzy searches might look at how similar the search term is to the word present in the document
Term searches might look at the percentage of search terms that were found in the document

TF/IDF algorithm - term frequency/inverse document frecquency

Term Frequency - how often does the term appear in the field? - more often, more relevant

Inverse document frequency - how often does the term appear in the index? - more often, less relevant

Field-length norm - how log is the field which was searched - longer fields, less relevant

Common terms queries with cutoff_frequency specified

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"common": {"reviews":{"query":"this is great", "cutoff_frequency":0.001}}}
}'

Cutoff Frequency - terms with a frequency of >0.1% are common terms

### Boolean Query

Matches documents by combining multiple queries using boolean operators such as AND, OR

must - the clause must appear in matching documents
should - the clause may appear in matching documents but may not sometimes
must_not - the clause must not appear in the document result
filter - the clause must appear in results but results are not scored

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"bool": {"must":{"match":{"street":"ditmas"}}, {"match":{"street":"avenue"}}}}
}'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"bool": {"should":{"match":{"street":"ditmas"}}, {"match":{"street":"avenue"}}}}
}'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"bool": {"must_not":{"match":{"street":"ditmas"}}, {"match":{"street":"avenue"}}}}
}'


Term Queries - the exact term needs to be found in the inverted index for indexed documents

The terms found in the index may vary based on how you analyze them

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"bool": {"should":{"term":{"street":{"value":"ditmas"}}}, {"term":{"street":{"value":"avenue"}}}}}
}'

curl -XGET 'localhost:9200/products/_search?pretty' -d'
{
"query": {"bool": {"should":{"term":{"street":{"value":"ditmas", "boost":2.0}}}, {"term":{"street":{"value":"avenue"}}}}}
}'

### Search Using the Filter Context

When queries run in the filter context, the documents in the result are not scored

Each document responds yes/no to whether it should be included in the result

	curl -XGET 'localhost:9200/products/_search?pretty' -d'
	{
		"query": {
			"bool": {
				"must": { "match_all": {}},
				"filter": {
					"range": {
						"age": {
							"gte": 20,
							"lte": 30
						}
					}
				}
			}
		}
	}'

we can use search queries along with a filter

	curl -XGET 'localhost:9200/products/_search?pretty' -d'
	{
		"query": {
			"bool": {
				"must": { 
					"match": {
						"state": "alabama"
					}
				},
				"filter": {
					"term": {
						"gender": "female"
					},
					"range": {
						"age": {
							"gte": 20
						}
					}
				}
			}
		}
	}'


### Executing Analytical Queries Through Aggregations : Types of Aggregations

How Elasticsearch can be used beyond search as an analytical engine

4 types of Aggregations:

- Metric - aggregations over a set of documents; single or multiple values as a result; all documents in a search result; documents within a logical group
- Bucketing - logically group documents based on search query; a document falls into a bucket if the criteria matches; each bucket associated with a key
- Matrix - operates on multiple fields and produces a matrix result; every value from filed A might combine with value from field B
- Pipeline - aggregations that work on the output of other aggregations

Matrix and Pipeline are experimental and might change in the future releases.



### Implementing Metric Aggregations


	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"avg_age": {
				"avg": {
					"field": "age"
				}			
			}
		}
	}'



	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 1,
		"aggs": {
			"avg_age": {
				"avg": {
					"field": "age"
				}			
			}
		}
	}'




	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"query": {
			"bool": {
				"filter": {
					"match": {"state": "minessota"}
				}
			}
		}
		"aggs": {
			"avg_age": {
				"avg": {
					"field": "age"
				}			
			}
		}
	}'


	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 1,
		"aggs": {
			"age_stats": {
				"stats": {
					"field": "age"
				}			
			}
		}
	}'

The Cardinality Aggregation

Cardinality, the number of unique values in a field. 
Enable **fielddata** for text fields via mappings

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"age_count": {
				"cardinality": {
					"field": "age"
				}			
			}
		}
	}'


Search - inverted index of the terms present in documents; the terms themselves can be hashed and stored in the index; which documents contain this term?

Aggregation - actual value of fields present in documents; actual values of the terms are needed, hash values do not suffice; what is the value of this field for this document?

Text field values are stored in an in-memory data structure called fielddata; fielddata is built on demand when a field is used for aggregations, sorting etc; fielddata on text fields take up lots of heap space; fielddata is disabled by default on text fields


	curl -XPOST 'localhost:9200/products/_mapping/personal?pretty' -d'
	{
		"properties": {
			"gender": {
				"type": "text",
				"fielddata": true		
			}
		}
	}'

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"gender_count": {
				"cardinality": {
					"field": "gender"
				}			
			}
		}
	}'


Implementing Bucketing Aggregations

Logically group documents into buckets - analog to group by in sql; each bucket satisfies some criterion

Bucket aggregations by field values

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"gender_bucket": {
				"terms": {
					"field": "gender"
				}			
			}
		}
	}'

sum_other_doc_count - the sum of documents counts that are not part of the response

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"age_bucket": {
				"terms": {
					"field": "age"
				}			
			}
		}
	}'

Document Counts are Approximate - Calculations aggregations across index shard can result in imperfections


	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"age_ranges": {
				"range": {
					"field": "age",
					"ranges": [
						{"to": 30 },
						{"from":30,"to":40},
						{"from":40,"to":55},
						{"from":55}
					]
				}			
			}
		}
	}'

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"age_ranges": {
				"range": {
					"field": "age",
					"keyed": true,
					"ranges": [
						{"to": 30 },
						{"from":30,"to":40},
						{"from":40,"to":55},
						{"from":55}
					]
				}			
			}
		}
	}'

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"age_ranges": {
				"range": {
					"field": "age",
					"keyed": true,
					"ranges": [
						{"key":"young", "to": 30 },
						{"key":"quarter-aged","from":30,"to":40},
						{"key":"quarter-aged2","from":40,"to":55},
						{"key":"quarter-aged3","from":55}
					]
				}			
			}
		}
	}'


#### Multi-level Nested Aggregations

Find the average age of male and female customers

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"gender_bucker": {
				"terms": {
					"field": "gender"
				},
				"aggs": {
					"average_age": {
						"avg": {
							"field": "age"
						}
					}
				}			
			}
		}
	}'


#### The Filter and Filters Bucketing Aggregations

- the "filter" aggregation to filter results
- the "filters" aggregation to specify multiple filter matches

The average age of customers in the state of Texas

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"aggs": {
			"state": {
				"filter": {"term": {"state":"texas"}},
				"aggs": {
					"average_age": {
						"avg": {
							"field": "age"
						}
					}
				}			
			}
		}
	}'

	curl -XPOST 'localhost:9200/products/_search?pretty' -d'
	{
		"size": 0,
		"aggs": {
			"states": {
				"filters": {
					"filters": {
						"washington": {
							"match": {"state": "washington"}
						},
						"carolina": {
							"match": {"state": "carolina"}
						}
					}	
				}
			}
		}
	}'