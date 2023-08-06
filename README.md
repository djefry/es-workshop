## Basic Elasticsearch Workshop
In this workshop we will spun elasticsearch and kibana using docker then knowing the basic functionality of elasticsearch. \
Why we use docker because with docker we don't need to think about the installation depedency and many other stuff, docker just make it simple.

### Requirements
Some requirements that need to have, not should be exact you can always install the newest version but this is the condition of my machine:
- Docker (20.10) 
- docker-compose (1.25.0)

Docker Images:\
Basically you don't need to think much about this because when you run docker-compose up this image will automatically pull to your local machine
- elasticsearch:7.17.7
- kibana:7.17.7

### How to use
- clone this repository
- run `docker-compose up`

## Index Operation
#### List all index without header
```
GET /_cat/indices
```

#### List all index with detail
```
GET /_cat/indices?v
```

#### Create Index
```
PUT /article1
```

#### Check Index
```
GET /article1
```

#### Delete Index
```
DELETE /article1
```

#### Create index with settings
```
PUT /article2/_settings
{
  "analysis": {
    "filter": {
      "autocomplete_filter": {
        "type": "edge_ngram",
        "min_gram": "1",
        "max_gram": "50"
      }
    },
    "analyzer": {
      "autocomplete_analyzer": {
        "filter": [
          "lowercase",
          "edgengram_filter"
        ],
        "type": "custom",
        "tokenizer": "whitespace"
      }
    }
  }
}
```

## Analyzer
#### Character Filter
```
PUT /character_filter_test
{
  "settings": {
    "analysis": {
      "char_filter": {
        "my_custom_char_filter": {
          "type": "mapping",
          "mappings": ["&=>and"]
        }
      },
      "analyzer": {
        "my_custom_analyzer": {
          "tokenizer": "standard",
          "char_filter": ["my_custom_char_filter"]
        }
      }
    }
  }
}
```

#### Test Character Filter
```
POST /character_filter_test/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Apples & Bananas"
}
```

#### Tokenizer
```
PUT /tokenizer_test
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_custom_tokenizer": {
          "type": "whitespace"
        }
      }
    }
  }
}
```

#### Test Tokenizer
```
POST /tokenizer_test/_analyze
{
  "tokenizer": "my_custom_tokenizer",
  "text": "Apples & Bananas"
}
```

#### Token Filter
```
PUT /token_filter_test
{
  "settings": {
    "analysis": {
      "filter": {
        "my_custom_token_filter": {
          "type": "lowercase"
        }
      },
      "analyzer": {
        "my_custom_analyzer": {
          "tokenizer": "standard",
          "filter": ["my_custom_token_filter"]
        }
      }
    }
  }
}
```

#### Test token filter
```
POST /token_filter_test/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "This is a SAMPLE & Text"
}
```

#### Analyzer Test
```
POST /_analyze
{
  "analyzer": "standard",
  "text": "This is a SAMPLE & Text"
}

POST /_analyze
{
  "analyzer": "standard",
  "text": "你好吗 - nihaoma"
}

POST /_analyze
{
  "analyzer": "smartcn",
  "text": "你好吗 - nihaoma"
}

POST /_analyze
{
  "analyzer": "smartcn",
  "text": "新冠肺炎疫情的现状 - current state of covid pandemic"
}

POST /_analyze
{
  "analyzer": "standard",
  "text": "新冠肺炎疫情的现状 - current state of covid pandemic"
}
```

#### Autocomplete analyzer test
```
POST /article2/_analyze
{
  "analyzer": "autocomplete_analyzer",
  "text": "新冠肺炎疫情的现状 - current state of covid pandemic"
}
```

## Install Custom Analyzer
```
docker ps  # check the available running docker
docker exec -it docker_elasticsearch_1 bash # the second from the last is the name of container
# Install
/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-smartcn
# Uninstall
/usr/share/elasticsearch/bin/elasticsearch-plugin remove analysis-smartcn
```
[List of available plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis.html)

### List plugins installed
```
GET /_cat/plugins
```

## Field Type
- Keyword \
Keyword is non analyzed type of data so elasticsearch will keep as is in the index (usefull for agregation and exact match)
- Text \
Text data will always analyzed if we not define the analyzer it will user standard analyzer
- Number \
For number data type
- Boolean \
For true or false data type
- Datetime \
For datetime data type

