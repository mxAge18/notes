### Elasticsearch检索数据

搜索查询，或称查询，是对Elasticsearch数据流或索引中数据信息的请求。

你可以把查询看作是一个问题，用Elasticsearch理解的方式来写。根据你的数据，你可以用查询来获取问题的答案，比如说

- 我的服务器上哪些进程的响应时间超过500毫秒？
- 我的网络上有哪些用户在上周运行了regsvr32.exe？
- 我的网站上有哪些页面包含一个特定的单词或短语？

一个搜索由一个或多个查询组成，这些查询被组合起来并被发送到Elasticsearch。匹配搜索查询的文件会在响应的命中率或搜索结果中返回。

一个搜索也可能包含额外的信息，用于更好地处理其查询。例如，一个搜索可能被限制在一个特定的索引，或者只返回特定数量的结果。

## Run a search

You can use the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) to search and [aggregate](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) data stored in Elasticsearch data streams or indices. The API’s `query` request body parameter accepts queries written in [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

The following request searches `my-index-000001` using a [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) query. This query matches documents with a `user.id` value of `kimchy`.

```console
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

The API response returns the top 10 documents matching the query in the `hits.hits` property.

返回前十个结果

```console-result
{
  "took": 5,
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
    "max_score": 1.3862942,
    "hits": [
      {
        "_index": "my-index-000001",
        "_type": "_doc",
        "_id": "kxWFcnMByiguvud1Z8vC",
        "_score": 1.3862942,
        "_source": {
          "@timestamp": "2099-11-15T14:12:12",
          "http": {
            "request": {
              "method": "get"
            },
            "response": {
              "bytes": 1070000,
              "status_code": 200
            },
            "version": "1.1"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "source": {
            "ip": "127.0.0.1"
          },
          "user": {
            "id": "kimchy"
          }
        }
      }
    ]
  }
}
```



### Define fields that exist only in a query

Instead of indexing your data and then searching it, you can define [runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/runtime-search-request.html) that only exist as part of your search query. You specify a `runtime_mappings` section in your search request to define the runtime field, which can optionally include a Painless script.

For example, the following query defines a runtime field called `day_of_week`. The included script calculates the day of the week based on the value of the `@timestamp` field, and uses `emit` to return the calculated value.

The query also includes a [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html) that operates on `day_of_week`.

你可以定义[Runtime字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/runtime-search-request.html)，而不是对你的数据进行索引，然后再进行搜索，这些字段只作为搜索查询的一部分存在。你在搜索请求中指定一个`runtime_mappings`部分来定义运行时字段，其中可以选择包括一个Painless脚本。

例如，下面的查询定义了一个名为 "day_of_week "的运行时间字段。包含的脚本根据`@timestamp`字段的值来计算星期几，并使用`emit`来返回计算值。

该查询还包括一个对`day_of_week`进行操作的[term aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html)

```console
GET /my-index-000001/_search
{
  "runtime_mappings": {
    "day_of_week": {
      "type": "keyword",
      "script": {
        "source":
        """emit(doc['@timestamp'].value.dayOfWeekEnum
        .getDisplayName(TextStyle.FULL, Locale.ROOT))"""
      }
    }
  },
  "aggs": {
    "day_of_week": {
      "terms": {
        "field": "day_of_week"
      }
    }
  }
}
```

The response includes an aggregation based on the `day_of_week` runtime field. Under `buckets` is a `key` with a value of `Sunday`. The query dynamically calculated this value based on the script defined in the `day_of_week` runtime field without ever indexing the field.

响应包括一个基于 "day_of_week "Runtime字段的聚合。在`buckets`下有一个`key`，其值为`Sunday`。查询根据`day_of_week`运行时字段中定义的脚本动态地计算出这个值，而没有对该字段进行索引。

```console-result
{
  ...
  ***
  "aggregations" : {
    "day_of_week" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Sunday",
          "doc_count" : 5
        }
      ]
    }
  }
}
```



### 常见的搜索选项

可用如下iptions自定义searches

#### **Query DSL**

[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)支持各种查询类型，你可以混合和匹配以获得你想要的结果。查询类型包括。

- [布尔型](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)和其他[复合型查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/compound-queries.html)，它使你能够根据多个标准组合查询和匹配结果
- [Term-level查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html)，用于过滤和寻找精确的匹配结果
- [全文查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html)，这在搜索引擎中是常用的。
- [地理](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html)和[空间查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/shape-queries.html)

#### **聚合查询**

你可以使用[搜索聚合](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)来获取你的搜索结果的统计数据和其他分析。汇总可以帮助你回答以下问题。

- 我的服务器的平均响应时间是多少？
- 我的网络上被用户点击的最多的IP地址是什么？
- 按客户分类的总交易收入是多少？

#### **搜索多个数据流和索引**。

你可以使用逗号分隔的值和类似grep的索引模式，在同一个请求中搜索多个数据流和索引。你甚至可以提升特定索引的搜索结果。参见[*搜索多个数据流和索引*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multiple-indices.html)。

#### **分页搜索结果**

默认情况下，搜索只返回前10个匹配的点击。要检索更多或更少的文件，请参阅 [*分页搜索结果*](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)。

#### **检索选定的字段**

搜索响应的`hit.hit`属性包括每个命中的完整文档[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)。要想只检索`_source`或其他字段的子集，请参阅[*检索选定字段*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html)。

#### **排序搜索结果**

默认情况下，搜索结果是按`_score`排序的，这是一个[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores)，用来衡量每个文档与查询的匹配程度。要自定义这些分数的计算，请使用[`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-score-query.html)查询。要按其他字段值对搜索结果进行排序，请参阅[*排序搜索结果*](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html)。

#### **运行一个异步搜索**。

Elasticsearch搜索被设计为在大量数据上快速运行，通常在几毫秒内返回结果。由于这个原因，搜索默认是*同步的。搜索请求在返回响应之前会等待完整的结果。

然而，对于跨[冻结索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/freeze-index-api.html)或[多个集群](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html)的搜索，完整的结果可能需要更长时间。

为了避免长时间的等待，你可以运行一个*异步*，或*async*的搜索。一个[异步搜索](https://www.elastic.co/guide/en/elasticsearch/reference/current/async-search-intro.html)可以让你现在检索一个长期运行的搜索的部分结果，并在以后获得完整的结果。

### 超时设置

默认情况下，搜索请求不会超时。在返回响应之前，请求会等待来自每个分片的完整结果。

虽然[async search](https://www.elastic.co/guide/en/elasticsearch/reference/current/async-search-intro.html)是为长期运行的搜索设计的，但你也可以使用`timeout`参数来指定你想等待每个分片完成的时间。每个分片在指定的时间段内收集点击率。如果在该时间段结束时，收集工作还没有完成，Elasticsearch只使用截至该时间点的累积点击率。搜索请求的总体延迟取决于搜索所需的分片数量和并发的分片请求数量。

```console
GET /my-index-000001/_search
{
  "timeout": "2s",
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

要为所有搜索请求设置集群范围内的默认超时，请使用[集群设置API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)配置`search.default_search_timeout`。如果请求中没有传递`timeout`参数，则使用这个全局超时时间。如果全局搜索超时在搜索请求完成之前就已经过期，则使用[task cancellation](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html#task-cancellation)取消该请求。`search.default_search_timeout`设置默认为`-1`（无超时）。

### 取消搜索

You can cancel a search request using the [task management API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html#task-cancellation). Elasticsearch also automatically cancels a search request when your client’s HTTP connection closes. We recommend you set up your client to close HTTP connections when a search request is aborted or times out.

你可以使用[任务管理API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html#task-cancellation)取消一个搜索请求。当你的客户端的HTTP连接关闭时，Elasticsearch也会自动取消一个搜索请求。我们建议你设置你的客户端，当搜索请求被中止或超时时，关闭HTTP连接。

### 追踪总命中率

一般来说，如果不访问所有的匹配，就不能准确地计算出总命中数，这对于匹配大量文档的查询来说是很昂贵的。`track_total_hits`参数允许你控制如何跟踪总命中数。考虑到经常有一个命中率的下限，如 "至少有10000个命中率"，默认设置为`10000'。这意味着请求将准确地计算总的点击量，最多为`10,000`次。如果你不需要某个阈值之后的准确命中数，这是一个很好的交易，可以加快搜索速度。

当设置为 "true "时，搜索响应将总是准确地跟踪符合查询的命中数（例如，当 "track_total_hits "设置为 "true "时，"total.relation "将总是等于 "eq"）。否则，搜索响应中的 "total "对象中返回的 "total.relation "将决定如何解释 "total.value"。`"gte "的值意味着`"total.value "是匹配查询的总命中率的下限，`"eq "的值表示`"total.value "是准确的计数。

```console
GET my-index-000001/_search
{
  "track_total_hits": true,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  }
}
```

... returns:

```console-result
{
  "_shards": ...
  "timed_out": false,
  "took": 100,
  "hits": {
    "max_score": 1.0,
    "total" : {
      "value": 2048,    
      "relation": "eq"  
    },
    "hits": ...
  }
}
```



|      | The total number of hits that match the query.    |
| ---- | ------------------------------------------------- |
|      | The count is accurate (e.g. `"eq"` means equals). |

It is also possible to set `track_total_hits` to an integer. For instance the following query will accurately track the total hit count that match the query up to 100 documents:

```console
GET my-index-000001/_search
{
  "track_total_hits": 100,
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/720.console) 

The `hits.total.relation` in the response will indicate if the value returned in `hits.total.value` is accurate (`"eq"`) or a lower bound of the total (`"gte"`).

For instance the following response:

```console-result
{
  "_shards": ...
  "timed_out": false,
  "took": 30,
  "hits": {
    "max_score": 1.0,
    "total": {
      "value": 42,         
      "relation": "eq"     
    },
    "hits": ...
  }
}
```



|      | 42 documents match the query       |
| ---- | ---------------------------------- |
|      | and the count is accurate (`"eq"`) |

... indicates that the number of hits returned in the `total` is accurate.

If the total number of hits that match the query is greater than the value set in `track_total_hits`, the total hits in the response will indicate that the returned value is a lower bound:

```console-result
{
  "_shards": ...
  "hits": {
    "max_score": 1.0,
    "total": {
      "value": 100,         
      "relation": "gte"     
    },
    "hits": ...
  }
}
```



|      | There are at least 100 documents that match the query |
| ---- | ----------------------------------------------------- |
|      | This is a lower bound (`"gte"`).                      |

If you don’t need to track the total number of hits at all you can improve query times by setting this option to `false`:

```console
GET my-index-000001/_search
{
  "track_total_hits": false,
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/721.console) 

... returns:

```console-result
{
  "_shards": ...
  "timed_out": false,
  "took": 10,
  "hits": {             
    "max_score": 1.0,
    "hits": ...
  }
}
```



|      | The total number of hits is unknown. |
| ---- | ------------------------------------ |
|      |                                      |

Finally you can force an accurate count by setting `"track_total_hits"` to `true` in the request.

###快速检查匹配的文档

如果你只想知道是否有任何与特定查询相匹配的文档，你可以把`size`设置为`0`，表示我们对搜索结果不感兴趣。你也可以把`terminate_after`设置为`1`，表示只要找到第一个匹配的文档，就可以终止查询的执行（每个分片）

```console
GET /_search?q=user.id:elkbee&size=0&terminate_after=1
```

`terminate_after` is always applied **after** the [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/current/filter-search-results.html#post-filter) and stops the query as well as the aggregation executions when enough hits have been collected on the shard. Though the doc count on aggregations may not reflect the `hits.total` in the response since aggregations are applied **before** the post filtering.

The response will not contain any hits as the `size` was set to `0`. The `hits.total` will be either equal to `0`, indicating that there were no matching documents, or greater than `0` meaning that there were at least as many documents matching the query when it was early terminated. Also if the query was terminated early, the `terminated_early` flag will be set to `true` in the response.

`terminate_after`总是在[`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/current/filter-search-results.html#post-filter)之后应用，并在分片上收集到足够多的命中时停止查询和聚合的执行。虽然聚合的文档计数可能不反映响应中的`hits.total'，因为聚合是在过滤之前应用的。

响应将不包含任何命中，因为`size`被设置为`0`。hits.total "将等于 "0"，表示没有匹配的文档，或者大于 "0"，表示当查询被提前终止时，至少有相同数量的文档匹配。另外，如果查询提前结束，`terminated_early'标志将在响应中被设置为`true'。

```console-result
{
  "took": 3,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

响应中的 "took "时间包含该请求的处理时间，从节点收到查询后迅速开始，直到所有搜索相关工作完成，并在上述JSON返回给客户端之前。这意味着它包括在线程池中等待的时间，在整个集群中执行分布式搜索和收集所有结果的时间。

## 折叠搜索结果

You can use the `collapse` parameter to collapse search results based on field values. The collapsing is done by selecting only the top sorted document per collapse key.

For example, the following search collapses results by `user.id` and sorts them by `http.response.bytes`.

你可以使用`collapse`参数来折叠基于字段值的搜索结果。折叠是通过每个折叠键只选择排序最靠前的文件来完成的。

例如，下面的搜索按`user.id`折叠结果，并按`http.response.bytes`排序。

```console
GET my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id"   #1      
  },
  "sort": [
    {
      "http.response.bytes": { #2
        "order": "desc"
      }
    }
  ],
  "from": 0         #3           
}
```

| Collapse the result set using the `user.id` field  |
| -------------------------------------------------- |
| #2 Sort the results by `http.response.bytes`       |
| #3 Define the offset of the first collapsed result |

> 响应中的总命中数表示没有折叠的匹配文件的数量。不同组的总数量是未知的。

The field used for collapsing must be a single valued [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html) or [`numeric`](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html) field with [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) activated.

折叠只适用于顶级点击，不影响聚合。

### 展开折叠的结果

也可以用inner_hits选项来展开每个折叠的top hits。（通过inner_hits字段，把折叠的结果放到innter_hits里面。

- inner_hits中的参数有：
  - name
  - from
  - size
  - Sort

```
ET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id",                       
    "inner_hits": {
      "name": "most_recent",                  
      "size": 5,                              
      "sort": [ { "@timestamp": "desc" } ]    
    },
    "max_concurrent_group_searches": 4        
  },
  "sort": [
    {
      "http.response.bytes": {
        "order": "desc"
      }
    }
  ]
}
```

- 使用user.id字段折叠结果集
- 响应中用于内部命中部分的名称
- 每个折叠键要检索的inner_hits的数量
- 如何对每个组内的文件进行排序
- 每组检索inner_hits时允许的并发请求的数量

关于支持的选项的完整列表和响应的格式，见inner hits。

它也可以为每个折叠的命中请求多个inner_hits。当你想得到多个折叠命中的表示时，这可能是有用的。

```console
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id",                   
    "inner_hits": [
      {
        "name": "largest_responses",      
        "size": 3,
        "sort": [
          {
            "http.response.bytes": {
              "order": "desc"
            }
          }
        ]
      },
      {
        "name": "most_recent",             
        "size": 3,
        "sort": [
          {
            "@timestamp": {
              "order": "desc"
            }
          }
        ]
      }
    ]
  },
  "sort": [
    "http.response.bytes"
  ]
}
```

组的扩展是通过为每个`inner_hit`请求发送一个额外的查询来完成的，以获得响应中返回的每个折叠的命中。如果你有太多的组或`inner_hit`请求，这可能会大大降低你的搜索速度。

`max_concurrent_group_searches`请求参数可以用来控制这个阶段允许的最大并发搜索数量。默认的是基于数据节点的数量和默认的搜索线程池大小。

> `collapse` cannot be used in conjunction with [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/paginate-search-results.html#scroll-search-results) or [rescore](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/filter-search-results.html#rescore).

###`search_after`与Collapse共同使用

字段折叠可以与[`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/paginate-search-results.html#search-after)参数共同使用。

只有在对同一字段进行排序和折叠时，才支持使用`search_after'。二次排序也是不允许的。例如，我们可以对`user.id`进行折叠和排序，同时使用`search_after`对结果进行分页。

```console
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id"
  },
  "sort": [ "user.id" ],
  "search_after": ["dd5ce1ad"]
}
```

### 二级折叠

A second level of collapsing is also supported and is applied to `inner_hits`.

For example, the following search collapses results by `geo.country_name`. Within each `geo.country_name`, inner hits are collapsed by `user.id`.

>  第二级折叠不能再使用inner_hits`.

```console
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "geo.country_name",
    "inner_hits": {
      "name": "by_location",
      "collapse": { "field": "user.id" },
      "size": 3
    }
  }
}
```

```console-result
{
  "hits" : {
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "oX9uXXoB0da05OCR3adK",
        "_score" : 0.5753642,
        "_source" : {
          "@timestamp" : "2099-11-15T14:12:12",
          "geo" : {
            "country_name" : "Amsterdam"
          },
          "http" : {
            "request" : {
              "method" : "get"
            },
            "response" : {
              "bytes" : 1070000,
              "status_code" : 200
            },
            "version" : "1.1"
          },
          "message" : "GET /search HTTP/1.1 200 1070000",
          "source" : {
            "ip" : "127.0.0.1"
          },
          "user" : {
            "id" : "kimchy"
          }
        },
        "fields" : {
          "geo.country_name" : [
            "Amsterdam"
          ]
        },
        "inner_hits" : {
          "by_location" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : null,
              "hits" : [
                {
                  "_index" : "my-index-000001",
                  "_type" : "_doc",
                  "_id" : "oX9uXXoB0da05OCR3adK",
                  "_score" : 0.5753642,
                  "_source" : {
                    "@timestamp" : "2099-11-15T14:12:12",
                    "geo" : {
                      "country_name" : "Amsterdam"
                    },
                    "http" : {
                      "request" : {
                        "method" : "get"
                      },
                      "response" : {
                        "bytes" : 1070000,
                        "status_code" : 200
                      },
                      "version" : "1.1"
                    },
                    "message" : "GET /search HTTP/1.1 200 1070000",
                    "source" : {
                      "ip" : "127.0.0.1"
                    },
                    "user" : {
                      "id" : "kimchy"
                    }
                  },
                  "fields" : {
                    "user.id" : [
                      "kimchy"
                    ]
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

## 过滤搜索结果

有两种方式过滤搜索结果：

- 用带`filter`从句的`boolean`查询; 搜索结果和聚合都应用

- 用搜索APIs的`post_filter`参数；只能对搜索应用，聚合不可用. You can use a post filter to calculate aggregations based on a broader result set, and then further narrow the results.

  You can also rescore hits after the post filter to improve relevance and reorder results.

### Post filter

用`post_filter`过滤搜索结果，结果是聚合计算后的结果，对聚合结果没有影响。

例如，你正在销售具有以下属性的衬衫：

```console
PUT /shirts
{
  "mappings": {
    "properties": {
      "brand": { "type": "keyword"},
      "color": { "type": "keyword"},
      "model": { "type": "keyword"}
    }
  }
}

