# ElasticSearch Autocomplete

ES index that will serve autocomplete needs with leveraging typos and errors.


### Create index
`PUT http://localhost:9201/music_bands`

Body:
```json
{
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 1,
        "analysis": {
            "analyzer": {
                "analyzer_autocomplete": {
                    "type": "custom",
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "filter_autocomplete"
                    ]
                }
            },
            "filter": {
                "filter_autocomplete": {
                    "type": "edge_ngram",
                    "min_gram": 2,
                    "max_gram": 16
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "title": {
                "type": "text",
                "analyzer": "analyzer_autocomplete",
                "search_analyzer": "standard"
            }
        }
    }
}
```

### Add documents

`POST http://localhost:9201/music_bands/_doc/{id}`

Body:
```json
{
    "title": "The Smiths" //Metallica, Megadeth, AC/DC, The Smiths, Sum 41,
}
```

### Search operation

`GET http://localhost:9201/music_bands/_search`

Body:

1. Example: `Sum 41`
```json
{
    "query": {
        "match": {
            "title": {
                "query": "suw 56",
                "analyzer": "analyzer_autocomplete"
            }
        }
    }
}
```

Result:

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.8672535,
        "hits": [
            {
                "_index": "music_bands",
                "_id": "6",
                "_score": 1.8672535,
                "_source": {
                    "title": "Sum 41"
                }
            }
        ]
    }
}
```

2. Example: `Metallica`

```json
{
    "query": {
        "match": {
            "title": {
                "query": "Metulika",
                "analyzer": "analyzer_autocomplete"
            }
        }
    }
}
```

Result: 

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.5616469,
        "hits": [
            {
                "_index": "music_bands",
                "_id": "1",
                "_score": 1.5616469,
                "_source": {
                    "title": "Metallica"
                }
            },
            {
                "_index": "music_bands",
                "_id": "2",
                "_score": 1.3132031,
                "_source": {
                    "title": "Megadeth"
                }
            }
        ]
    }
}
```
3. Example `AC\DC`

```json
{
    "query": {
        "match": {
            "title": {
                "query": "acdisi",
                "analyzer": "analyzer_autocomplete"
            }
        }
    }
}
```

Result: 
```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.8672535,
        "hits": [
            {
                "_index": "music_bands",
                "_id": "3",
                "_score": 1.8672535,
                "_source": {
                    "title": "AC/DC"
                }
            }
        ]
    }
}
```