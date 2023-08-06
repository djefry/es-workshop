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
### Keyword
Keyword is non analyzed type of data so elasticsearch will keep as is in the index (usefull for agregation and exact match)
### Text
Text data will always analyzed if we not define the analyzer it will user standard analyzer
### Number
For number data type
### Boolean
For true or false data type
### Datetime
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

## Index Operation
#### Auto create document key
```
POST article/_doc
{
  "title": "World War III Films",
  "description": "Film that shows the situation of world war III"
}
```
#### Replace data
```
PUT article/_doc/1
{
  "title": "The Witcher III",
  "description": "Game that use by most of reviewer to test the GPU or gaming performance"
}
```
#### Partial update data
```
POST article/_doc/1/_update
{
  "title": "Residen Evil",
  "description": "Another game that use by reviewer to test gamming performance"
}
```
## Check Index

## Search Operation
#### Exact Search
#### Matching Search
#### Autocomplete Search
#### Typo Correction