PUT /shirts/_doc/1?refresh
{
  "brand": "gucci",
  "color": "red",
  "model": "slim"
}
```

想象一下，一个用户指定了两个过滤器。

颜色：红色和品牌：Gucci。

`color:red` and `brand:gucci`. You only want to show them red shirts made by Gucci in the search results. Normally you would do this with a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/query-dsl-bool-query.html):

```
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"color": "red"}}
          ,
        {"term": {"brand": "gucci"}}
      ]
    }
  }
}
```

然而，你也想使用分面导航来显示一个用户可以点击的其他选项的列表。也许你有一个model字段，可以让用户将他们的搜索结果限制在红色的Gucci T恤或礼服衬衫上。

This can be done with a [`terms` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/search-aggregations-bucket-terms-aggregation.html):

```console
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" } 
    }
  }
}
```

但也许你还想告诉用户有多少件其他颜色的Gucci衬衫。如果你只是在颜色字段上添加一个术语聚合，你将只得到红色，因为你的查询只返回Gucci的红色衬衫。

相反，你想在聚合过程中包括所有颜色的衬衫，然后只对搜索结果应用颜色过滤器。这就是post_filter的目的。

```console
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } 
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } 
    },
    "color_red": {
      "filter": {
        "term": { "color": "red" } 
      },
      "aggs": {
        "models": {
          "terms": { "field": "model" } 
        }
      }
    }
  },
  "post_filter": { 
    "term": { "color": "red" }
  }
}
```

|      | The main query now finds all shirts by Gucci, regardless of color. |
| ---- | ------------------------------------------------------------ |
|      | The `colors` agg returns popular colors for shirts by Gucci. |
|      | The `color_red` agg limits the `models` sub-aggregation to **red** Gucci shirts. |
|      | Finally, the `post_filter` removes colors other than red from the search `hits` |

### Rescore过滤的搜索结果

Rescoring can help to improve precision by reordering just the top (eg 100 - 500) documents returned by the [`query`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/search-search.html#request-body-search-query) and [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/filter-search-results.html#post-filter) phases, using a secondary (usually more costly) algorithm, instead of applying the costly algorithm to all documents in the index.

重新排序可以帮助提高精确性，只对[`query`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/search-search.html#request-body-search-query)和[`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/filter-search-results.html#post-filter)阶段返回的上级（例如100-500）文档进行重新排序，使用二级（通常更昂贵）算法，而不是对索引中的所有文档应用昂贵的算法。

A `rescore` request is executed on each shard before it returns its results to be sorted by the node handling the overall search request.

Currently the rescore API has only one implementation: the query rescorer, which uses a query to tweak the scoring. In the future, alternative rescorers may be made available, for example, a pair-wise rescorer.

在每个分片上执行 "rescore "请求，然后再返回其结果，由处理整体搜索请求的节点进行排序。

目前rescore API只有一个实现：查询rescorer，它使用查询来调整评分。在未来，可能会提供其他的rescorer，例如，一对一的rescorer。

>  An error will be thrown if an explicit [`sort`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/sort-search-results.html) (other than `_score` in descending order) is provided with a `rescore` query.
>
> 如果一个明确的[`sort`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/sort-search-results.html)，将产生一个错误。(除了降序的"_score")与 "rescore "查询一起提供。

>  when exposing pagination to your users, you should not change `window_size` as you step through each page (by passing different `from` values) since that can alter the top hits causing results to confusingly shift as the user steps through pages.
>
> 当向用户展示分页时，你不应该在浏览每一页时改变`window_size'（通过传递不同的`from'值），因为这可能会改变最高命中率，导致用户在浏览页面时，结果会发生混乱。



#### Query rescorer

The query rescorer executes a second query only on the Top-K results returned by the [`query`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/search-search.html#request-body-search-query) and [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/filter-search-results.html#post-filter) phases. The number of docs which will be examined on each shard can be controlled by the `window_size` parameter, which defaults to 10.

By default the scores from the original query and the rescore query are combined linearly to produce the final `_score` for each document. The relative importance of the original query and of the rescore query can be controlled with the `query_weight` and `rescore_query_weight` respectively. Both default to `1`.

查询重构器只对查询和post_filter阶段返回的Top-K结果执行第二次查询。每个分片上被检查的文档数量可以通过window_size参数来控制，该参数默认为10。

默认情况下，来自原始查询和重新评分查询的分数被线性地结合起来，产生每个文档的最终_分数。原始查询和重核查询的相对重要性可以分别用query_weight和rescore_query_weight来控制。两者都默认为1。

For example:

```console
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : {
      "window_size" : 50,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }
}
```

The way the scores are combined can be controlled with the `score_mode`:

| Score Mode | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `total`    | Add the original score and the rescore query score. The default. |
| `multiply` | Multiply the original score by the rescore query score. Useful for [`function query`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/query-dsl-function-score-query.html) rescores. |
| `avg`      | Average the original score and the rescore query score.      |
| `max`      | Take the max of original score and the rescore query score.  |
| `min`      | Take the min of the original score and the rescore query score. |

#### Multiple rescores

It is also possible to execute multiple rescores in sequence:

```console
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : [ {
      "window_size" : 100,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }, {
      "window_size" : 10,
      "query" : {
         "score_mode": "multiply",
         "rescore_query" : {
            "function_score" : {
               "script_score": {
                  "script": {
                    "source": "Math.log10(doc.count.value + 2)"
                  }
               }
            }
         }
      }
   } ]
}
```

The first one gets the results of the query then the second one gets the results of the first, etc. The second rescore will "see" the sorting done by the first rescore so it is possible to use a large window on the first rescore to pull documents into a smaller window for the second rescore.

第一个得到查询的结果，然后第二个得到第一个的结果，等等。第二个重核将 "看到 "第一个重核所做的排序，因此有可能在第一个重核上使用一个大窗口，将文件拉到第二个重核的一个小窗口。

## Highlighting

Highlighters enable you to get highlighted snippets from one or more fields in your search results so you can show users where the query matches are. When you request highlights, the response contains an additional `highlight` element for each search hit that includes the highlighted fields and the highlighted fragments.

> Note:Highlighters don’t reflect the boolean logic of a query when extracting terms to highlight. Thus, for some complex boolean queries (e.g nested boolean queries, queries using `minimum_should_match` etc.), parts of documents may be highlighted that don’t correspond to query matches.

Highlighting显示需要字段的内容，如果字段没有stored, 相关的字段将走_source中提取。

For example, to get highlights for the `content` field in each search hit using the default highlighter, include a `highlight` object in the request body that specifies the `content` field:

```console
GET /_search
{
  "query": {
    "match": { "content": "kimchy" }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

Elasticsearch supports three highlighters: `unified`, `plain`, and `fvh` (fast vector highlighter). You can specify the highlighter `type` you want to use for each field.

三种高亮器：`unified`, `plain`, and `fvh` (fast vector highlighter).

### Unified highlighter

The `unified` highlighter uses the Lucene Unified Highlighter. This highlighter breaks the text into sentences and uses the BM25 algorithm to score individual sentences as if they were documents in the corpus. It also supports accurate phrase and multi-term (fuzzy, prefix, regex) highlighting. This is the default highlighter.

使用Lucene的统一高亮程序。这个高亮器将文本分解成句子，并使用BM25算法对单个句子进行评分，就像它们是语料库中的文档一样。它还支持精确的短语和多词（模糊、前缀、重合）高亮。这是默认的高亮程序。

### Plain highlighter

The `plain` highlighter uses the standard Lucene highlighter. It attempts to reflect the query matching logic in terms of understanding word importance and any word positioning criteria in phrase queries.

普通高亮器使用标准的Lucene高亮器。它试图在理解词的重要性和短语查询中任何词的定位标准方面反映查询的匹配逻辑。

> warning 
>
> The `plain` highlighter works best for highlighting simple query matches in a single field. To accurately reflect query logic, it creates a tiny in-memory index and re-runs the original query criteria through Lucene’s query execution planner to get access to low-level match information for the current document. This is repeated for every field and every document that needs to be highlighted. If you want to highlight a lot of fields in a lot of documents with complex queries, we recommend using the `unified` highlighter on `postings` or `term_vector` fields.
>
> `plain`高亮器最适合于突出单个字段的简单查询匹配。为了准确反映查询逻辑，它创建了一个微小的内存索引，并通过Lucene的查询执行计划器重新运行原始查询条件，以获得当前文档的低级匹配信息。这个过程对每个字段和每个需要突出显示的文档都要重复。如果你想通过复杂的查询来突出显示很多文档中的很多字段，我们建议在`postings`或`term_vector`字段上使用`unified`突出显示。

### Fast vector highlighter

The `fvh` highlighter uses the Lucene Fast Vector highlighter. This highlighter can be used on fields with `term_vector` set to `with_positions_offsets` in the mapping. The fast vector highlighter:

- Can be customized with a [`boundary_scanner`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/highlighting.html#boundary-scanners).
- Requires setting `term_vector` to `with_positions_offsets` which increases the size of the index
- Can combine matches from multiple fields into one result. See `matched_fields`
- Can assign different weights to matches at different positions allowing for things like phrase matches being sorted above term matches when highlighting a Boosting Query that boosts phrase matches over term matches

The `fvh` highlighter does not support span queries. If you need support for span queries, try an alternative highlighter, such as the `unified` highlighter.

`fvh`高亮器使用Lucene快速矢量高亮器。这个高亮器可以用于映射中`term_vector`设置为`with_positions_offsets`的字段。快速向量高亮程序。

- 可以使用[`boundary_scanner`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/highlighting.html#boundary-scanners)来定制。
- 需要将`term_vector`设置为`with_positions_offsets`，这将增加索引的大小。
- 可以将多个字段的匹配结果合并为一个结果。见`matched_fields`。
- 可以给不同位置的匹配结果分配不同的权重，例如，当高亮显示提升查询时，短语匹配结果被排序在术语匹配结果之上。

> `fvh`高亮显示器不支持跨度查询。如果你需要对跨度查询的支持，请尝试其他的高亮程序，如`unified`高亮程序。

### 偏移策略

为了从被查询的术语中创建有意义的搜索片段，高亮显示器需要知道原始文本中每个单词的起始和结束字符偏移量。这些偏移量可以从以下方面获得：

- The postings list.

   If `index_options` is set to `offsets` in the mapping, the `unified` highlighter uses this information to highlight documents without re-analyzing the text. 

  如果 "index_options "在映射中被设置为 "offsets"，那么 "统一 "高亮程序就会使用这些信息来高亮文档，而无需重新分析文本。

  It re-runs the original query directly on the postings and extracts the matching offsets from the index, limiting the collection to the highlighted documents. 

  它直接在帖子上重新运行原始查询，并从索引中提取匹配的偏移量，将集合限制在高亮的文档上。

  This is important if you have large fields because it doesn’t require reanalyzing the text to be highlighted. It also requires less disk space than using `term_vectors`.

  如果你有大字段，这很重要，因为它不需要重新分析要突出显示的文本。它也比使用`term_vectors`需要更少的磁盘空间。

- Term vectors. If `term_vector` information is provided by setting `term_vector` to `with_positions_offsets` in the mapping, the `unified` highlighter automatically uses the `term_vector` to highlight the field. 

  如果在映射中把 "term_vector "设置为 "with_positions_offsets "来提供 "term_vector "信息，那么 "unified "高亮程序会自动使用 "term_vector "来高亮字段。

  It’s fast especially for large fields (> `1MB`) and for highlighting multi-term queries like `prefix` or `wildcard` because it can access the dictionary of terms for each document. The `fvh` highlighter always uses term vectors.

  它的速度很快，特别是对于大的字段（> `1MB`）和突出显示多术语查询，如`前缀`或`通配符`，因为它可以访问每个文档的术语字典。`fvh`高亮程序总是使用术语向量。

- Plain highlighting. This mode is used by the `unified` when there is no other alternative.

  当没有其他选择时，这个模式被`unified`使用。

    It creates a tiny in-memory index and re-runs the original query criteria through Lucene’s query execution planner to get access to low-level match information on the current document. This is repeated for every field and every document that needs highlighting. The `plain` highlighter always uses plain highlighting.

  它在内存中创建一个微小的索引，并通过Lucene的查询执行计划器重新运行原始查询条件，以获得当前文档的低级匹配信息。这个过程对每个字段和每个需要高亮的文档都要重复进行。`plain`高亮程序总是使用普通高亮。

> Plain highlighting for large texts may require substantial amount of time and memory. To protect against this, the maximum number of text characters that will be analyzed has been limited to 1000000. This default limit can be changed for a particular index with the index setting [`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/index-modules.html#index-max-analyzed-offset).
>
> 对大型文本进行普通的高亮显示可能需要大量的时间和内存。为了防止这种情况，被分析的文本字符的最大数量被限制为1000000。这个默认的限制可以通过索引设置[`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/index-modules.html#index-max-analyzed-offset)为某个特定的索引改变。

### 高亮设置

可以全局设置，也可以在fields层面进行重写

#### **boundary_chars**

A string that contains each boundary character. Defaults to `.,!? \t\n`.

#### **boundary_max_scan**

How far to scan for boundary characters. Defaults to `20`.对边界字符的扫描范围。默认为`20'。

#### **boundary_scanner**

Specifies how to break the highlighted fragments: `chars`, `sentence`, or `word`. Only valid for the `unified` and `fvh` highlighters.  定义如何分割突出显示的片段，只对unified和fvh有用。

Defaults to `sentence` for the `unified` highlighter. 

Defaults to `chars` for the `fvh` highlighter.

- **`chars`**

  Use the characters specified by `boundary_chars` as highlighting boundaries. The `boundary_max_scan` setting controls how far to scan for boundary characters. Only valid for the `fvh` highlighter.

  使用由`boundary_chars`指定的字符作为高亮边界。boundary_max_scan "设置控制扫描边界字符的距离。

- **`sentence`**

  Break highlighted fragments at the next sentence boundary, as determined by Java’s [BreakIterator](https://docs.oracle.com/javase/8/docs/api/java/text/BreakIterator.html). You can specify the locale to use with `boundary_scanner_locale`.

  > When used with the `unified` highlighter, the `sentence` scanner splits sentences bigger than `fragment_size` at the first word boundary next to `fragment_size`. You can set `fragment_size` to 0 to never split any sentence.

- **`word`**

  Break highlighted fragments at the next word boundary, as determined by Java’s [BreakIterator](https://docs.oracle.com/javase/8/docs/api/java/text/BreakIterator.html). You can specify the locale to use with `boundary_scanner_locale`.

#### **boundary_scanner_locale**

控制使用哪种语言来搜索句子和词语的边界。

Controls which locale is used to search for sentence and word boundaries. This parameter takes a form of a language tag, e.g. `"en-US"`, `"fr-FR"`, `"ja-JP"`. More info can be found in the [Locale Language Tag](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#forLanguageTag-java.lang.String-) documentation. The default value is [Locale.ROOT](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT).

#### **encoder**

Indicates if the snippet should be HTML encoded: `default` (no encoding) or `html` (HTML-escape the snippet text and then insert the highlighting tags)

标识片段是否要进行HTML编码：默认为no， 可改为html

#### **fields**

Specifies the fields to retrieve highlights for. You can use wildcards to specify fields. For example, you could specify `comment_*` to get highlights for all [text](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/text.html) and [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/keyword.html) fields that start with `comment_`.

指定检索高亮的字段，可以用通配符来定义字段。例如：可以用comment_*可以把所有text和关键字中以comment___的字段高亮

> Only text and keyword fields are highlighted when you use wildcards. If you use a custom mapper and want to highlight on a field anyway, you must explicitly specify that field name.

#### **force_source**

Highlight based on the source even if the field is stored separately. Defaults to `false`. 即时字段是单独存储的，高亮基于source显示，默认false

#### **fragmenter**

Specifies how text should be broken up in highlight snippets: `simple` or `span`. Only valid for the `plain` highlighter. Defaults to `span`.

指定文本应如何在高亮片段中被分割开来。`simple`或`span`。只对`plain`高亮显示有效。默认为 "span"。

- **`simple`**

  Breaks up text into same-sized fragments.

- **`span`**

  Breaks up text into same-sized fragments, but tries to avoid breaking up text between highlighted terms. This is helpful when you’re querying for phrases. Default.

#### **fragment_offset**

Controls the margin from which you want to start highlighting. Only valid when using the `fvh` highlighter.

控制你想开始加亮的边距。只在使用 "fvh "高亮时有效。

#### **fragment_size**

The size of the highlighted fragment in characters. Defaults to 100.

突出显示的片段的大小，以字符为单位。默认为100。

#### **highlight_query**

Highlight matches for a query other than the search query. This is especially useful if you use a rescore query because those are not taken into account by highlighting by default.

突出显示除搜索查询之外的其他查询的匹配结果。如果你使用一个重新评分的查询，这一点特别有用，因为那些查询在默认情况下不会被高亮显示所考虑。

>  Elasticsearch does not validate that `highlight_query` contains the search query in any way so it is possible to define it so legitimate query results are not highlighted. Generally, you should include the search query as part of the `highlight_query`.

#### **matched_fields**

Combine matches on multiple fields to highlight a single field. This is most intuitive for multifields that analyze the same string in different ways. All `matched_fields` must have `term_vector` set to `with_positions_offsets`, but only the field to which the matches are combined is loaded so only that field benefits from having `store` set to `yes`. Only valid for the `fvh` highlighter.

结合多个字段的匹配来突出一个字段。这对以不同方式分析同一字符串的多字段是最直观的。所有`matched_fields`都必须将`term_vector`设置为`with_positions_offsets`，但只有被组合的字段被加载，所以只有该字段从`store`设置为`yes`中受益。只对 "fvh "高亮显示有效。

#### **no_match_size**

The amount of text you want to return from the beginning of the field if there are no matching fragments to highlight. Defaults to 0 (nothing is returned).

如果没有匹配的片段需要突出显示，你想从字段的开头返回的文本量。默认值为0（不返回任何内容）。

#### **number_of_fragments**

The maximum number of fragments to return. If the number of fragments is set to 0, no fragments are returned. Instead, the entire field contents are highlighted and returned. This can be handy when you need to highlight short texts such as a title or address, but fragmentation is not required. If `number_of_fragments` is 0, `fragment_size` is ignored. Defaults to 5.

要返回的最大片段数。如果片段数量被设置为0，则不返回任何片段。相反，整个字段的内容被高亮显示并返回。当你需要突出显示短文，如标题或地址时，这可能很方便，但不需要碎片。如果`number_of_fragments`是0，`fragment_size`被忽略。默认为5。

#### **order**

Sorts highlighted fragments by score when set to `score`. By default, fragments will be output in the order they appear in the field (order: `none`). Setting this option to `score` will output the most relevant fragments first. Each highlighter applies its own logic to compute relevancy scores. See the document [How highlighters work internally](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/highlighting.html#how-es-highlighters-work-internally) for more details how different highlighters find the best fragments.

当设置为`score`时，按分数对突出的片段进行排序。默认情况下，片段将按照它们在字段中出现的顺序输出（顺序：`none'）。将此选项设置为`score`将首先输出最相关的片段。每个高亮器都应用自己的逻辑来计算相关度分数。关于不同的高亮器如何找到最佳片段的更多细节，请参见文档[高亮器的内部工作方式](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/highlighting.html#how-es-highlighters-work-internally)。

#### **phrase_limit**

Controls the number of matching phrases in a document that are considered. Prevents the `fvh` highlighter from analyzing too many phrases and consuming too much memory. When using `matched_fields`, `phrase_limit` phrases per matched field are considered. Raising the limit increases query time and consumes more memory. Only supported by the `fvh` highlighter. Defaults to 256.

控制一个文档中被考虑的匹配短语的数量。防止`fvh`高亮显示器分析太多的短语和消耗太多的内存。当使用`matched_fields`时，每个匹配的字段会考虑`phrase_limit`短语。提高限制会增加查询时间和消耗更多内存。只支持`fvh'高亮显示。默认为256。

#### **pre_tags**

Use in conjunction with `post_tags` to define the HTML tags to use for the highlighted text. By default, highlighted text is wrapped in `<em>` and `</em>` tags. Specify as an array of strings.

与`post_tags`一起使用，定义高亮文本使用的HTML标签。默认情况下，高亮显示的文本被包裹在`<em>`和`</em>`标签中。指定为一个字符串数组。

#### **post_tags**

Use in conjunction with `pre_tags` to define the HTML tags to use for the highlighted text. By default, highlighted text is wrapped in `<em>` and `</em>` tags. Specify as an array of strings.

#### **require_field_match**

By default, only fields that contains a query match are highlighted. Set `require_field_match` to `false` to highlight all fields. Defaults to `true`.

默认情况下，只有包含查询匹配的字段被高亮显示。将`require_field_match`设置为`false`以突出显示所有字段。默认为 "true"。



#### **max_analyzed_offset**

By default, the maximum number of characters analyzed for a highlight request is bounded by the value defined in the [`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/index-modules.html#index-max-analyzed-offset) setting, and when the number of characters exceeds this limit an error is returned. If this setting is set to a non-negative value, the highlighting stops at this defined maximum limit, and the rest of the text is not processed, thus not highlighted and no error is returned. The [`max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/highlighting.html#max-analyzed-offset) query setting does **not** override the [`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/index-modules.html#index-max-analyzed-offset) which prevails when it’s set to lower value than the query setting.

默认情况下，高亮请求分析的最大字符数以[`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/index-modules.html#index-max-analyzed-offset)设置中定义的值为界，当字符数超过这个限制时，会返回错误。如果这个设置是一个非负值，高亮就会在这个定义的最大极限处停止，其余的文本不被处理，因此不高亮，也不返回错误。[`max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/highlighting.html#max-analyzed-offset)的查询设置不会覆盖[`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/index-modules.html#index-max-analyzed-offset)，当它被设置为比查询设置更低的值时，就以它为准。

#### **tags_schema**

Set to `styled` to use the built-in tag schema. The `styled` schema defines the following `pre_tags` and defines `post_tags` as `</em>`.`<em class="hlt1">, <em class="hlt2">, <em class="hlt3">, <em class="hlt4">, <em class="hlt5">, <em class="hlt6">, <em class="hlt7">, <em class="hlt8">, <em class="hlt9">, <em class="hlt10">`

设置为`styled`可以使用内置的标签模式。`styled`模式定义了以下`pre_tags`并将`post_tags`定义为`</em>`。 `em class="hlt1">, <em class="hlt2">, <em class="hlt3">, <em class="hlt4">, <em class="hlt5">, <em class="hlt6">, <em class="hlt7">, <em class="hlt8">, <em class="hlt9">, <em class="hlt10">`.

#### **type**

The highlighter to use: `unified`, `plain`, or `fvh`. Defaults to `unified`.

### 高亮示例

#### 覆盖全局设置

你可以在全局范围内指定highlighter设置，并有选择地覆盖个别字段的设置。

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "number_of_fragments" : 3,
    "fragment_size" : 150,
    "fields" : {
      "body" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
      "blog.title" : { "number_of_fragments" : 0 },
      "blog.author" : { "number_of_fragments" : 0 },
      "blog.comment" : { "number_of_fragments" : 5, "order" : "score" }
    }
  }
}
```

#### 指定highlight query

You can specify a `highlight_query` to take additional information into account when highlighting. For example, the following query includes both the search query and rescore query in the `highlight_query`. Without the `highlight_query`, highlighting would only take the search query into account.

```console
GET /_search
{
  "query": {
    "match": {
      "comment": {
        "query": "foo bar"
      }
    }
  },
  "rescore": {
    "window_size": 50,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "comment": {
            "query": "foo bar",
            "slop": 1
          }
        }
      },
      "rescore_query_weight": 10
    }
  },
  "_source": false,
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "highlight_query": {
          "bool": {
            "must": {
              "match": {
                "comment": {
                  "query": "foo bar"
                }
              }
            },
            "should": {
              "match_phrase": {
                "comment": {
                  "query": "foo bar",
                  "slop": 1,
                  "boost": 10.0
                }
              }
            },
            "minimum_should_match": 0
          }
        }
      }
    }
  }
}
```



#### 设置highlighter type

The `type` field allows to force a specific highlighter type. The allowed values are: `unified`, `plain` and `fvh`. The following is an example that forces the use of the plain highlighter:

```console
GET /_search
{
  "query": {
    "match": { "user.id": "kimchy" }
  },
  "highlight": {
    "fields": {
      "comment": { "type": "plain" }
    }
  }
}
```

#### 配置 highlighting tags

By default, the highlighting will wrap highlighted text in `<em>` and `</em>`. This can be controlled by setting `pre_tags` and `post_tags`, for example:

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "pre_tags" : ["<tag1>"],
    "post_tags" : ["</tag1>"],
    "fields" : {
      "body" : {}
    }
  }
}
```