[List of supported datatype](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

## Mapping
We can only add to current mapping, can't rename, delete if you want to rename or delete field you should recreate the mapping.
### Update index with mappings
```
PUT /article2/_mapping
{
  "properties": {
    "title": {
      "analyzer": "autocomplete_analyzer",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      },
      "type": "text"
    },
    "descripton": {
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      },
      "type": "text"
    },
    "likes": {
      "type": "long"
    }
  }
}
```

### Get mappings
```
GET /article2/_mapping
```

### Create Index with settings and mapping
```
PUT /article2
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": "1",
          "max_gram": "50"
        }
      },
      "analyzer": {
        "autocomplete_analyzer": {
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ],
          "type": "custom",
          "tokenizer": "whitespace"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "analyzer": "autocomplete_analyzer",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        },
        "type": "text"
      },
      "descripton": {
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        },
        "type": "text"
      },
      "likes": {
        "type": "long"
      }
    }
  }
}
```

### Check article2 mapping and setting
```
GET /article2
```

## Indexing Operation
### Add data to index 
#### Automatically creating the document id also we can automatically create the index
```
POST article2/_doc
{
  "title": "World War III Films",
  "description": "Film that shows the situation of world war III",
  "likes": 1000
}

POST article2/_doc
{
  "title": "World War III Films",
  "description": "Film that shows the situation of world war III"
}
```

#### Specify manually the document id
```
POST article2/_doc/1
{
  "title": "The Witcher III",
  "description": "Game that use by most of reviewer to test the GPU or gaming performance",
  "likes": 500
}

PUT article2/_doc/2
{
  "title": "The Witcher III",
  "description": "Game that use by most of reviewer to test the GPU or gaming performance"
}
```

#### Get specific document
```
GET /article2/_doc/<id_article>

GET article2/_doc/1

GET article2/_doc/2
```

### Update specific document
#### Update with replace
```
PUT /article2/_doc/1
{
  "title": "Resident Evil",
  "description": "Another game that use by reviewer to test gamming performance"
}
```

#### Partial Update
```
POST article2/_update/1
{ 
  "doc": {
    "likes": 1200
  }
}
```

#### Delete Data
```
DELETE article2/_doc/1
```

## Search Operation
### Search without parameter
Elasticsearch will perform a search on all documents in the specified index and will return the top 10 results (default value for size parameter).
```
GET /article2/_search
```

### Exact Match Search
#### Exact match search
```
GET /article2/_search
{
  "query": {
    "term": {
      "title.keyword": "Resident Evil"
    }
  }
}
```

#### Exact match with lower case query
```
GET /article2/_search
{
  "query": {
    "term": {
      "title.keyword": "resident evil"
    }
  }
}
```

#### Exact match support lowercase search
```
GET /article2/_search
{
  "query": {
    "term": {
      "title.keyword": {
        "value": "resident evil",
        "case_insensitive": true
      }
    }
  }
}
```

### Matching Search
#### Dummy Data For Next Search
```
PUT /article2/_doc/3
{
  "title": "Primary Resident",
  "description": "Primary Resident for living in the city"
}

PUT /article2/_doc/4
{
  "title": "Resident Evil II",
  "description": "Successor of the resident evil 1"
}

PUT /article2/_doc/5
{
  "title": "Residual Agent",
  "description": "Film about the living residence in the city"
}

PUT /article2/_doc/6
{
  "title": "Resilience",
  "description": "Book that tells us how to build resilience"
}
```

#### Simple match query
```
GET /article2/_search
{
  "query": {
    "term": {
      "title": "resident"
    }
  }
}

GET /article2/_search
{
  "query": {
    "term": {
      "title": "Resident"
    }
  }
}

GET /article2/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Resident",
        "analyzer": "standard"
      }
    }
  }
}

GET /article2/_search
{
  "query": {
    "match": {
      "description": "game living"
    }
  }
}
```

#### Bollean matching query
```
GET /article2/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "description": "game living"
          }
        }
      ],
      "should": [
        {
          "match": {
            "description": "city"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "title": "evil"
          }
        }
      ]
    }
  }
}
```

#### Span First Query
```
GET /article2/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "resident"
          }
        }
      ]
    }
  }
}

GET /article2/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "resident"
          }
        }
      ],
      "should": [
        {
          "span_first": {
            "match": {
              "span_term": {
                "title": "resident"
              }
            },
            "end": 1
          }
        }
      ]
    }
  }
}
```

#### Exact pharse in text field
```
GET /article2/_search
{
  "query": {
    "match_phrase": {
      "title": "resident evil"
    }
  }
}
```

#### Auto complete
```
GET /article2/_search
{
  "query": {
    "match": {
      "title": "r"
    }
  }
}
```

#### Auto complete with span first
```
GET /article2/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "r"
          }
        }
      ],
      "should": [
        {
          "span_first": {
            "match": {
              "span_term": {
                "title": "r"
              }
            },
            "end": 2
          }
        }
      ]
    }
  }
}
```

#### Typo Correction
```
GET /article2/_search
{
  "query": {
    "fuzzy": {
      "description": {
        "value": "citi",
        "fuzziness": "AUTO"
      }
    }
  }
}
```
