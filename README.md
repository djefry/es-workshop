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
#### List all index
#### Create index
#### Delete index

## Analyzer
#### Standard analyzer
#### Smartcn analyzer

## Field Type


## Mapping
#### Check mapping
#### Create mapping
We can only add to current mapping, can't rename, delete if you want to rename or delete field you should recreate the mapping.

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
