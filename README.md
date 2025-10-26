## Quick prototype to learn Elastic Search
1. [Setup guide - Claude](https://claude.ai/share/b1e8e3b2-ec3b-4448-834b-0cf9ffbb9ff0)
2. [Detailed discussion - Claude](https://claude.ai/share/b1e8e3b2-ec3b-4448-834b-0cf9ffbb9ff0)

## Run locally

### Run containers
```bash
docker-compose up -d
```

### Open Kibana Dashboard - Devtools
```
http://localhost:5601/app/dev_tools#/console
```

### Queries

```
GET _cluster/health
```

Create an index - "books"
```
PUT /books
```

```
GET /books
```

```
GET _cat/indices?v
```

Insert a document
```
POST /books/_doc
{
  "title": "how to be an author?",
  "author": "C.W. Lewis",
  "year": 2001,
  "pages": 233
}
```

Bulk insert
```
POST /books/_bulk
{"index":{"_index":"books"}}
{"title":"Clean Code","author":"Robert Martin","year":2008,"pages":464}
{"index":{"_index":"books"}}
{"title":"Design Patterns","author":"Gang of Four","year":1994,"pages":395}
{"index":{"_index":"books"}}
{"title":"Refactoring","author":"Martin Fowler","year":1999,"pages":431}
```

Mappings - In ES mappings are the schema definitions for an index, specifying how documents and their fields are stored and indexed
```
GET /books/_mapping
```

Query all documents
```
GET /books/_search
{
  "query": {
    "match_all": {}
  }
}
```

Query books matching the title
```
GET /books/_search
{
  "query": {
    "match": {
      "title": "pragmatic"
    }
  }
}
```

Group by on book title. Note that field is `title.keyword` and not `title`
```
GET /books/_search
{
  "size": 0,
  "aggs": {
    "unique_titles": {
      "terms": {
        "field": "title.keyword"
      }
    }
  }
}
```

Query books **containing** word `the` and sorting by `title`
```
GET /books/_search
{
  "query": {
    "match": {
      "title": "the"
    }
  },
  "sort": [
    {
      "title.keyword": "asc"
    }
  ]
}
```

Searching on multiple fields with boosting (here title is boosted 10X)
```
GET /books/_search
{
  "query": {
    "multi_match": {
      "query": "Sindbad",
      "fields": ["author", "title^10"]
    }
  }
}
```

Using `filter` which just filters out the search results without affecting relevancy score
```
GET /books/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "guide"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "year": {
              "gte": 2005
            }
          }
        }
      ]
    }
  }
}
```

Running same query but using `must` which affects relevancy score but scores poorly for those not meeting the conditions of the query.
```
GET /books/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "guide"
          }
        },
        {
          "range": {
            "year": {
              "lte": 2005
            }
          }
        }
      ]
    }
  }
}
```

Fuzzy search
```
GET /books/_search
{
  "query": {
    "match": {
      "title": {
        "query": "guide",
        "fuzziness": "AUTO"
      }
      
    }
  }
}
```

Aggregation: finding the book w/ max pages for each decade
```
GET /books/_search
{
  "size": 0,
  "aggs": {
    "books_by_decade": {
      "histogram": {
        "field": "year",
        "interval": 10
      },
      "aggs": {
        "max_pages": {
          "max": {
            "field": "pages"
          }
        }
      }
    }
  }
}

```
