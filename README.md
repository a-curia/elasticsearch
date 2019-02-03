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
