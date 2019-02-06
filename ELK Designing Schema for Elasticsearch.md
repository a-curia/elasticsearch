# Designing Schema for Elasticsearch

curl -XGET 'localhost:9200/books/_mappings?&pretty'

**books1.json**

	{
		"mappings": {
			"fiction": {
				"_source": {
					"enabled": false
				}
			}
		}
	}

**books2.json**

	{
		"properties": {
			"pages": {
				"type": "integer"
			}
		}
	}

curl -XPUT 'localhost:9200/my_index/samples/1?&pretty' -H 'Content-Type: application/json' -d'{ "count": "5" }'



curl -XPUT 'localhost:9200/my_index1&pretty' -H 'Content-Type: application/json' -d'{ "mappings": { "my_type": { "numeric_detection":true } } }'

curl -XPUT 'localhost:9200/my_index1/my_type/1&pretty' -H 'Content-Type: application/json' -d'{ "count": "5" }'

curl -XGET 'localhost:9200/my_index1/my_type/_mapping?&pretty'


### Explicit Mappings

custom1.json
	
	{
		"settings": {
			"number_of_shards": 1,
			"number_of_replicas": 0
		},
		"dynamic": "strinct",
		"mappings": {
			"fiction": {
				"properties": {
					"title": { "type": "text" },
					"author": { "type": "text" }
					"available": { "type": "boolean" }
					"pages": { "type": "integer" }
					"cost": { "type": "float" },
					"published": {
						"type": "date",
						"format": "YYYY-MM-DD"
					}
				}
			}
		}
	}

"dynamic" can be:
true: new fields are accepted and idexed(default)
false: new fields are accepted but ignored and not indexed
strict: new fields not accepted, exception is thrown

custom2.json

	{
		"mappings": {
			"fiction": {
				"_source": {
					"enabled": false
				},
				"_all": {
					"enabled": false
				}
				"properties": {
					"title": { "type": "text" },
					"author": { "type": "text" }
					"available": { "type": "boolean" }
					"pages": { "type": "integer" }
					"cost": { "type": "float" },
					"published": {
						"type": "date",
						"format": "YYYY-MM-DD"
					}
				}
			}
		}
	}


_all : concatenated contents of all fields into one big string using space as a delimiter - disabled in v6 by default; Enables searching within all fields in a document without knowing which fields holds the information

### Mapped vs. Unmapped Fields

the effect of mappings on how a filed is indexed and searched

curl -XPOST 'localhost:9200/products/discounts/1&pretty' -H 'Content-Type: application/json' -d'{ "name": "Multi grain bread", "cost": "55", "discount": "20" }'

curl -XGET 'localhost:9200/products/_mapping?&pretty'

this query works only on full-text fields

curl -XGET 'localhost:9200/products/discounts/_search?pretty' -H 'Content-Type: application/json' -d'{ "query": { "wildcard": {"discount": "2*" } } }'

**mapping1.json**

{
	"mappings": {
		"discounts": {
			"properties": {
				"name": {"type": "text"},
				"cost": {"type": "integer"},
				"discount": {"type": "integer"}
			}
		}
	}
}

curl -XPUT 'localhost:9200/products_copy&pretty' -H 'Content-Type: application/json' -d @mapping1.json
curl -XPOST 'localhost:9200/products_copy/discounts/1&pretty' -H 'Content-Type: application/json' -d'{ "name": "Multi grain bread", "cost": "55", "discount": "20" }'

this should not work anymore

curl -XGET 'localhost:9200/products/discounts/_search?pretty' -H 'Content-Type: application/json' -d'{ "query": { "wildcard": {"discount": "2*" } } }'

create the blog index

curl -XPOST 'localhost:9200/blog/posts/1&pretty' -H 'Content-Type: application/json' -d'{ "name": "Top 10 Ocean Mysteries", "date": "10-06-2017" }' 

curl -XGET 'localhost:9200/blog/_mapping?&pretty'

date has been detected as a string field because it's not respoectig the format YYYY-mm-dd, but we can do some partial matches

**mappin2.json**

	{
		"mappings": {
			"posts": {
				"properties": {
					"name": {"type": "text"},
					"date": {
						"type": "date",
						"format": "DD-MM-YYYY"
					}
				}
			}
		}
	}

curl -XPUT 'localhost:9200/blog_copy&pretty' -H 'Content-Type: application/json' -d @mappin2.json

curl -XPOST 'localhost:9200/blog_copy/posts/1&pretty' -H 'Content-Type: application/json' -d'{ "name": "Top 10 Ocean Mysteries", "date": "10-06-2017" }' 

curl -XGET 'localhost:9200/blog_copy/_mapping?&pretty'

curl -XGET 'localhost:9200/blog_copy/posts/_search?pretty' -H 'Content-Type: application/json' -d'{ "query": { "match": {"date": "18-06-2017" } } }'

partial matching will not work

### Dynamic Templates for Custom Mapping

match_mapping_type : match those fields which have the type specified

* matches all data types
*  

**mappings1.json**

	{
		"mappings": {
			"type_one": {
				"dynamic_templates": [
					{
						"integers": {
							"match_mapping_types": "long",
							"mapping": {
								"type": "integer"
							}
						}	
					},
					{
						"strings": {
							"match_mapping_types": "string",
							"mapping": {
								"type": "text"
							}
						}	
					}
				]
			}
		}
	}

multiple templates can be specified which are matched in order; if a later template has the same name the previous template will be replaced


match - select fields which match the specified pattern

unmatch - exclude those fields which have been previously matched using the "match" pattern

**mappings3.json**

	{
		"mappings": {
			"type_one": {
				"dynamic_templates": [
					{
						"longs_as_strings": {
							"match_mapping_types": "string",
							"match": "long_*",
							"unmatch": "*_text",
							"mapping": {
								"type": "long"
							}
						}	
					}
				]
			}
		}
	}

path_match and path_unmatch : work exactly like the "match" and "unmatch" keywords, except that they operate on the full name

**mappings4.json**

	{
		"mappings": {
			"type_three": {
				"dynamic_templates": [
					{
						"full_name": {
							"path_match": "name.*",
							"path_unmatch": "*.middle*",
							"mapping": {
								"type": "text",
								"copy_to": "full_name"
							}
						}	
					}
				]
			}
		}
	}


	"name": {
		"first": "AAAA",
		"middle": "sdsfsdF",
		"last": "dsfsdf
	}


### Analysers