When using the fast vector highlighter, you can specify additional tags and the "importance" is ordered.

你可以指定额外的标签，而且 "重要性 "是有顺序的。

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "pre_tags" : ["<tag1>", "<tag2>"],
    "post_tags" : ["</tag1>", "</tag2>"],
    "fields" : {
      "body" : {}
    }
  }
}
```

You can also use the built-in `styled` tag schema:

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "tags_schema" : "styled",
    "fields" : {
      "comment" : {}
    }
  }
}
```

#### 在source高亮

Forces the highlighting to highlight fields based on the source even if fields are stored separately. Defaults to `false`.

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "fields" : {
      "comment" : {"force_source" : true}
    }
  }
}
```

#### 所有字段启用高亮

By default, only fields that contains a query match are highlighted. Set `require_field_match` to `false` to highlight all fields.

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "require_field_match": false,
    "fields": {
      "body" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
    }
  }
}
```

#### 合并多个字段的匹配

This is only supported by the `fvh` highlighter

The Fast Vector Highlighter can combine matches on multiple fields to highlight a single field. This is most intuitive for multifields that analyze the same string in different ways. All `matched_fields` must have `term_vector` set to `with_positions_offsets` but only the field to which the matches are combined is loaded so only that field would benefit from having `store` set to `yes`.

In the following examples, `comment` is analyzed by the `english` analyzer and `comment.plain` is analyzed by the `standard` analyzer.

这只被`fvh`高亮程序所支持

快速矢量高亮程序可以结合多个字段的匹配来高亮一个字段。这对以不同方式分析同一字符串的多字段来说是最直观的。所有`matched_fields`都必须将`term_vector`设置为`with_positions_offsets`，但只有被匹配的字段才会被加载，所以只有该字段会从`store`设置为`yes`中受益。

在下面的例子中，`comment'是由`english'分析器分析的，`comment.plain'是由`standard'分析器分析的。

```console
GET /_search
{
  "query": {
    "query_string": {
      "query": "comment.plain:running scissors",
      "fields": [ "comment" ]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "matched_fields": [ "comment", "comment.plain" ],
        "type": "fvh"
      }
    }
  }
}
```

The above matches both "run with scissors" and "running with scissors" and would highlight "running" and "scissors" but not "run". If both phrases appear in a large document then "running with scissors" is sorted above "run with scissors" in the fragments list because there are more matches in that fragment.

```console
GET /_search
{
  "query": {
    "query_string": {
      "query": "running scissors",
      "fields": ["comment", "comment.plain^10"]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "matched_fields": ["comment", "comment.plain"],
        "type" : "fvh"
      }
    }
  }
}
```

The above highlights "run" as well as "running" and "scissors" but still sorts "running with scissors" above "run with scissors" because the plain match ("running") is boosted.

```console
GET /_search
{
  "query": {
    "query_string": {
      "query": "running scissors",
      "fields": [ "comment", "comment.plain^10" ]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "matched_fields": [ "comment.plain" ],
        "type": "fvh"
      }
    }
  }
}
```

The above query wouldn’t highlight "run" or "scissor" but shows that it is just fine not to list the field to which the matches are combined (`comment`) in the matched fields.

Technically it is also fine to add fields to `matched_fields` that don’t share the same underlying string as the field to which the matches are combined. The results might not make much sense and if one of the matches is off the end of the text then the whole query will fail.

There is a small amount of overhead involved with setting `matched_fields` to a non-empty array so always prefer

```js
    "highlight": {
        "fields": {
            "comment": {}
        }
    }
```

to

```js
    "highlight": {
        "fields": {
            "comment": {
                "matched_fields": ["comment"],
                "type" : "fvh"
            }
        }
    }
```

#### 明确排列突出显示的字段

Elasticsearch按照字段发送的顺序突出显示，但根据JSON规范，对象是无序的。如果你需要明确字段被高亮显示的顺序，请将`fields`指定为一个数组。

```console
GET /_search
{
  "highlight": {
    "fields": [
      { "title": {} },
      { "text": {} }
    ]
  }
}
```

None of the highlighters built into Elasticsearch care about the order that the fields are highlighted but a plugin might.

Elasticsearch内置的高亮器都不关心字段被高亮的顺序，但一个插件可能会

#### 控制高亮的片段

Each field highlighted can control the size of the highlighted fragment in characters (defaults to `100`), and the maximum number of fragments to return (defaults to `5`). For example:

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "fields" : {
      "comment" : {"fragment_size" : 150, "number_of_fragments" : 3}
    }
  }
}
```

On top of this it is possible to specify that highlighted fragments need to be sorted by score:

在此基础上，还可以指定突出显示的片段需要按分数排序。

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "order" : "score",
    "fields" : {
      "comment" : {"fragment_size" : 150, "number_of_fragments" : 3}
    }
  }
}
```

If the `number_of_fragments` value is set to `0` then no fragments are produced, instead the whole content of the field is returned, and of course it is highlighted. This can be very handy if short texts (like document title or address) need to be highlighted but no fragmentation is required. Note that `fragment_size` is ignored in this case.

如果`number_of_fragments`值设置为`0`，则不会产生任何片段，而是返回字段的全部内容，当然也会高亮显示。如果需要突出显示短文（如文件标题或地址），但又不需要碎片，这就非常方便了。请注意，在这种情况下，`fragment_size`被忽略。

```console
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "fields" : {
      "body" : {},
      "blog.title" : {"number_of_fragments" : 0}
    }
  }
}
```

When using `fvh` one can use `fragment_offset` parameter to control the margin to start highlighting from.

In the case where there is no matching fragment to highlight, the default is to not return anything. Instead, we can return a snippet of text from the beginning of the field by setting `no_match_size` (default `0`) to the length of the text that you want returned. The actual length may be shorter or longer than specified as it tries to break on a word boundary.

```console
GET /_search
{
  "query": {
    "match": { "user.id": "kimchy" }
  },
  "highlight": {
    "fields": {
      "comment": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "no_match_size": 150
      }
    }
  }
}
```

#### Highlight using the postings list

Here is an example of setting the `comment` field in the index mapping to allow for highlighting using the postings:

```console
PUT /example
{
  "mappings": {
    "properties": {
      "comment" : {
        "type": "text",
        "index_options" : "offsets"
      }
    }
  }
}
```

Here is an example of setting the `comment` field to allow for highlighting using the `term_vectors` (this will cause the index to be bigger):

```console
PUT /example
{
  "mappings": {
    "properties": {
      "comment" : {
        "type": "text",
        "term_vector" : "with_positions_offsets"
      }
    }
  }
}
```

#### Specify a fragmenter for the plain highlighter

When using the `plain` highlighter, you can choose between the `simple` and `span` fragmenters:

```console
GET my-index-000001/_search
{
  "query": {
    "match_phrase": { "message": "number 1" }
  },
  "highlight": {
    "fields": {
      "message": {
        "type": "plain",
        "fragment_size": 15,
        "number_of_fragments": 3,
        "fragmenter": "simple"
      }
    }
  }
}
```

Response:

```console-result
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.6011951,
    "hits": [
      {
        "_index": "my-index-000001",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.6011951,
        "_source": {
          "message": "some message with the number 1",
          "context": "bar"
        },
        "highlight": {
          "message": [
            " with the <em>number</em>",
            " <em>1</em>"
          ]
        }
      }
    ]
  }
}
```

 

```console
GET my-index-000001/_search
{
  "query": {
    "match_phrase": { "message": "number 1" }
  },
  "highlight": {
    "fields": {
      "message": {
        "type": "plain",
        "fragment_size": 15,
        "number_of_fragments": 3,
        "fragmenter": "span"
      }
    }
  }
}
```

Response:

```console-result
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.6011951,
    "hits": [
      {
        "_index": "my-index-000001",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.6011951,
        "_source": {
          "message": "some message with the number 1",
          "context": "bar"
        },
        "highlight": {
          "message": [
            " with the <em>number</em> <em>1</em>"
          ]
        }
      }
    ]
  }
}
```



If the `number_of_fragments` option is set to `0`, `NullFragmenter` is used which does not fragment the text at all. This is useful for highlighting the entire contents of a document or field.

### 高亮如何内部运作

给定一个查询和一个文本（文档字段的内容），高亮器的目标是为查询找到最好的文本片段，并在找到的片段中突出显示查询词。为此，高亮程序需要解决几个问题。

- 如何将一个文本分解成片段？
- 如何在所有片段中找到最佳片段？
- 如何突出显示片段中的查询词？

#### 如何将一个文本分解成片段？

Plain highlighter begins with analyzing the text using the given analyzer, and creating a token stream from it. 

Plain highlighter uses a very simple algorithm to break the token stream into fragments. It loops through terms in the token stream, and every time the current term’s end_offset exceeds `fragment_size` multiplied by the number of created fragments, a new fragment is created. A little more computation is done with using `span` fragmenter to avoid breaking up text between highlighted terms. But overall, since the breaking is done only by `fragment_size`, some fragments can be quite odd, e.g. beginning with a punctuation mark.

Unified or FVH highlighters do a better job of breaking up a text into fragments by utilizing Java’s `BreakIterator`. This ensures that a fragment is a valid sentence as long as `fragment_size` allows for this.

#### How to find the best fragments?

Relevant settings: `number_of_fragments`.

To find the best, most relevant, fragments, a highlighter needs to score each fragment in respect to the given query. The goal is to score only those terms that participated in generating the *hit* on the document. For some complex queries, this is still work in progress.

The plain highlighter creates an in-memory index from the current token stream, and re-runs the original query criteria through Lucene’s query execution planner to get access to low-level match information for the current text. For more complex queries the original query could be converted to a span query, as span queries can handle phrases more accurately. Then this obtained low-level match information is used to score each individual fragment. The scoring method of the plain highlighter is quite simple. Each fragment is scored by the number of unique query terms found in this fragment. The score of individual term is equal to its boost, which is by default is 1. Thus, by default, a fragment that contains one unique query term, will get a score of 1; and a fragment that contains two unique query terms, will get a score of 2 and so on. The fragments are then sorted by their scores, so the highest scored fragments will be output first.

FVH doesn’t need to analyze the text and build an in-memory index, as it uses pre-indexed document term vectors, and finds among them terms that correspond to the query. FVH scores each fragment by the number of query terms found in this fragment. Similarly to plain highlighter, score of individual term is equal to its boost value. In contrast to plain highlighter, all query terms are counted, not only unique terms.

Unified highlighter can use pre-indexed term vectors or pre-indexed terms offsets, if they are available. Otherwise, similar to Plain Highlighter, it has to create an in-memory index from the text. Unified highlighter uses the BM25 scoring model to score fragments.

#### How to highlight the query terms in a fragment?

Relevant settings: `pre-tags`, `post-tags`.

The goal is to highlight only those terms that participated in generating the *hit* on the document. For some complex boolean queries, this is still work in progress, as highlighters don’t reflect the boolean logic of a query and only extract leaf (terms, phrases, prefix etc) queries.

Plain highlighter given the token stream and the original text, recomposes the original text to highlight only terms from the token stream that are contained in the low-level match information structure from the previous step.

FVH and unified highlighter use intermediate data structures to represent fragments in some raw form, and then populate them with actual text.

A highlighter uses `pre-tags`, `post-tags` to encode highlighted terms.

### An example of the work of the unified highlighter

Let’s look in more details how unified highlighter works.

First, we create a index with a text field `content`, that will be indexed using `english` analyzer, and will be indexed without offsets or term vectors.

```js
PUT test_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

