---
title: Indices APIs
categories:
  - elk
tags:
  - index
abbrlink: ef78ba07
date: 2018-09-04 17:46:22
---



# Defines a Template

```sh
PUT _template/logstash
{
  "order" : 0,
  "index_patterns": [
      "logstash-*"
    ],
    "settings": {
      "index": {
        "number_of_shards": 2,
        "number_of_replicas": 1,
        "refresh_interval": "5s"
      }
    },
    "mappings": {
      "_default_": {
        "dynamic_templates": [
          {
            "message_field": {
              "path_match": "message",
              "match_mapping_type": "string",
              "mapping": {
                "type": "text",
                "norms": false
              }
            }
          },
          {
            "string_fields": {
              "match": "*",
              "match_mapping_type": "string",
              "mapping": {
                "type": "text",
                "norms": false,
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              }
            }
          }
        ],
        "properties": {
          "@timestamp": {
            "type": "date"
          },
          "@version": {
            "type": "keyword"
          },
          "geoip": {
            "dynamic": true,
            "properties": {
              "ip": {
                "type": "ip"
              },
              "location": {
                "type": "geo_point"
              },
              "latitude": {
                "type": "half_float"
              },
              "longitude": {
                "type": "half_float"
              }
            }
          }
        }
      }
    },
    "aliases": {}
}
```

> order 默认 0



<!-- more -->



# Deleting a Template

```sh
DELETE _template/logstash
```



# Getting templates

```sh
GET _template/logstash

# You can also match several templates by using wildcards like:
GET _template/logstash, logstash_1
```



# Template exists

```sh
HEAD _template/logstash
```



# Multiple Templates Matching