We put the following document into the index:

```js
PUT test_index/_doc/doc1
{
  "content" : "For you I'm only a fox like a hundred thousand other foxes. But if you tame me, we'll need each other. You'll be the only boy in the world for me. I'll be the only fox in the world for you."
}
```

And we ran the following query with a highlight request:

```js
GET test_index/_search
{
  "query": {
    "match_phrase" : {"content" : "only fox"}
  },
  "highlight": {
    "type" : "unified",
    "number_of_fragments" : 3,
    "fields": {
      "content": {}
    }
  }
}
```

After `doc1` is found as a hit for this query, this hit will be passed to the unified highlighter for highlighting the field `content` of the document. Since the field `content` was not indexed either with offsets or term vectors, its raw field value will be analyzed, and in-memory index will be built from the terms that match the query:

```
{"token":"onli","start_offset":12,"end_offset":16,"position":3},
{"token":"fox","start_offset":19,"end_offset":22,"position":5},
{"token":"fox","start_offset":53,"end_offset":58,"position":11},
{"token":"onli","start_offset":117,"end_offset":121,"position":24},
{"token":"onli","start_offset":159,"end_offset":163,"position":34},
{"token":"fox","start_offset":164,"end_offset":167,"position":35}
```

Our complex phrase query will be converted to the span query: `spanNear([text:onli, text:fox], 0, true)`, meaning that we are looking for terms "onli: and "fox" within 0 distance from each other, and in the given order. The span query will be run against the created before in-memory index, to find the following match:

```
{"term":"onli", "start_offset":159, "end_offset":163},
{"term":"fox", "start_offset":164, "end_offset":167}
```

In our example, we have got a single match, but there could be several matches. Given the matches, the unified highlighter breaks the text of the field into so called "passages". Each passage must contain at least one match. The unified highlighter with the use of Java’s `BreakIterator` ensures that each passage represents a full sentence as long as it doesn’t exceed `fragment_size`. For our example, we have got a single passage with the following properties (showing only a subset of the properties here):

```
Passage:
    startOffset: 147
    endOffset: 189
    score: 3.7158387
    matchStarts: [159, 164]
    matchEnds: [163, 167]
    numMatches: 2
```

Notice how a passage has a score, calculated using the BM25 scoring formula adapted for passages. Scores allow us to choose the best scoring passages if there are more passages available than the requested by the user `number_of_fragments`. Scores also let us to sort passages by `order: "score"` if requested by the user.

As the final step, the unified highlighter will extract from the field’s text a string corresponding to each passage:

```
"I'll be the only fox in the world for you."
```

and will format with the tags <em> and </em> all matches in this string using the passages’s `matchStarts` and `matchEnds` information:

```
I'll be the <em>only</em> <em>fox</em> in the world for you.
```

This kind of formatted strings are the final result of the highlighter returned to the user.

## Long-running搜索

Elasticsearch generally allows you to quickly search across big amounts of data. There are situations where a search executes on many shards, possibly against [frozen indices](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/freeze-index-api.html) and spanning multiple [remote clusters](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/modules-remote-clusters.html), for which results are not expected to be returned in milliseconds. When you need to execute long-running searches, synchronously waiting for its results to be returned is not ideal. Instead, Async search lets you submit a search request that gets executed *asynchronously*, monitor the progress of the request, and retrieve results at a later stage. You can also retrieve partial results as they become available but before the search has completed.

You can submit an async search request using the [submit async search](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/async-search.html#submit-async-search) API. The [get async search](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/async-search.html#get-async-search) API allows you to monitor the progress of an async search request and retrieve its results. An ongoing async search can be deleted through the [delete async search](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/async-search.html#delete-async-search) API.

Elasticsearch通常允许你在大量的数据中快速搜索。在有些情况下，搜索会在许多分片上执行，可能是针对[冻结索引](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/freeze-index-api.html)和跨越多个[远程集群](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/modules-remote-clusters.html)，对于这些搜索，不期望在几毫秒内返回结果。当你需要执行长期运行的搜索时，同步地等待其结果的返回并不理想。相反，Async搜索可以让你提交一个搜索请求，该请求会被异步执行*，监控请求的进展，并在稍后阶段检索结果。你也可以在搜索完成之前检索到部分结果，因为它们是可用的。

你可以使用[submit async search](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/async-search.html#submit-async-search) API提交一个异步搜索请求。[get async search](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/async-search.html#get-async-search) API允许你监控一个异步搜索请求的进度并检索其结果。正在进行的异步搜索可以通过[delete async search](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/async-search.html#delete-async-search) API删除。

## 近实时搜索

The overview of [documents and indices](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/documents-indices.html) indicates that when a document is stored in Elasticsearch, it is indexed and fully searchable in *near real-time*--within 1 second. What defines near real-time search?

[文档和索引](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/documents-indices.html)的概述表明，当一个文档存储在Elasticsearch中时，它在*近实时*--1秒内就会被索引并完全搜索。什么定义了近实时搜索？

Lucene, the Java libraries on which Elasticsearch is based, introduced the concept of per-segment search. A *segment* is similar to an inverted index, but the word *index* in Lucene means "a collection of segments plus a commit point". After a commit, a new segment is added to the commit point and the buffer is cleared.

Sitting between Elasticsearch and the disk is the filesystem cache. Documents in the in-memory indexing buffer ([Figure 1](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/near-real-time.html#img-pre-refresh)) are written to a new segment ([Figure 2](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/near-real-time.html#img-post-refresh)). The new segment is written to the filesystem cache first (which is cheap) and only later is it flushed to disk (which is expensive). However, after a file is in the cache, it can be opened and read just like any other file.



Lucene，Elasticsearch所基于的Java库，引入了每段搜索的概念。一个*segment*类似于一个倒排索引，但在Lucene中，*index*这个词意味着 "一个segment的集合加上一个commit point"。在提交之后，一个新的segment被添加到提交点，并且缓冲区被清空。

坐落在Elasticsearch和磁盘之间的是文件系统缓存。内存索引缓冲区中的文件（[图1](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/near-real-time.html#img-pre-refresh)）被写入一个新segment（[图2](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/near-real-time.html#img-post-refresh)）。新的segment首先被写入文件系统缓存（这很便宜），然后才被刷新到磁盘（这很昂贵）。然而，当一个文件进入缓冲区后，它可以像其他文件一样被打开和读取。

Figure 1. A Lucene index with new documents in the in-memory buffer

![A Lucene index with new documents in the in-memory buffer](assets/lucene-in-memory-buffer.png)



Lucene allows new segments to be written and opened, making the documents they contain visible to search without performing a full commit. This is a much lighter process than a commit to disk, and can be done frequently without degrading performance.

Lucene允许写入和打开新的segment，使它们所包含的文件在搜索中可见，而不需要执行完整的提交。这比提交到磁盘的过程要更轻量级，而且可以经常进行而不降低性能。

Figure 2. The buffer contents are written to a segment, which is searchable, but is not yet committed

![The buffer contents are written to a segment, which is searchable, but is not yet committed](assets/lucene-written-not-committed.png)

In Elasticsearch, this process of writing and opening a new segment is called a *refresh*. A refresh makes all operations performed on an index since the last refresh available for search. You can control refreshes through the following means:

在Elasticsearch中，这个写入和打开新segment的过程被称为*refresh*。刷新使上次刷新后在索引上进行的所有操作都可用于搜索。你可以通过以下方式来控制刷新。

- 等待刷新的时间间隔
- 设置[?refresh](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/docs-refresh.html)选项
- 使用[refresh API](https://www.elastic.co/guide/en/elasticsearch/reference/7.14/indices-refresh.html)来明确完成刷新(`POST _refresh`)

By default, Elasticsearch periodically refreshes indices every second, but only on indices that have received one search request or more in the last 30 seconds. This is why we say that Elasticsearch has *near* real-time search: document changes are not visible to search immediately, but will become visible within this timeframe.

默认情况下，Elasticsearch每秒钟都会定期刷新索引，但只刷新在过去30秒内收到一个或多个搜索请求的索引。这就是为什么我们说Elasticsearch有*近*实时的搜索：文件的变化不会立即被搜索到，但会在这个时间范围内变得可见。

## 分页搜索结果

默认情况下，搜索会返回前10个匹配的结果。要翻阅更大的结果集，你可以使用搜索API的from和size参数。from参数定义了要跳过的点击数，默认为0。 size参数是要返回的最大点击数。这两个参数一起定义了一个页面的结果。

```console
GET /_search
{
  "from": 5,
  "size": 20,
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

避免使用from和大小，以使页面太深或一次请求太多的结果。搜索请求通常横跨多个分片。每个分片必须将其请求的命中和任何先前页面的命中加载到内存中。对于深层页面或大的结果集，这些操作会大大增加内存和CPU的使用，导致性能下降或节点故障。

默认情况下，你不能使用`from`和`size`来翻阅超过10,000个点击。这个限制是由[`index.max_result_window`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-max-result-window)索引设置的一个保障。如果你需要翻阅超过10,000条信息，请使用[`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#search-after)参数代替。

> Elasticsearch uses Lucene’s internal doc IDs as tie-breakers. These internal doc IDs can be completely different across replicas of the same data. When paging search hits, you might occasionally see that documents with the same sort values are not ordered consistently.

### Search_after

你可以使用search_after参数来检索下一页的点击率，使用前一页的一组排序值。

使用search_after需要用相同的查询和排序值进行多次搜索请求。如果在这些请求之间发生刷新，你的结果的顺序可能会改变，导致各页的结果不一致。为了防止这种情况，你可以创建一个时间点（PIT）来保留你的搜索的当前索引状态。

```apl
POST /my-index-000001/_pit?keep_alive=1m
```

The API returns a PIT ID.

```json
{
  "id": "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
}
```

为了获得第一页的结果，提交一个带有排序参数的搜索请求。如果使用PIT，在pit.id参数中指定PIT ID，并在请求路径中省略目标数据流或索引。

> important
>
> All PIT search requests add an implicit sort tiebreaker field called `_shard_doc`, which can also be provided explicitly. If you cannot use a PIT, we recommend that you include a tiebreaker field in your `sort`. This tiebreaker field should contain a unique value for each document. If you don’t include a tiebreaker field, your paged results could miss or duplicate hits.
>
> 所有的PIT搜索请求都增加了一个隐含的分类破译字段，称为_shard_doc，也可以明确提供。如果你不能使用PIT，我们建议你在你的排序中包含一个破译字段。这个分界线字段应该为每个文档包含一个唯一的值。如果你不包括tiebreaker字段，你的分页结果可能会错过或重复点击。



> Note:
>
> Search after requests have optimizations that make them faster when the sort order is `_shard_doc` and total hits are not tracked. If you want to iterate over all documents regardless of the order, this is the most efficient option.
>
> 当排序顺序为_shard_doc且不跟踪总命中率时，搜索后请求有优化功能，使其更快。如果你想遍历所有的文件，而不管顺序如何，这是最有效的选择。



> important 
>
> If the `sort` field is a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html) in some target data streams or indices but a [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html) field in other targets, use the `numeric_type` parameter to convert the values to a single resolution and the `format` parameter to specify a [date format](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html) for the `sort` field. Otherwise, Elasticsearch won’t interpret the search after parameter correctly in each request.
>
> 如果`sort`字段在某些目标数据流或索引中是[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)，但在其他目标中是[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html)字段，请使用`numeric_type`参数将数值转换为单一分辨率，并使用`format`参数为`sort`字段指定[date format](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html) 。否则，Elasticsearch不会正确解释每个请求中的搜索后参数。

```apl
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
	    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
	    "keep_alive": "1m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos", "numeric_type" : "date_nanos" }}
  ]
}
```

The search response includes an array of `sort` values for each hit. If you used a PIT, a tiebreaker is included as the last `sort` values for each hit. This tiebreaker called `_shard_doc` is added automatically on every search requests that use a PIT. The `_shard_doc` value is the combination of the shard index within the PIT and the Lucene’s internal doc ID, it is unique per document and constant within a PIT. You can also add the tiebreaker explicitly in the search request to customize the order:

搜索响应包括每个命中的`sort`值数组。如果你使用了PIT，一个破译器将作为每个命中的最后一个`sort`值被包括在内。这个称为"_shard_doc "的分界线被自动添加到每个使用PIT的搜索请求中。`_shard_doc`值是PIT中的分片索引和Lucene内部文档ID的组合，它是每个文档唯一的，在PIT中是恒定的。你也可以在搜索请求中明确地添加分界线来定制顺序。

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
	    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
	    "keep_alive": "1m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos"}},
    {"_shard_doc": "desc"}
  ]
}
```

```json
{
  "pit_id" : "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
  "took" : 17,
  "timed_out" : false,
  "_shards" : ...,
  "hits" : {
    "total" : ...,
    "max_score" : null,
    "hits" : [
      ...
      {
        "_index" : "my-index-000001",
        "_id" : "FaslK3QBySSL_rrj9zM5",
        "_score" : null,
        "_source" : ...,
        "sort" : [                                
          "2021-05-20T05:30:04.832Z",
          4294967298                              
        ]
      }
    ]
  }
}
```

| Updated `id` for the point in time.                          |      |
| ------------------------------------------------------------ | ---- |
| Sort values for the last returned hit.                       |      |
| The tiebreaker value, unique per document within the `pit_id`. |      |

要获得下一页的结果，使用最后一次命中的排序值（包括平局）作为search_after参数，重新运行之前的搜索。如果使用PIT，在pit.id参数中使用最新的PIT ID。搜索的查询和排序参数必须保持不变。如果提供，from参数必须是0（默认）或-1。

```apl
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
	    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
	    "keep_alive": "1m"
  },
  "sort": [
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos"}}
  ],
  "search_after": [                                
    "2021-05-20T05:30:04.832Z",
    4294967298
  ],
  "track_total_hits": false                        
}
```

| PIT ID returned by the previous search.                    |
| ---------------------------------------------------------- |
| Sort values from the previous search’s last hit.           |
| Disable the tracking of total hits to speed up pagination. |

You can repeat this process to get additional pages of results. If using a PIT, you can extend the PIT’s retention period using the `keep_alive` parameter of each search request.

When you’re finished, you should delete your PIT.

你可以重复这个过程以获得更多的结果页。如果使用PIT，你可以使用每个搜索请求的`keep_alive`参数延长PIT的保留期。

当你完成后，你应该删除你的PIT。

```apl
DELETE /_pit
{
    "id" : "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
}
```

### 滚动搜索结果(不推荐 没继续看)

我们不再推荐使用滚动API进行深度分页。如果你需要在分页超过10,000次时保留索引状态，请使用带有时间点（PIT）的[`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#search-after) 参数。

虽然 "搜索 "请求返回单个 "页面 "的结果，但 "滚动 "API可用于从单个搜索请求中检索大量结果（甚至所有结果），其方式与在传统数据库中使用游标的方式相同。

滚动的目的不是为了满足用户的实时请求，而是为了处理大量的数据，例如，为了将一个数据流或索引的内容重新编入一个新的数据流或索引，并采用不同的配置。

## Retrieve inner hits

The [parent-join](https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html) and [nested](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html) features allow the return of documents that have matches in a different scope. In the parent/child case, parent documents are returned based on matches in child documents or child documents are returned based on matches in parent documents. In the nested case, documents are returned based on matches in nested inner objects.

In both cases, the actual matches in the different scopes that caused a document to be returned are hidden. In many cases, it’s very useful to know which inner nested objects (in the case of nested) or children/parent documents (in the case of parent/child) caused certain information to be returned. The inner hits feature can be used for this. This feature returns per search hit in the search response additional nested hits that caused a search hit to match in a different scope.

[parent-join](https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html)和[nested](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)功能允许返回的文档由不同scope的数据。在parent/child 的情况下，父文档基于子文档中的匹配被返回，或者子文档基于父文档中的匹配被返回。在nested的情况下，doc是根据nested的内部对象中的匹配来返回的。

在这两种情况下，导致文档被返回的不同作用域中的实际匹配是隐藏的。在许多情况下，知道哪个内部nested对象（在嵌套的情况下）或子/父文档（在父/子的情况下）导致某些信息被返回是非常有用的。inner hits功能可以用于此。这个功能在搜索响应中返回每个导致搜索命中在不同范围内匹配的额外嵌套命中.

Inner hits can be used by defining an `inner_hits` definition on a `nested`, `has_child` or `has_parent` query and filter. The structure looks like this:

```js
"<query>" : {
  "inner_hits" : {
    <inner_hits_options>
  }
}
```



If `inner_hits` is defined on a query that supports it then each search hit will contain an `inner_hits` json object with the following structure:

```js
"hits": [
  {
    "_index": ...,
    "_type": ...,
    "_id": ...,
    "inner_hits": {
      "<inner_hits_name>": {
        "hits": {
          "total": ...,
          "hits": [
            {
              "_type": ...,
              "_id": ...,
               ...
            },
            ...
          ]
        }
      }
    },
    ...
  },
  ...
]
```



### 参数

Inner hits support the following options:

| `from` | The offset from where the first hit to fetch for each `inner_hits` in the returned regular search hits. |
| ------ | ------------------------------------------------------------ |
| `size` | The maximum number of hits to return per `inner_hits`. By default the top three matching hits are returned. |
| `sort` | How the inner hits should be sorted per `inner_hits`. By default the hits are sorted by the score. |
| `name` | The name to be used for the particular inner hit definition in the response. Useful when multiple inner hits have been defined in a single search request. The default depends in which query the inner hit is defined. For `has_child` query and filter this is the child type, `has_parent` query and filter this is the parent type and the nested query and filter this is the nested path.响应中的特定内部命中定义所使用的名称。当在一个搜索请求中定义了多个内部命中时，这个名字很有用。默认值取决于内部命中是在哪个查询中定义的。对于`has_child`查询和过滤器，这是子类型，`has_parent`查询和过滤器，这是父类型，嵌套查询和过滤器，这是嵌套路径。 |

Inner hits also supports the following per document features:

- [Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html)
- [Explain](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-explain)
- [Search fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#search-fields-param)
- [Source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-source-filtering)
- [Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#script-fields)
- [Doc value fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#docvalue-fields)
- [Include versions](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-version)
- [Include Sequence Numbers and Primary Terms](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-seq-no-primary-term)

### Nested inner hits

The nested `inner_hits` can be used to include nested inner objects as inner hits to a search hit.

```console
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested"
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "number": 1
    },
    {
      "author": "nik9000",
      "number": 2
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": { "comments.number": 2 }
      },
      "inner_hits": {} 
    }
  }
}
```

```json
# 加inner_hits的结果
{
  "took" : 13,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Test title",
          "comments" : [
            {
              "author" : "kimchy",
              "number" : 1
            },
            {
              "author" : "nik9000",
              "number" : 2
            }
          ]
        },
        "inner_hits" : {
          "comments" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.0,
              "hits" : [
                {
                  "_index" : "test",
                  "_type" : "_doc",
                  "_id" : "1",
                  "_nested" : {
                    "field" : "comments",
                    "offset" : 1
                  },
                  "_score" : 1.0,
                  "_source" : {
                    "number" : 2,
                    "author" : "nik9000"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
#不加的结果
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Test title",
          "comments" : [
            {
              "author" : "kimchy",
              "number" : 1
            },
            {
              "author" : "nik9000",
              "number" : 2
            }
          ]
        }
      }
    ]
  }
}
```



| The inner hit definition in the nested query. No other options need to be defined. |
| ------------------------------------------------------------ |

An example of a response snippet that could be generated from the above search request:

```console-result
{
  ...,
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": ...,
        "inner_hits": {
          "comments": { 
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.0,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 1.0,
                  "_source": {
                    "author": "nik9000",
                    "number": 2
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```



| The name used in the inner hit definition in the search request. A custom key can be used via the `name` option. |
| ------------------------------------------------------------ |

The `_nested` metadata is crucial in the above example, because it defines from what inner nested object this inner hit came from. The `field` defines the object array field the nested hit is from and the `offset` relative to its location in the `_source`. Due to sorting and scoring the actual location of the hit objects in the `inner_hits` is usually different than the location a nested inner object was defined.

在上面的例子中，`_nested`元数据是至关重要的，因为它定义了这个内部命中来自于哪个内部嵌套对象。字段 "定义了嵌套命中的对象阵列字段，以及相对于它在"_source "中的位置的 "offset"。由于排序和评分的原因，在`inner_hits`中命中对象的实际位置通常与嵌套的内部对象定义的位置不同。

By default the `_source` is returned also for the hit objects in `inner_hits`, but this can be changed. Either via `_source` filtering feature part of the source can be returned or be disabled. If stored fields are defined on the nested level these can also be returned via the `fields` feature.

默认情况下，`_source'也会返回给`inner_hits'中的命中对象，但这可以改变。通过`_source`过滤功能，可以返回或禁用部分来源。如果在嵌套层定义了存储字段，也可以通过`fields`功能返回

An important default is that the `_source` returned in hits inside `inner_hits` is relative to the `_nested` metadata. So in the above example only the comment part is returned per nested hit and not the entire source of the top level document that contained the comment.

一个重要的默认值是，在`inner_hits`内的点击率中返回的`_source`是相对于`_nested`元数据的。所以在上面的例子中，每个嵌套的点击只返回评论部分，而不是包含评论的整个顶层文档的来源。

### Nested inner hits 和 `_source`

Nested document don’t have a `_source` field, because the entire source of document is stored with the root document under its `_source` field. To include the source of just the nested document, the source of the root document is parsed and just the relevant bit for the nested document is included as source in the inner hit. Doing this for each matching nested document has an impact on the time it takes to execute the entire search request, especially when `size` and the inner hits' `size` are set higher than the default. To avoid the relatively expensive source extraction for nested inner hits, one can disable including the source and solely rely on doc values fields. Like this:

Nested document 没有"_source "字段，因为整个文档的来源是与根文档一起存储在其`_source `字段下。为了包括嵌套文件的来源，根文件的来源被解析，只是嵌套文件的相关部分被包括在内部命中的来源。对每一个匹配的嵌套文档进行这样的处理会影响到执行整个搜索请求的时间，特别是当`size`和内部命中的`size`设置得比默认值高时。为了避免对嵌套的内部命中进行相对昂贵的源提取，我们可以不包括源，而仅仅依靠文档值字段。像这样:

Inner_hits的source字段就不会提取了。

"inner_hits": {
        "_source": false,
        "docvalue_fields": [
          "comments.text.keyword"
        ]
      }

```console
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested"
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text"
    },
    {
      "author": "nik9000",
      "text": "words words words"
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": { "comments.text": "words" }
      },
      "inner_hits": {
        "_source": false,
        "docvalue_fields": [
          "comments.text.keyword"
        ]
      }
    }
  }
}
```

### 层次分明的nested对象和inner hits.

If a mapping has multiple levels of hierarchical nested object fields each level can be accessed via dot notated path. For example if there is a `comments` nested field that contains a `votes` nested field and votes should directly be returned with the root hits then the following path can be defined:

如果一个映射有多个层次的嵌套对象字段，每个层次可以通过点标记的路径来访问。例如，如果有一个 "comments "嵌套字段包含一个 "votes "嵌套字段，votes应该直接与根点击一起返回，那么可以定义以下路径。

```console
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested",
        "properties": {
          "votes": {
            "type": "nested"
          }
        }
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text",
      "votes": []
    },
    {
      "author": "nik9000",
      "text": "words words words",
      "votes": [
        {"value": 1 , "voter": "kimchy"},
        {"value": -1, "voter": "other"}
      ]
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments.votes",
        "query": {
          "match": {
            "comments.votes.voter": "kimchy"
          }
        },
        "inner_hits" : {}
    }
  }
}
```

Which would look like:

```console-result
{
  ...,
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.6931471,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.6931471,
        "_source": ...,
        "inner_hits": {
          "comments.votes": { 
            "hits": {
              "total" : {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 0.6931471,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1,
                    "_nested": {
                      "field": "votes",
                      "offset": 0
                    }
                  },
                  "_score": 0.6931471,
                  "_source": {
                    "value": 1,
                    "voter": "kimchy"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```



This indirect referencing is only supported for nested inner hits.

### Parent/child inner hits

The parent/child `inner_hits` can be used to include parent or child:

```console
PUT test
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "my_parent": "my_child"
        }
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "number": 1,
  "my_join_field": "my_parent"
}

PUT test/_doc/2?routing=1&refresh
{
  "number": 1,
  "my_join_field": {
    "name": "my_child",
    "parent": "1"
  }
}

POST test/_search
{
  "query": {
    "has_child": {
      "type": "my_child",
      "query": {
        "match": {
          "number": 1
        }
      },
      "inner_hits": {}    
    }
  }
}
```

| The inner hit definition like in the nested example. |
| ---------------------------------------------------- |

An example of a response snippet that could be generated from the above search request:

```json
{
  ...,
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "number": 1,
          "my_join_field": "my_parent"
        },
        "inner_hits": {
          "my_child": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.0,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "2",
                  "_score": 1.0,
                  "_routing": "1",
                  "_source": {
                    "number": 1,
                    "my_join_field": {
                      "name": "my_child",
                      "parent": "1"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

## 从搜索中检索指定字段

By default, each hit in the search response includes the document [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html), which is the entire JSON object that was provided when indexing the document. There are two recommended methods to retrieve selected fields from a search query:

- Use the [`fields` option](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#search-fields-param) to extract the values of fields present in the index mapping
- Use the [`_source` option](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering) if you need to access the original data that was passed at index time

You can use both of these methods, though the `fields` option is preferred because it consults both the document data and index mappings. In some instances, you might want to use [other methods](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#field-retrieval-methods) of retrieving data.

默认情况下，搜索响应中的每个命中包括文档[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)，这是索引文档时提供的整个JSON对象。有两种推荐的方法来检索搜索查询中的选定字段。

- 使用[`fields`选项](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#search-fields-param)来提取索引映射中存在的字段的值
- 如果你需要访问索引时传递的原始数据，使用[`_source`选项](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering)

你可以使用这两种方法，尽管`fields`选项是首选，因为它同时查询了文档数据和索引映射。在某些情况下，你可能想使用[其他方法](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#field-retrieval-methods)来检索数据。

### `fields` 选项

To retrieve specific fields in the search response, use the `fields` parameter. Because it consults the index mappings, the `fields` parameter provides several advantages over referencing the `_source` directly. Specifically, the `fields` parameter:

要检索搜索响应中的特定字段，请使用`fields`参数。因为查询索引映射，`fields`参数比直接引用`_source`有几个优势。具体来说，`fields`参数:

- 以符合其映射类型的标准化方式返回每个值
- 接受[多字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html)和[字段别名](https://www.elastic.co/guide/en/elasticsearch/reference/current/field-alias.html)
- 格式化日期和空间数据类型
- 检索[Runtime字段值](https://www.elastic.co/guide/en/elasticsearch/reference/current/runtime-retrieving-fields.html)
- 返回由脚本在索引时计算的字段

其他映射选项也被尊重，包括[`ignore_above`](https://www.elastic.co/guide/en/elasticsearch/reference/current/ignore-above.html), [`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/current/ignore-malformed.html), 和 [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html)。

`fields`选项返回值的方式与Elasticsearch的索引方式一致。对于标准字段，这意味着`fields`选项在`_source`中查找数值，然后使用映射来解析和格式化它们。

#### 搜索特定字段

The following search request uses the `fields` parameter to retrieve values for the `user.id` field, all fields starting with `http.response.`, and the `@timestamp` field.

下面的搜索请求使用`fields`参数来检索`user.id`字段的值，所有以`http.response.`开头的字段，以及`@timestamp`字段。

Using object notation, you can pass a `format` parameter for certain fields to apply a custom format for the field’s values:

- [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html) and [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html) fields accept a [date format](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html)
- [Spatial fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html#spatial_datatypes) accept either `geojson` for [GeoJSON](http://www.geojson.org/) (the default) or `wkt` for [Well Known Text](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)

Other field types do not support the `format` parameter.

使用对象符号，你可以为某些字段传递一个`format`参数，为字段的值应用一个自定义格式。

- [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)和[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html)字段接受[日期格式](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html)
- [空间字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html#spatial_datatypes)接受[GeoJSON](http://www.geojson.org/)的`geojson`（默认）或[Well Known Text](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)的`wkt`。

其他字段类型不支持`format`参数。

```console
POST my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "fields": [
    "user.id",
    "http.response.*",         
    {
      "field": "@timestamp",
      "format": "epoch_millis" 
    }
  ],
  "_source": false
}
```

|      | Both full field names and wildcard patterns are accepted.    |
| ---- | ------------------------------------------------------------ |
|      | Use the `format` parameter to apply a custom format for the field’s values. |

#### 响应总返回数组

The `fields` response always returns an array of values for each field, even when there is a single value in the `_source`. This is because Elasticsearch has no dedicated array type, and any field could contain multiple values. The `fields` parameter also does not guarantee that array values are returned in a specific order. See the mapping documentation on [arrays](https://www.elastic.co/guide/en/elasticsearch/reference/current/array.html) for more background.

The response includes values as a flat list in the `fields` section for each hit. Because the `fields` parameter doesn’t fetch entire objects, only leaf fields are returned.

`fields`响应总是为每个字段返回一个数值数组，即使在`_source`中只有一个数值。这是因为Elasticsearch没有专门的数组类型，任何字段都可能包含多个值。`fields`参数也不能保证数组值以特定顺序返回。更多的背景资料请参见[arrays](https://www.elastic.co/guide/en/elasticsearch/reference/current/array.html)上的映射文档。

响应包括每个命中的`fields`部分的平坦列表的值。因为 "fields "参数没有获取整个对象，只有叶子字段被返回。

```console-result
{
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_id" : "0",
        "_score" : 1.0,
        "_type" : "_doc",
        "fields" : {
          "user.id" : [
            "kimchy"
          ],
          "@timestamp" : [
            "4098435132000"
          ],
          "http.response.bytes": [
            1070000
          ],
          "http.response.status_code": [
            200
          ]
        }
      }
    ]
  }
}
```

#### 检索嵌套字段

[`嵌套`字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)的`字段`响应与普通对象字段的响应略有不同。普通的 `对象` 字段内的叶子值以平面列表形式返回，而 `嵌套 字段内的值则被分组，以保持原始嵌套数组内每个对象的独立性。对于嵌套字段数组中的每一个条目，其值也是以平面列表的形式返回，除非在父嵌套对象中还有其他的 "嵌套 "字段，在这种情况下，对更深的嵌套字段再次重复同样的程序。

给出以下映射，其中`user`是一个嵌套字段，在对以下文档进行索引并检索`user`字段下的所有字段后。

The `fields` response for [`nested` fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html) is slightly different from that of regular object fields. While leaf values inside regular `object` fields are returned as a flat list, values inside `nested` fields are grouped to maintain the independence of each object inside the original nested array. For each entry inside a nested field array, values are again returned as a flat list unless there are other `nested` fields inside the parent nested object, in which case the same procedure is repeated again for the deeper nested fields.

Given the following mapping where `user` is a nested field, after indexing the following document and retrieving all fields under the `user` field:

```console
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "group" : { "type" : "keyword" },
      "user": {
        "type": "nested",
        "properties": {
          "first" : { "type" : "keyword" },
          "last" : { "type" : "keyword" }
        }
      }
    }
  }
}

PUT my-index-000001/_doc/1?refresh=true
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

POST my-index-000001/_search
{
  "fields": ["*"],
  "_source": false
}
```

The response will group `first` and `last` name instead of returning them as a flat list.

```console-result
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
    "max_score": 1.0,
    "hits": [{
      "_index": "my-index-000001",
      "_id": "1",
      "_score": 1.0,
      "_type": "_doc",
      "fields": {
        "group" : ["fans"],
        "user": [{
            "first": ["John"],
            "last": ["Smith"],
          },
          {
            "first": ["Alice"],
            "last": ["White"],
          }
        ]
      }
    }]
  }
}
```

嵌套字段将按其嵌套路径分组，无论使用何种模式来检索它们。例如，如果你只查询前面例子中的`user.first`字段。

```apl
POST my-index-000001/_search
{
  "fields": ["user.first"],
  "_source": false
}
```

The response returns only the user’s first name, but still maintains the structure of the nested `user` array:

响应只返回用户的名字，但仍然保持嵌套的`user`数组的结构。

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
    "max_score": 1.0,
    "hits": [{
      "_index": "my-index-000001",
      "_id": "1",
      "_score": 1.0,
      "_type": "_doc",
      "fields": {
        "user": [{
            "first": ["John"],
          },
          {
            "first": ["Alice"],
          }
        ]
      }
    }]
  }
}
```

However, when the `fields` pattern targets the nested `user` field directly, no values will be returned because the pattern doesn’t match any leaf fields.

然而，当 "fields "模式直接针对嵌套的 "user "字段时，由于该模式不匹配任何叶子字段，所以不会返回任何值。

#### 检索未映射字符

By default, the `fields` parameter returns only values of mapped fields. However, Elasticsearch allows storing fields in `_source` that are unmapped, such as setting [dynamic field mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-field-mapping.html) to `false` or by using an object field with `enabled: false`. These options disable parsing and indexing of the object content.

To retrieve unmapped fields in an object from `_source`, use the `include_unmapped` option in the `fields` section:

默认情况下，`fields`参数只返回已映射字段的值。然而，Elasticsearch允许在`_source`中存储未映射的字段，例如将[动态字段映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-field-mapping.html)设置为`false`或通过使用`enabled: false`的对象字段。这些选项禁用了对对象内容的解析和索引。

**要从`_source`检索对象中未映射的字段，使用`fields`部分的`include_unmapped`选项。**

```console
PUT my-index-000001
{
  "mappings": {
    "enabled": false 
  }
}

PUT my-index-000001/_doc/1?refresh=true
{
  "user_id": "kimchy",
  "session_data": {
     "object": {
       "some_field": "some_value"
     }
   }
}

POST my-index-000001/_search
{
  "fields": [
    "user_id",
    {
      "field": "session_data.object.*",
      "include_unmapped" : true 
    }
  ],
  "_source": false
}
```

|      | Disable all mappings.                                |
| ---- | ---------------------------------------------------- |
|      | Include unmapped fields matching this field pattern. |

The response will contain field results under the `session_data.object.*` path, even if the fields are unmapped. The `user_id` field is also unmapped, but it won’t be included in the response because `include_unmapped` isn’t set to `true` for that field pattern.

响应将包含`session_data.object.*`路径下的字段结果，即使这些字段是未映射的。`user_id`字段也是未映射的，但它不会被包含在响应中，因为`include_unmapped`没有为该字段模式设置为`true`。

```console-result
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_id" : "1",
        "_score" : 1.0,
        "_type" : "_doc",
        "fields" : {
          "session_data.object.some_field": [
            "some_value"
          ]
        }
      }
    ]
  }
}
```

####  `_source` 参数

You can use the `_source` parameter to select what fields of the source are returned. This is called *source filtering*.

The following search API request sets the `_source` request body parameter to `false`. The document source is not included in the response.

你可以使用`_source`参数来选择返回源的哪些字段。这被称为*源过滤*。

下面的搜索API请求将`_source`请求主体参数设置为`false`。文档源不包括在响应中。

```console
GET /_search
{
  "_source": false,
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

To return only a subset of source fields, specify a wildcard (`*`) pattern in the `_source` parameter. The following search API request returns the source for only the `obj` field and its properties.

要想只返回源字段的一个子集，请在`_source`参数中指定一个通配符（`*`）模式。下面的搜索API请求只返回`obj`字段及其属性的源。

```console
GET /_search
{
  "_source": "obj.*",
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also specify an array of wildcard patterns in the `_source` field. The following search API request returns the source for only the `obj1` and `obj2` fields and their properties.

你也可以在`_source`字段中指定一个通配符模式的阵列。下面的搜索API请求只返回`obj1`和`obj2`字段及其属性的来源。

```console
GET /_search
{
  "_source": [ "obj1.*", "obj2.*" ],
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

For finer control, you can specify an object containing arrays of `includes` and `excludes` patterns in the `_source` parameter.

If the `includes` property is specified, only source fields that match one of its patterns are returned. You can exclude fields from this subset using the `excludes` property.

If the `includes` property is not specified, the entire document source is returned, excluding any fields that match a pattern in the `excludes` property.

The following search API request returns the source for only the `obj1` and `obj2` fields and their properties, excluding any child `description` fields.

为了更精细的控制，你可以在`_source`参数中指定一个包含`includes`和`excludes`模式数组的对象。

如果指定了`includes`属性，只有符合其模式之一的源字段才会被返回。你可以使用`excludes'属性将字段从这个子集中排除。

如果没有指定`includes`属性，则返回整个文档源，排除任何与`excludes`属性中的模式相匹配的字段。

下面的搜索API请求只返回`obj1`和`obj2`字段及其属性的源，不包括任何子`描述`字段。

```console
GET /_search
{
  "_source": {
    "includes": [ "obj1.*", "obj2.*" ],
    "excludes": [ "*.description" ]
  },
  "query": {
    "term": {
      "user.id": "kimchy"
    }
  }
}
```

### 其他获取data的方法

> **使用字段通常更好 Using `fields` is typically better**
>
> These options are usually not required. Using the `fields` option is typically the better choice, unless you absolutely need to force loading a stored or `docvalue_fields`.
>
> 这些选项通常不是必需的。使用字段选项通常是更好的选择，除非你绝对需要强制加载一个存储或docvalue_fields。

一个文档的`_source'在Lucene中被存储为一个字段。这种结构意味着整个`source`对象必须被加载和解析，即使你只请求它的一部分。为了避免这种限制，你可以尝试其他的选项来加载字段。

- 使用[`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#docvalue-fields)参数来获取选定字段的值。当返回相当少的支持doc值的字段时，这可能是一个不错的选择，例如关键词和日期。
- 使用[`stored_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-stored-fields)参数来获取特定存储字段的值（使用[`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html)映射选项的字段）。

Elasticsearch总是试图从`_source`加载数值。这种行为与源过滤的含义相同，Elasticsearch需要加载和解析整个`_source`来检索一个字段。

#### Doc value fields

You can use the [`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#docvalue-fields) parameter to return [doc values](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) for one or more fields in the search response.

你可以使用[`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#docvalue-fields)参数来返回搜索响应中一个或多个字段的[doc values](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html)。

Doc values store the same values as the `_source` but in an on-disk, column-based structure that’s optimized for sorting and aggregations. Since each field is stored separately, Elasticsearch only reads the field values that were requested and can avoid loading the whole document `_source`.

Doc values存储的值与`_source`相同，但在磁盘上，基于列的结构中，为排序和聚合进行了优化。由于每个字段是单独存储的，Elasticsearch只读取被请求的字段值，可以避免加载整个文档`_source`。

Doc values are stored for supported fields by default. However, doc values are not supported for [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) or [`text_annotated`](https://www.elastic.co/guide/en/elasticsearch/plugins/7.14/mapper-annotated-text-usage.html) fields.

默认情况下，支持的字段会存储Doc values。然而，[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)或[`text_annotated`](https://www.elastic.co/guide/en/elasticsearch/plugins/7.14/mapper-annotated-text-usage.html)字段不支持Doc values。

The following search request uses the `docvalue_fields` parameter to retrieve doc values for the `user.id` field, all fields starting with `http.response.`, and the `@timestamp` field:

下面的搜索请求使用`docvalue_fields`参数来检索`user.id`字段、所有以`http.response.`开头的字段和`@timestamp`字段的文档值。

```console
GET my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "docvalue_fields": [
    "user.id",
    "http.response.*", 
    {
      "field": "date",
      "format": "epoch_millis" 
    }
  ]
}
```

|      | Both full field names and wildcard patterns are accepted.    |
| ---- | ------------------------------------------------------------ |
|      | Using object notation, you can pass a `format` parameter to apply a custom format for the field’s doc values. [Date fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html) support a [date `format`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html). [Numeric fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html) support a [DecimalFormat pattern](https://docs.oracle.com/javase/8/docs/api/java/text/DecimalFormat.html). Other field datatypes do not support the `format` parameter. |

You cannot use the `docvalue_fields` parameter to retrieve doc values for nested objects. If you specify a nested object, the search returns an empty array (`[ ]`) for the field. To access nested fields, use the [`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/current/inner-hits.html) parameter’s `docvalue_fields` property.

你不能使用`docvalue_fields`参数来检索嵌套对象的文档值。如果你指定了一个嵌套的对象，搜索会返回一个空数组（`[ ]`）的字段。要访问嵌套字段，请使用[`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/current/inner-hits.html) 参数的`docvalue_fields`属性。

#### Stored fields

It’s also possible to store an individual field’s values by using the [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html) mapping option. You can use the `stored_fields` parameter to include these stored values in the search response.

The `stored_fields` parameter is for fields that are explicitly marked as stored in the mapping, which is off by default and generally not recommended. Use [source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering) instead to select subsets of the original source document to be returned.

Allows to selectively load specific stored fields for each document represented by a search hit.

也可以通过使用[`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html)的映射选项来存储单个字段的值。你可以使用`stored_fields`参数在搜索响应中包括这些存储的值。

`stored_fields`参数用于在映射中明确标记为存储的字段，默认是关闭的，一般不推荐使用。使用[source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering)代替，以选择返回原始源文件的子集。

允许有选择地对搜索结果所代表的每个文档加载特定的存储字段。

```console
GET /_search
{
  "stored_fields" : ["user", "postDate"],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

`*` can be used to load all stored fields from the document.

An empty array will cause only the `_id` and `_type` for each hit to be returned, for example:

```console
GET /_search
{
  "stored_fields" : [],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

If the requested fields are not stored (`store` mapping set to `false`), they will be ignored.

Stored field values fetched from the document itself are always returned as an array. On the contrary, metadata fields like `_routing` are never returned as an array.

Also only leaf fields can be returned via the `stored_fields` option. If an object field is specified, it will be ignored.

On its own, `stored_fields` cannot be used to load fields in nested objects — if a field contains a nested object in its path, then no data will be returned for that stored field. To access nested fields, `stored_fields` must be used within an [`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/current/inner-hits.html) block.

如果请求的字段没有被存储（`store`映射设置为`false`），它们将被忽略。

从文档本身获取的存储字段值总是以数组形式返回。相反，元数据字段如`_routing`永远不会以数组形式返回。

另外，只有叶子字段可以通过`stored_fields`选项返回。如果指定了一个对象字段，它将被忽略。

就其本身而言，`stored_fields`不能用于加载嵌套对象中的字段 - 如果一个字段在其路径中包含一个嵌套对象，那么该存储字段的数据将不会被返回。要访问嵌套字段，`stored_fields`必须在一个[`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/current/inner-hits.html)块中使用。

##### Disable stored fields

To disable the stored fields (and metadata fields) entirely use: `_none_`:

```console
GET /_search
{
  "stored_fields": "_none_",
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering) and [`version`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-version) parameters cannot be activated if `_none_` is used.

#### Script fields

You can use the `script_fields` parameter to retrieve a [script evaluation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html) (based on different fields) for each hit. For example:

```console
GET /_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value * 2"
      }
    },
    "test2": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value * params.factor",
        "params": {
          "factor": 2.0
        }
      }
    }
  }
}
```

Script fields can work on fields that are not stored (`price` in the above case), and allow to return custom values to be returned (the evaluated value of the script).

Script fields can also access the actual `_source` document and extract specific elements to be returned from it by using `params['_source']`. Here is an example:

```console
GET /_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": "params['_source']['message']"
    }
  }
}
```

Note the `_source` keyword here to navigate the json-like model.

It’s important to understand the difference between `doc['my_field'].value` and `params['_source']['my_field']`. The first, using the doc keyword, will cause the terms for that field to be loaded to memory (cached), which will result in faster execution, but more memory consumption. Also, the `doc[...]` notation only allows for simple valued fields (you can’t return a json object from it) and makes sense only for non-analyzed or single term based fields. However, using `doc` is still the recommended way to access values from the document, if at all possible, because `_source` must be loaded and parsed every time it’s used. Using `_source` is very slow.

## 通过集群搜索 暂时未看

**Cross-cluster search** lets you run a single search request against one or more [remote clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-remote-clusters.html). For example, you can use a cross-cluster search to filter and analyze log data stored on clusters in different data centers.

Cross-cluster search requires [remote clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-remote-clusters.html).

### Supported APIs

The following APIs support cross-cluster search:

- [Search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)
- [Async search](https://www.elastic.co/guide/en/elasticsearch/reference/current/async-search.html)
- [Multi search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html)
- [Search template](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template.html)
- [Multi search template](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-search-template.html)
- [Field capabilities](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-field-caps.html)
- [experimental] This functionality is experimental and may be changed or removed completely in a future release. Elastic will take a best effort approach to fix any issues, but experimental features are not subject to the support SLA of official GA features.[EQL search](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-search-api.html)

### Cross-cluster search examples

#### Remote cluster setup

To perform a cross-cluster search, you must have at least one remote cluster configured.

If you want to search across clusters in the cloud, you can [configure remote clusters on Elasticsearch Service](https://www.elastic.co/guide/en/cloud/current/ec-enable-ccs.html). Then, you can search across clusters and [set up cross-cluster replication](https://www.elastic.co/guide/en/elasticsearch/reference/current/ccr-getting-started.html).

The following [cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API request adds three remote clusters:`cluster_one`, `cluster_two`, and `cluster_three`.

```console
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_one": {
          "seeds": [
            "127.0.0.1:9300"
          ]
        },
        "cluster_two": {
          "seeds": [
            "127.0.0.1:9301"
          ]
        },
        "cluster_three": {
          "seeds": [
            "127.0.0.1:9302"
          ]
        }
      }
    }
  }
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/789.console) 

#### Search a single remote cluster

The following [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) API request searches the `my-index-000001` index on a single remote cluster, `cluster_one`.

```console
GET /cluster_one:my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "_source": ["user.id", "message", "http.response.status_code"]
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/790.console) 

The API returns the following response:

```console-result
{
  "took": 150,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0,
    "skipped": 0
  },
  "_clusters": {
    "total": 1,
    "successful": 1,
    "skipped": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "cluster_one:my-index-000001", 
        "_type": "_doc",
        "_id": "0",
        "_score": 1,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      }
    ]
  }
}
```



|      | The search response body includes the name of the remote cluster in the `_index` parameter. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### Search multiple remote clusters

The following [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html) API request searches the `my-index-000001` index on three clusters:

- Your local cluster
- Two remote clusters, `cluster_one` and `cluster_two`

```console
GET /my-index-000001,cluster_one:my-index-000001,cluster_two:my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "_source": ["user.id", "message", "http.response.status_code"]
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/791.console) 

The API returns the following response:

```console-result
{
  "took": 150,
  "timed_out": false,
  "num_reduce_phases": 4,
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0,
    "skipped": 0
  },
  "_clusters": {
    "total": 3,
    "successful": 3,
    "skipped": 0
  },
  "hits": {
    "total" : {
        "value": 3,
        "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "my-index-000001", 
        "_type": "_doc",
        "_id": "0",
        "_score": 2,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      },
      {
        "_index": "cluster_one:my-index-000001", 
        "_type": "_doc",
        "_id": "0",
        "_score": 1,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      },
      {
        "_index": "cluster_two:my-index-000001", 
        "_type": "_doc",
        "_id": "0",
        "_score": 1,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      }
    ]
  }
}
```



|      | This document’s `_index` parameter doesn’t include a cluster name. This means the document came from the local cluster. |
| ---- | ------------------------------------------------------------ |
|      | This document came from `cluster_one`.                       |
|      | This document came from `cluster_two`.                       |

### Skip unavailable clusters

By default, a cross-cluster search returns an error if **any** cluster in the request is unavailable.

To skip an unavailable cluster during a cross-cluster search, set the [`skip_unavailable`](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-remote-info.html#skip-unavailable) cluster setting to `true`.

The following [cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API request changes `cluster_two`'s `skip_unavailable` setting to `true`.

```console
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote.cluster_two.skip_unavailable": true
  }
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/792.console) 

If `cluster_two` is disconnected or unavailable during a cross-cluster search, Elasticsearch won’t include matching documents from that cluster in the final results.

### Selecting gateway and seed nodes in sniff mode

For remote clusters using the [sniff connection](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-remote-clusters.html#sniff-mode) mode, gateway and seed nodes need to be accessible from the local cluster via your network.

By default, any non-[master-eligible](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node) node can act as a gateway node. If wanted, you can define the gateway nodes for a cluster by setting `cluster.remote.node.attr.gateway` to `true`.

For cross-cluster search, we recommend you use gateway nodes that are capable of serving as [coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-node) for search requests. If wanted, the seed nodes for a cluster can be a subset of these gateway nodes.

### Cross-cluster search in proxy mode

[Proxy mode](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-remote-clusters.html#proxy-mode) remote cluster connections support cross-cluster search. All remote connections connect to the configured `proxy_address`. Any desired connection routing to gateway or [coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-node) must be implemented by the intermediate proxy at this configured address.

### How cross-cluster search handles network delays

Because cross-cluster search involves sending requests to remote clusters, any network delays can impact search speed. To avoid slow searches, cross-cluster search offers two options for handling network delays:

- **[Minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html#ccs-min-roundtrips)**

  By default, Elasticsearch reduces the number of network roundtrips between remote clusters. This reduces the impact of network delays on search speed. However, Elasticsearch can’t reduce network roundtrips for large search requests, such as those including a [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results) or [inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/current/inner-hits.html).See [Minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html#ccs-min-roundtrips) to learn how this option works.

- **[Don’t minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html#ccs-unmin-roundtrips)**

  For search requests that include a scroll or inner hits, Elasticsearch sends multiple outgoing and ingoing requests to each remote cluster. You can also choose this option by setting the [`ccs_minimize_roundtrips`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#ccs-minimize-roundtrips) parameter to `false`. While typically slower, this approach may work well for networks with low latency.See [Don’t minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html#ccs-unmin-roundtrips) to learn how this option works.

#### Minimize network roundtrips

Here’s how cross-cluster search works when you minimize network roundtrips.

1. You send a cross-cluster search request to your local cluster. A coordinating node in that cluster receives and parses the request.

   ![ccs min roundtrip client request](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-client-request.svg)

2. The coordinating node sends a single search request to each cluster, including the local cluster. Each cluster performs the search request independently, applying its own cluster-level settings to the request.

   ![ccs min roundtrip cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-cluster-search.svg)

3. Each remote cluster sends its search results back to the coordinating node.

   ![ccs min roundtrip cluster results](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-cluster-results.svg)

4. After collecting results from each cluster, the coordinating node returns the final results in the cross-cluster search response.

   ![ccs min roundtrip client response](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-client-response.svg)

#### Don’t minimize network roundtrips

Here’s how cross-cluster search works when you don’t minimize network roundtrips.

1. You send a cross-cluster search request to your local cluster. A coordinating node in that cluster receives and parses the request.

   ![ccs min roundtrip client request](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-client-request.svg)

2. The coordinating node sends a [search shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-shards.html) API request to each remote cluster.

   ![ccs min roundtrip cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-cluster-search.svg)

3. Each remote cluster sends its response back to the coordinating node. This response contains information about the indices and shards the cross-cluster search request will be executed on.

   ![ccs min roundtrip cluster results](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-cluster-results.svg)

4. The coordinating node sends a search request to each shard, including those in its own cluster. Each shard performs the search request independently.

   When network roundtrips aren’t minimized, the search is executed as if all data were in the coordinating node’s cluster. We recommend updating cluster-level settings that limit searches, such as `action.search.shard_count.limit`, `pre_filter_shard_size`, and `max_concurrent_shard_requests`, to account for this. If these limits are too low, the search may be rejected.

   ![ccs dont min roundtrip shard search](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-dont-min-roundtrip-shard-search.svg)

5. Each shard sends its search results back to the coordinating node.

   ![ccs dont min roundtrip shard results](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-dont-min-roundtrip-shard-results.svg)

6. After collecting results from each cluster, the coordinating node returns the final results in the cross-cluster search response.

   ![ccs min roundtrip client response](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ccs/ccs-min-roundtrip-client-response.svg)

### Supported configurations

Generally, [cross-cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-remote-clusters.html#gateway-nodes-selection) can search remote clusters that are one major version ahead or behind the coordinating node’s version.

For the [EQL search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-search-api.html), the local and remote clusters must use the same Elasticsearch version.

Cross-cluster search can also search remote clusters that are being [upgraded](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html) so long as both the "upgrade from" and "upgrade to" version are compatible with the gateway node.

For example, a coordinating node running Elasticsearch 5.6 can search a remote cluster running Elasticsearch 6.8, but that cluster can not be upgraded to 7.1. In this case you should first upgrade the coordinating node to 7.1 and then upgrade remote cluster.

Running multiple versions of Elasticsearch in the same cluster beyond the duration of an upgrade is not supported.

Only features that exist across all searched clusters are supported. Using a recent feature with a remote cluster where the feature is not supported will result in undefined behavior.



## 搜索多数据流||索引

To search multiple data streams and indices, add them as comma-separated values in the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)'s request path.

The following request searches the `my-index-000001` and `my-index-000002` indices.

```console
GET /my-index-000001,my-index-000002/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```



You can also search multiple data streams and indices using an index pattern.

The following request targets the `my-index-*` index pattern. The request searches any data streams or indices in the cluster that start with `my-index-`.

```console
GET /my-index-*/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```



To search all data streams and indices in a cluster, omit the target from the request path. Alternatively, you can use `_all` or `*`.

The following requests are equivalent and search all data streams and indices in the cluster.

```console
GET /_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}

GET /_all/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}

GET /*/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```



### 索引权重

When searching multiple indices, you can use the `indices_boost` parameter to boost results from one or more specified indices. This is useful when hits coming from some indices matter more than hits from other.

You cannot use `indices_boost` with data streams.

当搜索多个指数时，你可以使用`indices_boost`参数来提高一个或多个指定指数的结果。当来自某些指数的搜索结果比来自其他指数的搜索结果更重要时，这就很有用。

你不能对数据流使用`indices_boost`。

```console
GET /_search
{
  "indices_boost": [
    { "my-index-000001": 1.4 },
    { "my-index-000002": 1.3 }
  ]
}
```

Aliases and index patterns can also be used:

```apl
GET /_search
{
  "indices_boost": [
    { "my-alias":  1.4 },
    { "my-index*": 1.3 }
  ]
}
```

If multiple matches are found, the first match will be used. For example, if an index is included in `alias1` and matches the `my-index*` pattern, a boost value of `1.4` is applied.

## 搜索路由分片

为了增加搜索容量以及防止硬件故障，ES通过在多个node中复制多份分片来存储索引的数据副本。

当运行一个搜索请求时，Elasticsearch会选择一个包含索引数据副本的节点，并将搜索请求转发给该节点的分片。这个过程被称为*搜索分片路由*或*路由*。

### 适应性副本选择

By default, Elasticsearch uses *adaptive replica selection* to route search requests. This method selects an eligible node using [shard allocation awareness](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#shard-allocation-awareness) and the following criteria:

- Response time of prior requests between the coordinating node and the eligible node
- How long the eligible node took to run previous searches
- Queue size of the eligible node’s `search` [threadpool](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)

Adaptive replica selection is designed to decrease search latency. However, you can disable adaptive replica selection by setting `cluster.routing.use_adaptive_replica_selection` to `false` using the [cluster settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html). If disabled, Elasticsearch routes search requests using a round-robin method, which may result in slower searches.

默认情况下，Elasticsearch使用*adaptive replica selection*来路由搜索请求。这种方法使用[shard allocation awareness(分片分配意识)](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#shard-allocation-awareness)和以下标准来选择一个合格的节点。

- 协调节点和合格节点之间先前请求的响应时间
- 符合条件的节点运行先前的搜索需要多长时间
- 合格节点的 "搜索"[线程池](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)的队列大小

自适应副本选择是为了减少搜索延迟。然而，你可以通过使用[集群设置API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)将`cluster.routing.use_adaptive_replica_selection`设置为`false`来禁用自适应复制选择。如果禁用，Elasticsearch会使用循环方法路由搜索请求，这可能会导致较慢的搜索。

### 偏好设置

By default, adaptive replica selection chooses from all eligible nodes and shards. However, you may only want data from a local node or want to route searches to a specific node based on its hardware. Or you may want to send repeated searches to the same shard to take advantage of caching.

默认情况下，自适应副本选择会从所有符合条件的节点和分片中选择。然而，你可能只想要本地节点的数据，或者想要根据硬件将搜索路由到一个特定的节点。或者你可能想把重复的搜索发送到同一个分片，以利用缓存的优势。

To limit the set of nodes and shards eligible for a search request, use the search API’s [`preference`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-preference) query parameter.

For example, the following request searches `my-index-000001` with a `preference` of `_local`. This restricts the search to shards on the local node. If the local node contains no shard copies of the index’s data, the request uses adaptive replica selection to another eligible node as a fallback.

为了限制符合搜索请求的节点和分片的集合，使用搜索API的[`preference`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-preference)查询参数。

例如，下面的请求搜索`my-index-000001`，`preference`为`_local`。这将搜索限制在本地节点上的分片。如果本地节点不包含索引数据的分片副本，该请求会使用自适应复制选择到另一个符合条件的节点作为后备。

```console
GET /my-index-000001/_search?preference=_local
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also use the `preference` parameter to route searches to specific shards based on a provided string. If the cluster state and selected shards do not change, searches using the same `preference` string are routed to the same shards in the same order.

We recommend using a unique `preference` string, such as a user name or web session ID. This string cannot start with a `_`.

You can use this option to serve cached results for frequently used and resource-intensive searches. If the shard’s data doesn’t change, repeated searches with the same `preference` string retrieve results from the same [shard request cache](https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-request-cache.html). For time series use cases, such as logging, data in older indices is rarely updated and can be served directly from this cache.

你也可以使用`preference`参数，根据提供的字符串将搜索路由到特定的分片。如果集群状态和选定的分片没有改变，使用相同的`preference`字符串的搜索会以相同的顺序被路由到相同的分片。

我们建议使用一个独特的`preference`字符串，例如用户名或网络会话ID。这个字符串不能以"_"开头。

你可以使用这个选项来为经常使用的和资源密集型的搜索提供缓存的结果。如果分片的数据没有变化，用相同的`preference`字符串重复搜索，从同一个[分片请求缓存](https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-request-cache.html)检索结果。对于时间序列的用例，如日志，旧索引中的数据很少更新，可以直接从这个缓存中提供。

The following request searches `my-index-000001` with a `preference` string of `my-custom-shard-string`.

```console
GET /my-index-000001/_search?preference=my-custom-shard-string
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

If the cluster state or selected shards change, the same `preference` string may not route searches to the same shards in the same order. This can occur for a number of reasons, including shard relocations and shard failures. A node can also reject a search request, which Elasticsearch would re-route to another node.

如果集群状态或选定的分片改变，相同的 "偏好 "字符串可能不会以相同的顺序将搜索路由到相同的分片。这种情况可能会发生，原因有很多，包括分片重新定位和分片失败。一个节点也可以拒绝一个搜索请求，Elasticsearch会将其重新路由到另一个节点。

### 使用routing值

When you index a document, you can specify an optional [routing value](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-routing-field.html), which routes the document to a specific shard.

For example, the following indexing request routes a document using `my-routing-value`.

当你索引一个文档时，你可以指定一个可选的[路由值](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-routing-field.html)，它将文档路由到一个特定的分片。

例如，下面的索引请求使用`my-routing-value'来路由一个文档。

```console
POST /my-index-000001/_doc?routing=my-routing-value
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
```



You can use the same routing value in the search API’s `routing` query parameter. This ensures the search runs on the same shard used to index the document.

你可以在搜索API的`routing`查询参数中使用相同的路由值。这可以确保搜索在用于索引文档的同一分片上运行。

```console
GET /my-index-000001/_search?routing=my-routing-value
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```



You can also provide multiple comma-separated routing values:

```console
GET /my-index-000001/_search?routing=my-routing-value,my-routing-value-2
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

### 搜索并发和并行

By default, Elasticsearch doesn’t reject search requests based on the number of shards the request hits. However, hitting a large number of shards can significantly increase CPU and memory usage.

默认情况下，Elasticsearch不会根据请求所击中的分片数量来拒绝搜索请求。然而，点击大量的分片会大大增加CPU和内存的使用。

For tips on preventing indices with large numbers of shards, see [Avoid oversharding](https://www.elastic.co/guide/en/elasticsearch/reference/current/avoid-oversharding.html).

You can use the `max_concurrent_shard_requests` query parameter to control maximum number of concurrent shards a search request can hit per node. This prevents a single request from overloading a cluster. The parameter defaults to a maximum of `5`.

关于防止索引有大量分片的提示，请参见[避免过度分片](https://www.elastic.co/guide/en/elasticsearch/reference/current/avoid-oversharding.html)。

你可以使用 "max_concurrent_shard_requests "查询参数来控制每个节点上搜索请求可以击中的最大并发分片数。这可以防止单个请求使集群过载。该参数的默认值是最大的`5'。

```console
GET /my-index-000001/_search?max_concurrent_shard_requests=3
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```



You can also use the `action.search.shard_count.limit` cluster setting to set a search shard limit and reject requests that hit too many shards. You can configure `action.search.shard_count.limit` using the [cluster settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html).

你也可以使用`action.search.shard_count.limit`集群设置来设置搜索分片的限制，并拒绝遇到太多分片的请求。你可以使用[cluster settings API]（https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html）配置`action.search.shard_count.limit`。

## 搜索模板

A search template is a stored search you can run with different variables.

If you use Elasticsearch as a search backend, you can pass user input from a search bar as parameters for a search template. This lets you run searches without exposing Elasticsearch’s query syntax to your users.

If you use Elasticsearch for a custom application, search templates let you change your searches without modifying your app’s code.

搜索模板是一个存储的搜索，你可以用不同的变量运行。

如果你使用Elasticsearch作为搜索后端，你可以将用户从搜索栏的输入作为搜索模板的参数。这让你在运行搜索时不需要将Elasticsearch的查询语法暴露给你的用户。

如果你将Elasticsearch用于自定义应用程序，搜索模板可以让你在不修改应用程序代码的情况下改变你的搜索。

### 创建搜索模板

To create or update a search template, use the [create stored script API](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-stored-script-api.html).

要创建或更新一个搜索模板，请使用[创建存储脚本API](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-stored-script-api.html)。

The request’s `source` supports the same parameters as the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-search-api-request-body)'s request body. `source` also supports [Mustache](https://mustache.github.io/) variables, typically enclosed in double curly brackets: `{{my-var}}`. When you run a templated search, Elasticsearch replaces these variables with values from `params`.

Search templates must use a `lang` of `mustache`.

The following request creates a search template with an `id` of `my-search-template`.

Request的`source`支持与[搜索API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-search-api-request-body)的请求主体相同的参数。`source`也支持[Mustache](https://mustache.github.io/)的变量，通常用双大括号括起来：`{{my-var}}`。当你运行一个模板化的搜索时，Elasticsearch会用`params`中的值替换这些变量。

搜索模板必须使用`mustache`的`lang`。

下面的请求创建了一个搜索模板，`id`为`my-search-template`。

```console
PUT _scripts/my-search-template
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "match": {
          "message": "{{query_string}}"
        }
      },
      "from": "{{from}}",
      "size": "{{size}}"
    },
    "params": {
      "query_string": "My query string"
    }
  }
}
```

Elasticsearch stores search templates as Mustache [scripts](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html) in the cluster state. Elasticsearch compiles search templates in the `template` script context. Settings that limit or disable scripts also affect search templates.

Elasticsearch将搜索模板作为Mustache [scripts](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)存储在集群状态中。Elasticsearch在`template`脚本上下文中编译搜索模板。限制或禁用脚本的设置也会影响搜索模板。

### 验证搜索模板

To test a template with different `params`, use the [render search template API](https://www.elastic.co/guide/en/elasticsearch/reference/current/render-search-template-api.html).

```console
POST _render/template
{
  "id": "my-search-template",
  "params": {
    "query_string": "hello world",
    "from": 20,
    "size": 10
  }
}
```

When rendered, the template outputs a [search request body](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-search-api-request-body).

```console-result
{
  "template_output": {
    "query": {
      "match": {
        "message": "hello world"
      }
    },
    "from": "20",
    "size": "10"
  }
}
```



You can also use the API to test inline templates.

```console
POST _render/template
{
    "source": {
      "query": {
        "match": {
          "message": "{{query_string}}"
        }
      },
      "from": "{{from}}",
      "size": "{{size}}"
    },
  "params": {
    "query_string": "hello world",
    "from": 20,
    "size": 10
  }
}
```

### 执行模板搜索

To run a search with a search template, use the [search template API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template-api.html). You can specify different `params` with each request.

要用搜索模板运行搜索，请使用[搜索模板API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template-api.html)。你可以在每个请求中指定不同的`参数'

```console
GET my-index/_search/template
{
  "id": "my-search-template",
  "params": {
    "query_string": "hello world",
    "from": 0,
    "size": 10
  }
}
```

The response uses the same properties as the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)'s response.

```console-result
{
  "took": 36,
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
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "my-index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "message": "hello world"
        }
      }
    ]
  }
}
```

### 运行多模板化的搜索

要用一个请求运行多个模板化的搜索，请使用[多搜索模板API](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-search-template.html)。这些请求通常比多个单独的搜索开销小，速度快。

```console
GET my-index/_msearch/template
{ }
{ "id": "my-search-template", "params": { "query_string": "hello world", "from": 0, "size": 10 }}
{ }
{ "id": "my-other-search-template", "params": { "query_type": "match_all" }}
```

### 获取搜索模板

To retrieve a search template, use the [get stored script API](https://www.elastic.co/guide/en/elasticsearch/reference/current/get-stored-script-api.html).

```console
GET _scripts/my-search-template
```

To get a list of all search templates and other stored scripts, use the [cluster state API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state.html).

要获得所有搜索模板和其他存储脚本的列表，请使用[集群状态API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state.html)

```console
GET _cluster/state/metadata?pretty&filter_path=metadata.stored_scripts
```

### 删除搜索模板

To delete a search template, use the [delete stored script API](https://www.elastic.co/guide/en/elasticsearch/reference/current/delete-stored-script-api.html).

```console
DELETE _scripts/my-search-template
```

### 设置默认值

To set a default value for a variable, use the following syntax:

```mustache
{{my-var}}{{^my-var}}default value{{/my-var}}
```

If a templated search doesn’t specify a value in its `params`, the search uses the default value instead. For example, the following template sets defaults for `from` and `size`.

如果一个模板化的搜索没有在它的`params`中指定一个值，搜索会使用默认值来代替。例如，下面的模板为`from`和`size`设置默认值。

```console
POST _render/template
{
  "source": {
    "query": {
      "match": {
        "message": "{{query_string}}"
      }
    },
    "from": "{{from}}{{^from}}0{{/from}}",
    "size": "{{size}}{{^size}}10{{/size}}"
  },
  "params": {
    "query_string": "hello world"
  }
}
```

### URL字符串编码

Use the `{{#url}}` function to URL encode a string.

```console
POST _render/template
{
  "source": {
    "query": {
      "term": {
        "url.full": "{{#url}}{{host}}/{{page}}{{/url}}"
      }
    }
  },
  "params": {
    "host": "http://example.com",
    "page": "hello-world"
  }
}
```

The template renders as:

```console-result
{
  "template_output": {
    "query": {
      "term": {
        "url.full": "http%3A%2F%2Fexample.com%2Fhello-world"
      }
    }
  }
}
```



### 连接值 Concatenate values

Use the `{{#join}}` function to concatenate array values as a comma-delimited string. For example, the following template concatenates two email addresses.

使用`{{#join}}`函数将数组值连接成以逗号分隔的字符串。例如，下面的模板将两个电子邮件地址连接起来。

```console
POST _render/template
{
  "source": {
    "query": {
      "match": {
        "user.group.emails": "{{#join}}emails{{/join}}"
      }
    }
  },
  "params": {
    "emails": [ "user1@example.com", "user_one@example.com" ]
  }
}
```

The template renders as:

```console-result
{
  "template_output": {
    "query": {
      "match": {
        "user.group.emails": "user1@example.com,user_one@example.com"
      }
    }
  }
}
```



You can also specify a custom delimiter.

```console
POST _render/template
{
  "source": {
    "query": {
      "range": {
        "user.effective.date": {
          "gte": "{{date.min}}",
          "lte": "{{date.max}}",
          "format": "{{#join delimiter='||'}}date.formats{{/join delimiter='||'}}"
	      }
      }
    }
  },
  "params": {
    "date": {
      "min": "2098",
      "max": "06/05/2099",
      "formats": ["dd/MM/yyyy", "yyyy"]
    }
  }
}
```



Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/815.console) 

The template renders as:

```console-result
{
  "template_output": {
    "query": {
      "range": {
        "user.effective.date": {
          "gte": "2098",
          "lte": "06/05/2099",
          "format": "dd/MM/yyyy||yyyy"
        }
      }
    }
  }
}
```



### 转成JSON

Use the `{{#toJson}}` function to convert a variable value to its JSON representation.

For example, the following template uses `{{#toJson}}` to pass an array. To ensure the request body is valid JSON, the `source` is written in the string format.

使用`{{#toJson}}函数将变量值转换为JSON表示。

例如，下面的模板使用`{#toJson}}`来传递一个数组。为了确保请求体是有效的JSON，`source`被写成字符串格式。

```console
POST _render/template
{
  "source": "{ \"query\": { \"terms\": { \"tags\": {{#toJson}}tags{{/toJson}} }}}",
  "params": {
    "tags": [
      "prod",
      "es01"
    ]
  }
}
```

The template renders as:

```console-result
{
  "template_output": {
    "query": {
      "terms": {
        "tags": [
          "prod",
          "es01"
        ]
      }
    }
  }
}
```



You can also use `{{#toJson}}` to pass objects.

```console
POST _render/template
{
  "source": "{ \"query\": {{#toJson}}my_query{{/toJson}} }",
  "params": {
    "my_query": {
      "match_all": { }
    }
  }
}
```

The template renders as:

```console-result
{
  "template_output" : {
    "query" : {
      "match_all" : { }
    }
  }
}
```



You can also pass an array of objects.

```console
POST _render/template
{
  "source": "{ \"query\": { \"bool\": { \"must\": {{#toJson}}clauses{{/toJson}} }}}",
  "params": {
    "clauses": [
      {
        "term": {
          "user.id": "kimchy"
        }
      },
      {
        "term": {
          "url.domain": "example.com"
        }
      }
    ]
  }
}
```

The template renders as:

```console-result
{
  "template_output": {
    "query": {
      "bool": {
        "must": [
          {
            "term": {
              "user.id": "kimchy"
            }
          },
          {
            "term": {
              "url.domain": "example.com"
            }
          }
        ]
      }
    }
  }
}
```



### 使用条件Use conditions

To create if conditions, use the following syntax:

```mustache
{{#condition}}content{{/condition}}
```

如果条件变量是`true`，Elasticsearch会显示其内容。例如，如果`year_scope`是`true`，下面的模板会搜索过去一年的数据。

```console
POST _render/template
{
  "source": "{ \"query\": { \"bool\": { \"filter\": [ {{#year_scope}} { \"range\": { \"@timestamp\": { \"gte\": \"now-1y/d\", \"lt\": \"now/d\" } } }, {{/year_scope}} { \"term\": { \"user.id\": \"{{user_id}}\" }}]}}}",
  "params": {
    "year_scope": true,
    "user_id": "kimchy"
  }
}
```

The template renders as:

```console-result
{
  "template_output" : {
    "query" : {
      "bool" : {
        "filter" : [
          {
            "range" : {
              "@timestamp" : {
                "gte" : "now-1y/d",
                "lt" : "now/d"
              }
            }
          },
          {
            "term" : {
              "user.id" : "kimchy"
            }
          }
        ]
      }
    }
  }
}
```



If `year_scope` is `false`, the template searches data from any time period.

```console
POST _render/template
{
  "source": "{ \"query\": { \"bool\": { \"filter\": [ {{#year_scope}} { \"range\": { \"@timestamp\": { \"gte\": \"now-1y/d\", \"lt\": \"now/d\" } } }, {{/year_scope}} { \"term\": { \"user.id\": \"{{user_id}}\" }}]}}}",
  "params": {
    "year_scope": false,
    "user_id": "kimchy"
  }
}
```

The template renders as:

```console-result
{
  "template_output" : {
    "query" : {
      "bool" : {
        "filter" : [
          {
            "term" : {
              "user.id" : "kimchy"
            }
          }
        ]
      }
    }
  }
}
```



To create if-else conditions, use the following syntax:

```mustache
{{#condition}}if content{{/condition}} {{^condition}}else content{{/condition}}
```



For example, the following template searches data from the past year if `year_scope` is `true`. Otherwise, it searches data from the past day.

```console
POST _render/template
{
  "source": "{ \"query\": { \"bool\": { \"filter\": [ { \"range\": { \"@timestamp\": { \"gte\": {{#year_scope}} \"now-1y/d\" {{/year_scope}} {{^year_scope}} \"now-1d/d\" {{/year_scope}} , \"lt\": \"now/d\" }}}, { \"term\": { \"user.id\": \"{{user_id}}\" }}]}}}",
  "params": {
    "year_scope": true,
    "user_id": "kimchy"
  }
}
```

## 搜索结果排序

Allows you to add one or more sorts on specific fields. Each sort can be reversed as well. The sort is defined on a per field level, with special field name for `_score` to sort by score, and `_doc` to sort by index order.

允许你在特定字段上添加一个或多个排序。每个排序也可以反转。排序是在每个字段层面上定义的，`_score`的特殊字段名是按分数排序，`_doc`是按索引顺序排序。

Assuming the following index mapping:

```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "post_date": { "type": "date" },
      "user": {
        "type": "keyword"
      },
      "name": {
        "type": "keyword"
      },
      "age": { "type": "integer" }
    }
  }
}
```

```console
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"order" : "asc", "format": "strict_date_optional_time_nanos"}},
    "user",
    { "name" : "desc" },
    { "age" : "desc" },
    "_score"
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

`_doc` has no real use-case besides being the most efficient sort order. So if you don’t care about the order in which documents are returned, then you should sort by `_doc`. This especially helps when [scrolling](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results).

`_doc`除了是最有效的排序顺序外，没有真正的用途。因此，如果你不关心文件的返回顺序，那么你应该按`_doc`排序。这在[滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results)时特别有用。

### 排序values

The search response includes `sort` values for each document. Use the `format` parameter to specify a [date format](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#built-in-date-formats) for the `sort` values of [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html) and [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html) fields. The following search returns `sort` values for the `post_date` field in the `strict_date_optional_time_nanos` format.

搜索响应包括每个文件的`sort`值。使用`format`参数为[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)和[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html)字段的`sort`值指定一个[date格式](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#built-in-date-formats)。下面的搜索以`strict_date_optional_time_nanos`格式返回`post_date`字段的`排序`值。	

```console
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"format": "strict_date_optional_time_nanos"}}
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

### 排序顺序

The `order` option can have the following values:

| `asc`  | Sort in ascending order  |
| ------ | ------------------------ |
| `desc` | Sort in descending order |

The order defaults to `desc` when sorting on the `_score`, and defaults to `asc` when sorting on anything else.

### 排序模式的参数

Elasticsearch supports sorting by array or multi-valued fields. The `mode` option controls what array value is picked for sorting the document it belongs to. The `mode` option can have the following values:

| `min`    | Pick the lowest value.                                       |
| -------- | ------------------------------------------------------------ |
| `max`    | Pick the highest value.                                      |
| `sum`    | Use the sum of all values as sort value. Only applicable for number based array fields. |
| `avg`    | Use the average of all values as sort value. Only applicable for number based array fields. |
| `median` | Use the median of all values as sort value. Only applicable for number based array fields. |

Elasticsearch支持通过数组或多值字段进行排序。`mode'选项控制在排序时选择哪一个数组值，以确定其所属的文档。`mode`选项可以有以下值。

| `min`    | 挑选最低的值。                                             |
| -------- | ---------------------------------------------------------- |
| `max`    | 挑选最高值。                                               |
| `sum`    | 使用所有值的总和作为排序值。只适用于基于数字的数组字段。   |
| `avg`    | 使用所有值的平均值作为排序值。只适用于基于数字的数组字段。 |
| `median` | 使用所有值的中值作为排序值。只适用于基于数字的数组字段。   |

The default sort mode in the ascending sort order is `min` — the lowest value is picked. The default sort mode in the descending order is `max` — the highest value is picked.

在升序排序中，默认的排序模式是 `min`--挑选最低值。降序的默认排序模式是 `max`--选择最高值。

#### 排序模式用例

In the example below the field price has multiple prices per document. In this case the result hits will be sorted by price ascending based on the average price per document.

在下面的例子中，每个文件的价格字段有多个价格。在这种情况下，结果的点击率将根据每份文件的平均价格，按价格升序排序。

```console
PUT /my-index-000001/_doc/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

### 数字字段排序

For numeric fields it is also possible to cast the values from one type to another using the `numeric_type` option. This option accepts the following values: [`"double", "long", "date", "date_nanos"`] and can be useful for searches across multiple data streams or indices where the sort field is mapped differently.

对于数字字段，也可以使用`numeric_type`选项将数值从一种类型转换成另一种类型。这个选项接受以下值。[`"double", "long", "date", "date_nanos"`]，对于在多个数据流或索引中搜索，排序字段的映射是不同的，这很有用。

Consider for instance these two indices:

```console
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "double" }
    }
  }
}
```

```console
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "long" }
    }
  }
}
```

Since `field` is mapped as a `double` in the first index and as a `long` in the second index, it is not possible to use this field to sort requests that query both indices by default. However you can force the type to one or the other with the `numeric_type` option in order to force a specific type for all indices:

由于 `field`在第一个索引中被映射为`double`，在第二个索引中被映射为 `long`，因此不可能使用这个字段对查询两个索引的请求进行默认排序。然而，你可以用`numeric_type`选项将类型强制为一个或另一个，以便为所有索引强制使用特定的类型。

```console
POST /index_long,index_double/_search
{
   "sort" : [
      {
        "field" : {
            "numeric_type" : "double"
        }
      }
   ]
}
```

In the example above, values for the `index_long` index are casted to a double in order to be compatible with the values produced by the `index_double` index. It is also possible to transform a floating point field into a `long` but note that in this case floating points are replaced by the largest value that is less than or equal (greater than or equal if the value is negative) to the argument and is equal to a mathematical integer.

This option can also be used to convert a `date` field that uses millisecond resolution to a `date_nanos` field with nanosecond resolution. Consider for instance these two indices:

在上面的例子中，为了与`index_double`索引产生的值兼容，`index_long`索引的值被铸成了一个double。也可以将一个float字段转换为`long`，但是注意在这种情况下，float被替换为小于或等于（如果是负值，则大于或等于）参数的最大值，并且等于一个数学整数。

这个选项也可以用来将一个使用毫秒分辨率的`date`字段转换成纳秒分辨率的`date_nanos`字段。例如，考虑这两个指数。

```console
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "date" }
    }
  }
}
```

```console
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "date_nanos" }
    }
  }
}
```

Values in these indices are stored with different resolutions so sorting on these fields will always sort the `date` before the `date_nanos` (ascending order). With the `numeric_type` type option it is possible to set a single resolution for the sort, setting to `date` will convert the `date_nanos` to the millisecond resolution while `date_nanos` will convert the values in the `date` field to the nanoseconds resolution:

```console
POST /index_long,index_double/_search
{
   "sort" : [
      {
        "field" : {
            "numeric_type" : "date_nanos"
        }
      }
   ]
}
```

To avoid overflow, the conversion to `date_nanos` cannot be applied on dates before 1970 and after 2262 as nanoseconds are represented as longs.

### 嵌套对象中排序

Elasticsearch also supports sorting by fields that are inside one or more nested objects. The sorting by nested field support has a `nested` sort option with the following properties:

- **`path`**

  Defines on which nested object to sort. The actual sort field must be a direct field inside this nested object. When sorting by nested field, this field is mandatory.

- **`filter`**

  A filter that the inner objects inside the nested path should match with in order for its field values to be taken into account by sorting. Common case is to repeat the query / filter inside the nested filter or query. By default no `nested_filter` is active.

- **`max_children`**

  The maximum number of children to consider per root document when picking the sort value. Defaults to unlimited.

- **`nested`**

  Same as top-level `nested` but applies to another nested path within the current nested object.

Elasticsearch还支持按一个或多个嵌套对象中的字段进行排序。支持按嵌套字段排序有一个`neste`排序选项，其属性如下。

- **`path`**

  定义对哪个嵌套对象进行排序。实际的排序字段必须是这个嵌套对象内部的直接字段。当按嵌套字段排序时，这个字段是必须的。

- **`filter`**

  一个过滤器，嵌套路径中的内部对象应该与之匹配，以便在排序时考虑到其字段值。常见的情况是在嵌套的过滤器或查询中重复查询/过滤。默认情况下，没有激活`nested_filter`。

- **`max_children`**

  在选择排序值时，每个根文件要考虑的最大子文件夹数量。默认为无限。

- **`nested`**

  与顶层的`nested`相同，但适用于当前嵌套

#### Elasticsearch 6.1之前的嵌套排序选项

`nested_path`和`nested_filter`选项已经被废弃，而采用上面记录的选项。

#### 嵌套排序示例

In the below example `offer` is a field of type `nested`. The nested `path` needs to be specified; otherwise, Elasticsearch doesn’t know on what nested level sort values need to be captured.

在下面的例子中，`offer`是一个`nested`类型的字段。嵌套的`path`需要被指定；否则，Elasticsearch不知道需要在哪个嵌套层次上捕获排序值。

```console
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
       {
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             "nested": {
                "path": "offer",
                "filter": {
                   "term" : { "offer.color" : "blue" }
                }
             }
          }
       }
    ]
}
```

In the below example `parent` and `child` fields are of type `nested`. The `nested_path` needs to be specified at each level; otherwise, Elasticsearch doesn’t know on what nested level sort values need to be captured.

在下面的例子中，`parent`和`child`字段是`nested`类型。`nested_path`需要在每一级指定；否则，Elasticsearch不知道需要在哪个嵌套级别上捕获排序值。

```console
POST /_search
{
   "query": {
      "nested": {
         "path": "parent",
         "query": {
            "bool": {
                "must": {"range": {"parent.age": {"gte": 21}}},
                "filter": {
                    "nested": {
                        "path": "parent.child",
                        "query": {"match": {"parent.child.name": "matt"}}
                    }
                }
            }
         }
      }
   },
   "sort" : [
      {
         "parent.child.age" : {
            "mode" :  "min",
            "order" : "asc",
            "nested": {
               "path": "parent",
               "filter": {
                  "range": {"parent.age": {"gte": 21}}
               },
               "nested": {
                  "path": "parent.child",
                  "filter": {
                     "match": {"parent.child.name": "matt"}
                  }
               }
            }
         }
      }
   ]
}
```

Nested sorting is also supported when sorting by scripts and sorting by geo distance.

### 缺失值

The `missing` parameter specifies how docs which are missing the sort field should be treated: The `missing` value can be set to `_last`, `_first`, or a custom value (that will be used for missing docs as the sort value). The default is `_last`.

`missing`参数指定如何处理缺少sort字段的文件。`missing`值可以设置为 `_last`、`_first`或一个自定义值（将用于遗漏的文档作为排序值）。默认是`_last`。

For example:

```console
GET /_search
{
  "sort" : [
    { "price" : {"missing" : "_last"} }
  ],
  "query" : {
    "term" : { "product" : "chocolate" }
  }
}
```

If a nested inner object doesn’t match with the `nested_filter` then a missing value is used.

### 忽略unmapped字段

By default, the search request will fail if there is no mapping associated with a field. The `unmapped_type` option allows you to ignore fields that have no mapping and not sort by them. The value of this parameter is used to determine what sort values to emit. Here is an example of how it can be used:

默认情况下，如果没有与某个字段相关的映射，搜索请求将失败。`unmapped_type`选项允许你忽略没有映射的字段，不按它们排序。这个参数的值被用来决定要发出什么样的排序值。下面是一个关于如何使用它的例子。

```console
GET /_search
{
  "sort" : [
    { "price" : {"unmapped_type" : "long"} }
  ],
  "query" : {
    "term" : { "product" : "chocolate" }
  }
}
```

If any of the indices that are queried doesn’t have a mapping for `price` then Elasticsearch will handle it as if there was a mapping of type `long`, with all documents in this index having no value for this field.

### Geo 距离排序

Allow to sort by `_geo_distance`. Here is an example, assuming `pin.location` is a field of type `geo_point`:

```console
GET /_search
{
  "sort" : [
    {
      "_geo_distance" : {
          "pin.location" : [-70, 40],
          "order" : "asc",
          "unit" : "km",
          "mode" : "min",
          "distance_type" : "arc",
          "ignore_unmapped": true
      }
    }
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

- **`distance_type`**

  How to compute the distance. Can either be `arc` (default), or `plane` (faster, but inaccurate on long distances and close to the poles).

- **`mode`**

  What to do in case a field has several geo points. By default, the shortest distance is taken into account when sorting in ascending order and the longest distance when sorting in descending order. Supported values are `min`, `max`, `median` and `avg`.

- **`unit`**

  The unit to use when computing sort values. The default is `m` (meters).

- **`ignore_unmapped`**

  Indicates if the unmapped field should be treated as a missing value. Setting it to `true` is equivalent to specifying an `unmapped_type` in the field sort. The default is `false` (unmapped field cause the search to fail).

geo distance sorting does not support configurable missing values: the distance will always be considered equal to `Infinity` when a document does not have values for the field that is used for distance computation.

The following formats are supported in providing the coordinates:

#### Lat Lon as Properties

```console
GET /_search
{
  "sort" : [
    {
      "_geo_distance" : {
        "pin.location" : {
          "lat" : 40,
          "lon" : -70
        },
        "order" : "asc",
        "unit" : "km"
      }
    }
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

#### Lat Lon as String

Format in `lat,lon`.

```console
GET /_search
{
  "sort": [
    {
      "_geo_distance": {
        "pin.location": "40,-70",
        "order": "asc",
        "unit": "km"
      }
    }
  ],
  "query": {
    "term": { "user": "kimchy" }
  }
}
```

#### Geohash

```console
GET /_search
{
  "sort": [
    {
      "_geo_distance": {
        "pin.location": "drm3btev3e86",
        "order": "asc",
        "unit": "km"
      }
    }
  ],
  "query": {
    "term": { "user": "kimchy" }
  }
}
```

#### Lat Lon as Array

Format in `[lon, lat]`, note, the order of lon/lat here in order to conform with [GeoJSON](http://geojson.org/).

```console
GET /_search
{
  "sort": [
    {
      "_geo_distance": {
        "pin.location": [ -70, 40 ],
        "order": "asc",
        "unit": "km"
      }
    }
  ],
  "query": {
    "term": { "user": "kimchy" }
  }
}
```

### Multiple reference points

Multiple geo points can be passed as an array containing any `geo_point` format, for example

```console
GET /_search
{
  "sort": [
    {
      "_geo_distance": {
        "pin.location": [ [ -70, 40 ], [ -71, 42 ] ],
        "order": "asc",
        "unit": "km"
      }
    }
  ],
  "query": {
    "term": { "user": "kimchy" }
  }
}
```



and so forth.

The final distance for a document will then be `min`/`max`/`avg` (defined via `mode`) distance of all points contained in the document to all points given in the sort request.

### Script Based Sorting

Allow to sort based on custom scripts, here is an example:

```console
GET /_search
{
  "query": {
    "term": { "user": "kimchy" }
  },
  "sort": {
    "_script": {
      "type": "number",
      "script": {
        "lang": "painless",
        "source": "doc['field_name'].value * params.factor",
        "params": {
          "factor": 1.1
        }
      },
      "order": "asc"
    }
  }
}
```

### Track Scores

When sorting on a field, scores are not computed. By setting `track_scores` to true, scores will still be computed and tracked.

```console
GET /_search
{
  "track_scores": true,
  "sort" : [
    { "post_date" : {"order" : "desc"} },
    { "name" : "desc" },
    { "age" : "desc" }
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

### Memory Considerations

When sorting, the relevant sorted field values are loaded into memory. This means that per shard, there should be enough memory to contain them. For string based types, the field sorted on should not be analyzed / tokenized. For numeric types, if possible, it is recommended to explicitly set the type to narrower types (like `short`, `integer` and `float`).



