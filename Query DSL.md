# DSL查询

Elasticsearch提供了一个基于JSON的完整的查询DSL（领域专用语言）来定义查询。把查询DSL看作是查询的AST（抽象语法树），由两种类型的条款组成。

- **叶子查询字句**

  叶子查询子句在一个特定的字段中寻找一个特定的值，例如[`match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html), [`term`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html) 或 [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html) 查询。这些查询可以单独使用。

- **复合查询子句**

  复合查询子句包裹其他叶子**或**复合查询，用于以逻辑方式组合多个查询（如[`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)或[`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-dis-max-query.html)查询），或改变其行为（如[`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-constant-score-query.html) 查询）。

查询子句的行为取决于它们是在[查询上下文还是过滤上下文](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html)中使用。

- **允许昂贵的查询**

  某些类型的查询由于其实现方式，通常会执行得很慢，这可能会影响集群的稳定性。这些查询可以分为以下几类。

  - 需要线性扫描的查询：
    - 脚本查询
  - 前期成本查询：
    - [`fuzzy` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields)
    - [`regexp` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-regexp-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields)
    - [`prefix` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields or those without [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-prefixes.html))
    - [`wildcard` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-wildcard-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields)
    - [`range` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html) on [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) and [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html) fields

- [连接查询]

- Queries on [deprecated geo-shapes](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-shape.html#prefix-trees)

- 可能有较高的单文件成本的查询：

  - [`script_score` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-score-query.html)
  - [`percolate` queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-percolate-query.html)

通过将`search.allow_expensive_queries`设置为`false`（默认为`true`），可以防止执行此类查询。

## 查询和过滤contex

### 相关性评分

By default, Elasticsearch sorts matching search results by **relevance score**, which measures how well each document matches a query.

The relevance score is a positive floating point number, returned in the `_score` metadata field of the [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html) API. The higher the `_score`, the more relevant the document. While each query type can calculate relevance scores differently, score calculation also depends on whether the query clause is run in a **query** or **filter** context.

默认情况下，Elasticsearch按照**相关性分数**对匹配的搜索结果进行排序，它衡量每个文档与查询的匹配程度。

相关性分数是一个正的浮点数，在[search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)的`_score`元数据字段中返回。API。`_score'越高，文档就越相关。虽然每种查询类型可以以不同的方式计算相关性分数，但分数的计算也取决于查询条款是在**查询**还是**过滤器**的情况下运行。

### 查询context

In the query context, a query clause answers the question “*How well does this document match this query clause?*” Besides deciding whether or not the document matches, the query clause also calculates a relevance score in the `_score` metadata field.

Query context is in effect whenever a query clause is passed to a `query` parameter, such as the `query` parameter in the [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-query) API.

在查询上下文中，查询子句回答了 "*这个文档与这个查询子句的匹配程度如何？"的问题。除了决定文档是否匹配，查询子句还在`_score`元数据字段中计算相关性分数。

只要查询子句被传递给`query`参数，例如[search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-query)API中的`query`参数，查询上下文就会生效。

### 过滤context

In a filter context, a query clause answers the question “*Does this document match this query clause?*” The answer is a simple Yes or No — no scores are calculated. Filter context is mostly used for filtering structured data, e.g.

- *Does this `timestamp` fall into the range 2015 to 2016?*
- *Is the `status` field set to `"published"`*?

Frequently used filters will be cached automatically by Elasticsearch, to speed up performance.

Filter context is in effect whenever a query clause is passed to a `filter` parameter, such as the `filter` or `must_not` parameters in the [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) query, the `filter` parameter in the [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-constant-score-query.html) query, or the [`filter`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-filter-aggregation.html) aggregation.

在一个过滤器的上下文中，一个查询子句回答了 "*这个文档是否与这个查询子句相匹配 "的问题。答案是一个简单的 "是 "或 "否"--不计算分数。过滤器上下文主要用于过滤结构化的数据，例如。

- *这个`时间戳'是否属于2015到2016的范围？
- *"状态 "字段是否设置为 "发布 "*？

经常使用的过滤器会被Elasticsearch自动缓存，以加快性能。

只要查询条款被传递给`过滤器`参数，如[`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)查询中的`过滤器`或`must_not`参数，[`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-constant-score-query.html)查询中的`过滤器`参数，或[`过滤器`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-filter-aggregation.html)聚合，过滤器上下文就生效了。

### 查询和过滤示例

Below is an example of query clauses being used in query and filter context in the `search` API. This query will match documents where all of the following conditions are met:

- The `title` field contains the word `search`.
- The `content` field contains the word `elasticsearch`.
- The `status` field contains the exact word `published`.
- The `publish_date` field contains a date from 1 Jan 2015 onwards.

```console
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```



|      | The `query` parameter indicates query context.               |
| ---- | ------------------------------------------------------------ |
|      | The `bool` and two `match` clauses are used in query context, which means that they are used to score how well each document matches. |
|      | The `filter` parameter indicates filter context. Its `term` and `range` clauses are used in filter context. They will filter out documents which do not match, but they will not affect the score for matching documents. |

Scores calculated for queries in query context are represented as single precision floating point numbers; they have only 24 bits for significand’s precision. Score calculations that exceed the significand’s precision will be converted to floats with loss of precision.

Use query clauses in query context for conditions which should affect the score of matching documents (i.e. how well does the document match), and use all other query clauses in filter context.

| `query`参数表示查询背景。                                    |
| ------------------------------------------------------------ |
| `bool`和两个`match`子句用于查询上下文，这意味着它们被用来对每个文档的匹配程度进行评分。 |
| `filter`参数表示过滤上下文。它的`term'和`range'子句被用于过滤上下文。它们将过滤掉不匹配的文档，但不会影响匹配文档的得分。 |

在查询上下文中为查询计算的分数以单精度浮点数表示；它们只有24位的显值精度。超过显著值精度的分数计算将被转换为浮点数，并损失精度。

在查询上下文中使用查询条款，这些条款应该影响匹配文档的得分（即文档的匹配程度）

## 复合查询

Compound queries wrap other compound or leaf queries, either to combine their results and scores, to change their behaviour, or to switch from query to filter context.

复合查询包裹着其他复合查询或叶子查询，要么是为了结合它们的结果和分数，要么是为了改变它们的行为，要么是为了从查询切换到过滤器的环境。

The queries in this group are:

- **[`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)**

  The default query for combining multiple leaf or compound query clauses, as `must`, `should`, `must_not`, or `filter` clauses. The `must` and `should` clauses have their scores combined — the more matching clauses, the better — while the `must_not` and `filter` clauses are executed in filter context.

  用于组合多个叶子或复合查询子句的默认查询，作为 "must"、"should"、"must_not "或 "filter"子句。必须 "must "should"子句的分数被合并 - 匹配的子句越多越好 - 而 "must_not "和 "filter"子句则在过滤上下文中执行。

- **[`boosting` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-boosting-query.html)**

  Return documents which match a `positive` query, but reduce the score of documents which also match a `negative` query.

  返回符合 "positive"查询的文件，但减少符合 "negative"查询的文件的分数。

- **[`constant_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-constant-score-query.html)**

  A query which wraps another query, but executes it in filter context. All matching documents are given the same “constant” `_score`.

  一个包裹另一个查询的查询，但在过滤器上下文中执行它。所有匹配的文件都被赋予相同的 "常数"`_score`。

- **[`dis_max` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-dis-max-query.html)**

  A query which accepts multiple queries, and returns any documents which match any of the query clauses. While the `bool` query combines the scores from all matching queries, the `dis_max` query uses the score of the single best- matching query clause.

  一个接受多个查询的查询，并返回符合任何查询条款的任何文件。虽然 "bool "查询结合了所有匹配查询的分数，但 "dis_max "查询使用单个最佳匹配查询条款的分数。

- **[`function_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html)**

  Modify the scores returned by the main query with functions to take into account factors like popularity, recency, distance, or custom algorithms implemented with scripting.

  用函数修改主查询返回的分数，以考虑流行度、经常性、距离等因素，或用脚本实现的自定义算法。

### Boolean查询

A query that matches documents matching boolean combinations of other queries. The bool query maps to Lucene `BooleanQuery`. It is built using one or more boolean clauses, each clause with a typed occurrence. The occurrence types are:

匹配其他查询的布尔组合的文件的查询。bool查询映射到Lucene的`BooleanQuery`。它是用一个或多个布尔子句建立的，每个子句有一个类型的出现。出现的类型是。

| Occur      | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `must`     | The clause (query) must appear in matching documents and will contribute to the score. |
| `filter`   | The clause (query) must appear in matching documents. However unlike `must` the score of the query will be ignored. Filter clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html), meaning that scoring is ignored and clauses are considered for caching. |
| `should`   | The clause (query) should appear in the matching document.   |
| `must_not` | The clause (query) must not appear in the matching documents. Clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html) meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of `0` for all documents is returned. |

The `bool` query takes a *more-matches-is-better* approach, so the score from each matching `must` or `should` clause will be added together to provide the final `_score` for each document.

`bool`查询采取more-matches-is-better是更好的方法，因此每个匹配的 "必须 "或 "应该 "子句的分数将被加在一起，以提供每个文档的最终"_score"。

```console
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user.id" : "kimchy" }
      },
      "filter": {
        "term" : { "tags" : "production" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tags" : "env1" } },
        { "term" : { "tags" : "deployed" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

#### 使用 `minimum_should_match`

You can use the `minimum_should_match` parameter to specify the number or percentage of `should` clauses returned documents *must* match.

If the `bool` query includes at least one `should` clause and no `must` or `filter` clauses, the default value is `1`. Otherwise, the default value is `0`.

你可以使用`minimum_should_match`参数来指定返回的`should`子句的数量或百分比，文件*必须*匹配。

如果 `bool`查询至少包括一个 `should `子句，没有 `must`或 `filter`子句，默认值是 `1`。否则，默认值为 `0`。

For other valid values, see the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html).

#### 通过 `bool.filter`滚动

Queries specified under the `filter` element have no effect on scoring — scores are returned as `0`. Scores are only affected by the query that has been specified. For instance, all three of the following queries return all documents where the `status` field contains the term `active`.

This first query assigns a score of `0` to all documents, as no scoring query has been specified:

在 `filter`元素下指定的查询对评分没有影响--分数被返回为 `0`。分数只受指定的查询的影响。例如，下面的三个查询都会返回所有`status`字段包含`active'一词的文件。

这第一个查询给所有的文件分配一个`0'的分数，因为没有指定评分查询。

```console
GET _search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

This `bool` query has a `match_all` query, which assigns a score of `1.0` to all documents.

```console
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

This `constant_score` query behaves in exactly the same way as the second example above. The `constant_score` query assigns a score of `1.0` to all documents matched by the filter.

这个 `constant_score` 查询的行为方式与上面的第二个示例完全相同。 `constant_score` 查询为过滤器匹配的所有文档分配一个 `1.0` 的分数。

```console
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

#### 命名查询

Each query accepts a `_name` in its top level definition. You can use named queries to track which queries matched returned documents. If named queries are used, the response includes a `matched_queries` property for each hit.

每个查询在其顶级定义中接受一个 `_name`。 您可以使用命名查询来跟踪哪些查询与返回的文档匹配。 如果使用命名查询，则响应包含每个命中的“matched_queries”属性。

```console
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name.first": { "query": "shay", "_name": "first" } } },
        { "match": { "name.last": { "query": "banon", "_name": "last" } } }
      ],
      "filter": {
        "terms": {
          "name.last": [ "banon", "kimchy" ],
          "_name": "test"
        }
      }
    }
  }
}
```

### Boosting查询

Returns documents matching a `positive` query while reducing the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) of documents that also match a `negative` query.

You can use the `boosting` query to demote certain documents without excluding them from the search results.

返回与`positive`查询匹配的文档，同时降低文档的 [相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) 也匹配“否定”查询。

您可以使用 `boosting` 查询来降级某些文档而不将它们从搜索结果中排除。

#### 请求示例

```console
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "text": "apple"
        }
      },
      "negative": {
        "term": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

####  `boosting`的top-level参数

- **`positive`**

  (Required, query object) Query you wish to run. Any returned documents must match this query.

- **`negative`**

  (Required, query object) Query used to decrease the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) of matching documents.If a returned document matches the `positive` query and this query, the `boosting` query calculates the final [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) for the document as follows:Take the original relevance score from the `positive` query.Multiply the score by the `negative_boost` value.

  用于降低匹配[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores)的查询 文档。如果返回的文档与 `positive` 查询和此查询匹配，`boosting` 查询将计算最终的 [相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query -filter-context.html#relevance-scores) 文档如下：从 `positive` 查询中获取原始相关性分数。将分数乘以 `negative_boost` 值。

- **`negative_boost`**

  (Required, float) Floating point number between `0` and `1.0` used to decrease the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) of documents matching the `negative` query.

  用于降低[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context）之间的“0”和“1.0”之间的浮点数 .html#relevance-scores) 匹配`negative` 查询的文档。

### Constant score查询

Wraps a [filter query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) and returns every matching document with a [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) equal to the `boost` parameter value.

包装一个 [filter query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) 并返回每个匹配的文档，并带有 [relevance score](https ://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) 等于 `boost` 参数值。

```console
GET /_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": { "user.id": "kimchy" }
      },
      "boost": 1.2
    }
  }
}
```

#### `constant_score` 的顶级参数

- **`filter`**

   （必填，查询对象）[filter query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) 你想运行。 任何返回的文档都必须与此查询匹配。过滤查询不计算 [relevance-scores](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) . 为了提高性能，Elasticsearch 会自动缓存常用的过滤器查询。

- **`boost`**

   （可选，浮点数）用作常量的浮点数 [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) 每个匹配“过滤器”查询的文档。 默认为“1.0”。

### Disjunction max查询

Returns documents matching one or more wrapped queries, called query clauses or clauses.

If a returned document matches multiple query clauses, the `dis_max` query assigns the document the highest relevance score from any matching clause, plus a tie breaking increment for any additional matching subqueries.

You can use the `dis_max` to search for a term in fields mapped with different [boost](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html) factors.



返回匹配一个或多个包装查询的文档，称为查询子句或子句。 

如果返回的文档与多个查询子句匹配，`dis_max` 查询会为文档分配任何匹配子句的最高相关性分数，并为任何其他匹配子查询分配一个打破平局的增量。 您可以使用 `dis_max` 在用不同 [boost](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html) 因子映射的字段中搜索术语。 

#### 示例

```console
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "term": { "title": "Quick pets" } },
        { "term": { "body": "Quick pets" } }
      ],
      "tie_breaker": 0.7
    }
  }
}
```

#### `dis_max`参数

- **`queries`**

  (Required, array of query objects) Contains one or more query clauses. Returned documents **must match one or more** of these queries. If a document matches multiple queries, Elasticsearch uses the highest [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html).

  （必需，查询对象数组）包含一个或多个查询子句。 返回的文档**必须与这些查询中的一个或多个**匹配。 如果一个文档匹配多个查询，Elasticsearch 使用最高的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html)。

- **`tie_breaker`**

  (Optional, float) Floating point number between `0` and `1.0` used to increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#relevance-scores) of documents matching multiple query clauses. Defaults to `0.0`.You can use the `tie_breaker` value to assign higher relevance scores to documents that contain the same term in multiple fields than documents that contain this term in only the best of those multiple fields, without confusing this with the better case of two different terms in the multiple fields.If a document matches multiple clauses, the `dis_max` query calculates the relevance score for the document as follows:Take the relevance score from a matching clause with the highest score.Multiply the score from any other matching clauses by the `tie_breaker` value.Add the highest score to the multiplied scores.If the `tie_breaker` value is greater than `0.0`, all matching clauses count, but the clause with the highest score counts most.

  （可选，浮点数）`0` 和`1.0` 之间的浮点数，用于增加[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context .html#relevance-scores) 匹配多个查询子句的文档。默认为 `0.0`。您可以使用 `tie_breaker` 值为在多个字段中包含相同术语的文档分配比仅在这些多个字段中最好的字段中包含该术语的文档更高的相关性分数，而不会将此与更好的混淆多个字段中两个不同术语的情况。如果一个文档匹配多个子句，`dis_max` 查询计算文档的相关性分数如下：从具有最高分数的匹配子句中获取相关性分数。乘以任何分数tie_breaker 值的其他匹配子句。将最高分数与相乘的分数相加。如果 tie_breaker 值大于 0.0，则所有匹配的子句都计数，但分数最高的子句计数最多。

### Function score查询

The `function_score` allows you to modify the score of documents that are retrieved by a query. This can be useful if, for example, a score function is computationally expensive and it is sufficient to compute the score on a filtered set of documents.

To use `function_score`, the user has to define a query and one or more functions, that compute a new score for each document returned by the query.

`function_score` 允许您修改查询检索到的文档的分数。 例如，如果评分函数的计算成本很高并且足以计算过滤后的文档集的评分，则这可能很有用。

要使用`function_score`，用户必须定义一个查询和一个或多个函数，为查询返回的每个文档计算一个新的分数。

`function_score` can be used with only one function like this:

```console
GET /_search
{
  "query": {
    "function_score": {
      "query": { "match_all": {} },
      "boost": "5",
      "random_score": {}, 
      "boost_mode": "multiply"
    }
  }
}
```

|      | See [Function score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#score-functions) for a list of supported functions. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Furthermore, several functions can be combined. In this case one can optionally choose to apply the function only if a document matches a given filtering query

此外，还可以组合多个功能。 在这种情况下，可以选择仅在文档与给定的过滤查询匹配时才应用该函数

```console
GET /_search
{
  "query": {
    "function_score": {
      "query": { "match_all": {} },
      "boost": "5", 
      "functions": [
        {
          "filter": { "match": { "test": "bar" } },
          "random_score": {}, 
          "weight": 23
        },
        {
          "filter": { "match": { "test": "cat" } },
          "weight": 42
        }
      ],
      "max_boost": 42,
      "score_mode": "max",
      "boost_mode": "multiply",
      "min_score": 42
    }
  }
}
```

|      | Boost for the whole query.                                   |
| ---- | ------------------------------------------------------------ |
|      | See [Function score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#score-functions) for a list of supported functions. |

The scores produced by the filtering query of each function do not matter.

If no filter is given with a function this is equivalent to specifying `"match_all": {}`

First, each document is scored by the defined functions. The parameter `score_mode` specifies how the computed scores are combined:

| `multiply` | scores are multiplied (default)                          |
| ---------- | -------------------------------------------------------- |
| `sum`      | scores are summed                                        |
| `avg`      | scores are averaged                                      |
| `first`    | the first function that has a matching filter is applied |
| `max`      | maximum score is used                                    |
| `min`      | minimum score is used                                    |

Because scores can be on different scales (for example, between 0 and 1 for decay functions but arbitrary for `field_value_factor`) and also because sometimes a different impact of functions on the score is desirable, the score of each function can be adjusted with a user defined `weight`. The `weight` can be defined per function in the `functions` array (example above) and is multiplied with the score computed by the respective function. If weight is given without any other function declaration, `weight` acts as a function that simply returns the `weight`.

In case `score_mode` is set to `avg` the individual scores will be combined by a **weighted** average. For example, if two functions return score 1 and 2 and their respective weights are 3 and 4, then their scores will be combined as `(1*3+2*4)/(3+4)` and **not** `(1*3+2*4)/2`.

The new score can be restricted to not exceed a certain limit by setting the `max_boost` parameter. The default for `max_boost` is FLT_MAX.

The newly computed score is combined with the score of the query. The parameter `boost_mode` defines how:

| `multiply` | query score and function score is multiplied (default)  |
| ---------- | ------------------------------------------------------- |
| `replace`  | only function score is used, the query score is ignored |
| `sum`      | query score and function score are added                |
| `avg`      | average                                                 |
| `max`      | max of query score and function score                   |
| `min`      | min of query score and function score                   |

By default, modifying the score does not change which documents match. To exclude documents that do not meet a certain score threshold the `min_score` parameter can be set to the desired score threshold.

For `min_score` to work, **all** documents returned by the query need to be scored and then filtered out one by one.

The `function_score` query provides several types of score functions.

- [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-script-score)
- [`weight`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-weight)
- [`random_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-random)
- [`field_value_factor`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-field-value-factor)
- [decay functions](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-decay): `gauss`, `linear`, `exp`

#### Script score

The `script_score` function allows you to wrap another query and customize the scoring of it optionally with a computation derived from other numeric field values in the doc using a script expression. Here is a simple sample:

```console
GET /_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "message": "elasticsearch" }
      },
      "script_score": {
        "script": {
          "source": "Math.log(2 + doc['my-int'].value)"
        }
      }
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/855.console) 

In Elasticsearch, all document scores are positive 32-bit floating point numbers.

If the `script_score` function produces a score with greater precision, it is converted to the nearest 32-bit float.

Similarly, scores must be non-negative. Otherwise, Elasticsearch returns an error.

On top of the different scripting field values and expression, the `_score` script parameter can be used to retrieve the score based on the wrapped query.

Scripts compilation is cached for faster execution. If the script has parameters that it needs to take into account, it is preferable to reuse the same script, and provide parameters to it:

```console
GET /_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "message": "elasticsearch" }
      },
      "script_score": {
        "script": {
          "params": {
            "a": 5,
            "b": 1.2
          },
          "source": "params.a / Math.pow(params.b, doc['my-int'].value)"
        }
      }
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/856.console) 

Note that unlike the `custom_score` query, the score of the query is multiplied with the result of the script scoring. If you wish to inhibit this, set `"boost_mode": "replace"`

#### Weight

The `weight` score allows you to multiply the score by the provided `weight`. This can sometimes be desired since boost value set on specific queries gets normalized, while for this score function it does not. The number value is of type float.

```js
"weight" : number
```

#### Random

The `random_score` generates scores that are uniformly distributed from 0 up to but not including 1. By default, it uses the internal Lucene doc ids as a source of randomness, which is very efficient but unfortunately not reproducible since documents might be renumbered by merges.

In case you want scores to be reproducible, it is possible to provide a `seed` and `field`. The final score will then be computed based on this seed, the minimum value of `field` for the considered document and a salt that is computed based on the index name and shard id so that documents that have the same value but are stored in different indexes get different scores. Note that documents that are within the same shard and have the same value for `field` will however get the same score, so it is usually desirable to use a field that has unique values for all documents. A good default choice might be to use the `_seq_no` field, whose only drawback is that scores will change if the document is updated since update operations also update the value of the `_seq_no` field.

It was possible to set a seed without setting a field, but this has been deprecated as this requires loading fielddata on the `_id` field which consumes a lot of memory.

```console
GET /_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 10,
        "field": "_seq_no"
      }
    }
  }
}
```

#### Field Value factor

The `field_value_factor` function allows you to use a field from a document to influence the score. It’s similar to using the `script_score` function, however, it avoids the overhead of scripting. If used on a multi-valued field, only the first value of the field is used in calculations.

As an example, imagine you have a document indexed with a numeric `my-int` field and wish to influence the score of a document with this field, an example doing so would look like:

```console
GET /_search
{
  "query": {
    "function_score": {
      "field_value_factor": {
        "field": "my-int",
        "factor": 1.2,
        "modifier": "sqrt",
        "missing": 1
      }
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/858.console) 

Which will translate into the following formula for scoring:

```
sqrt(1.2 * doc['my-int'].value)
```

There are a number of options for the `field_value_factor` function:

| `field`    | Field to be extracted from the document.                     |
| ---------- | ------------------------------------------------------------ |
| `factor`   | Optional factor to multiply the field value with, defaults to `1`. |
| `modifier` | Modifier to apply to the field value, can be one of: `none`, `log`, `log1p`, `log2p`, `ln`, `ln1p`, `ln2p`, `square`, `sqrt`, or `reciprocal`. Defaults to `none`. |

| Modifier     | Meaning                                                      |
| ------------ | ------------------------------------------------------------ |
| `none`       | Do not apply any multiplier to the field value               |
| `log`        | Take the [common logarithm](https://en.wikipedia.org/wiki/Common_logarithm) of the field value. Because this function will return a negative value and cause an error if used on values between 0 and 1, it is recommended to use `log1p` instead. |
| `log1p`      | Add 1 to the field value and take the common logarithm       |
| `log2p`      | Add 2 to the field value and take the common logarithm       |
| `ln`         | Take the [natural logarithm](https://en.wikipedia.org/wiki/Natural_logarithm) of the field value. Because this function will return a negative value and cause an error if used on values between 0 and 1, it is recommended to use `ln1p` instead. |
| `ln1p`       | Add 1 to the field value and take the natural logarithm      |
| `ln2p`       | Add 2 to the field value and take the natural logarithm      |
| `square`     | Square the field value (multiply it by itself)               |
| `sqrt`       | Take the [square root](https://en.wikipedia.org/wiki/Square_root) of the field value |
| `reciprocal` | [Reciprocate](https://en.wikipedia.org/wiki/Multiplicative_inverse) the field value, same as `1/x` where `x` is the field’s value |

- **`missing`**

  Value used if the document doesn’t have that field. The modifier and factor are still applied to it as though it were read from the document.

Scores produced by the `field_value_score` function must be non-negative, otherwise an error will be thrown. The `log` and `ln` modifiers will produce negative values if used on values between 0 and 1. Be sure to limit the values of the field with a range filter to avoid this, or use `log1p` and `ln1p`.

Keep in mind that taking the log() of 0, or the square root of a negative number is an illegal operation, and an exception will be thrown. Be sure to limit the values of the field with a range filter to avoid this, or use `log1p` and `ln1p`.

#### Decay functions

Decay functions score a document with a function that decays depending on the distance of a numeric field value of the document from a user given origin. This is similar to a range query, but with smooth edges instead of boxes.

To use distance scoring on a query that has numerical fields, the user has to define an `origin` and a `scale` for each field. The `origin` is needed to define the “central point” from which the distance is calculated, and the `scale` to define the rate of decay. The decay function is specified as

```js
"DECAY_FUNCTION": { 
    "FIELD_NAME": { 
          "origin": "11, 12",
          "scale": "2km",
          "offset": "0km",
          "decay": 0.33
    }
}
```

|      | The `DECAY_FUNCTION` should be one of `linear`, `exp`, or `gauss`. |
| ---- | ------------------------------------------------------------ |
|      | The specified field must be a numeric, date, or geopoint field. |

In the above example, the field is a [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html) and origin can be provided in geo format. `scale` and `offset` must be given with a unit in this case. If your field is a date field, you can set `scale` and `offset` as days, weeks, and so on. Example:

```console
GET /_search
{
  "query": {
    "function_score": {
      "gauss": {
        "@timestamp": {
          "origin": "2013-09-17", 
          "scale": "10d",
          "offset": "5d",         
          "decay": 0.5            
        }
      }
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/859.console) 

|      | The date format of the origin depends on the [`format`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html) defined in your mapping. If you do not define the origin, the current time is used. |
| ---- | ------------------------------------------------------------ |
|      | The `offset` and `decay` parameters are optional.            |

| `origin` | The point of origin used for calculating distance. Must be given as a number for numeric field, date for date fields and geo point for geo fields. Required for geo and numeric field. For date fields the default is `now`. Date math (for example `now-1h`) is supported for origin. |
| -------- | ------------------------------------------------------------ |
| `scale`  | Required for all types. Defines the distance from origin + offset at which the computed score will equal `decay` parameter. For geo fields: Can be defined as number+unit (1km, 12m,…). Default unit is meters. For date fields: Can to be defined as a number+unit ("1h", "10d",…). Default unit is milliseconds. For numeric field: Any number. |
| `offset` | If an `offset` is defined, the decay function will only compute the decay function for documents with a distance greater than the defined `offset`. The default is 0. |
| `decay`  | The `decay` parameter defines how documents are scored at the distance given at `scale`. If no `decay` is defined, documents at the distance `scale` will be scored 0.5. |

In the first example, your documents might represents hotels and contain a geo location field. You want to compute a decay function depending on how far the hotel is from a given location. You might not immediately see what scale to choose for the gauss function, but you can say something like: "At a distance of 2km from the desired location, the score should be reduced to one third." The parameter "scale" will then be adjusted automatically to assure that the score function computes a score of 0.33 for hotels that are 2km away from the desired location.

In the second example, documents with a field value between 2013-09-12 and 2013-09-22 would get a weight of 1.0 and documents which are 15 days from that date a weight of 0.5.

##### Supported decay functions

The `DECAY_FUNCTION` determines the shape of the decay:

- **`gauss`**

  Normal decay, computed as:![Gaussian](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/Gaussian.png)where ![sigma](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/sigma.png) is computed to assure that the score takes the value `decay` at distance `scale` from `origin`+-`offset`![sigma calc](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/sigma_calc.png)See [Normal decay, keyword `gauss`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#gauss-decay) for graphs demonstrating the curve generated by the `gauss` function.

- **`exp`**

  Exponential decay, computed as:![Exponential](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/Exponential.png)where again the parameter ![lambda](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/lambda.png) is computed to assure that the score takes the value `decay` at distance `scale` from `origin`+-`offset`![lambda calc](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/lambda_calc.png)See [Exponential decay, keyword `exp`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#exp-decay) for graphs demonstrating the curve generated by the `exp` function.

- **`linear`**

  Linear decay, computed as:![Linear](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/Linear.png).where again the parameter `s` is computed to assure that the score takes the value `decay` at distance `scale` from `origin`+-`offset`![s calc](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/s_calc.png)In contrast to the normal and exponential decay, this function actually sets the score to 0 if the field value exceeds twice the user given scale value.

For single functions the three decay functions together with their parameters can be visualized like this (the field in this example called "age"):

![decay 2d](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/decay_2d.png)

##### Multi-values fields

If a field used for computing the decay contains multiple values, per default the value closest to the origin is chosen for determining the distance. This can be changed by setting `multi_value_mode`.

| `min` | Distance is the minimum distance     |
| ----- | ------------------------------------ |
| `max` | Distance is the maximum distance     |
| `avg` | Distance is the average distance     |
| `sum` | Distance is the sum of all distances |

Example:

```js
    "DECAY_FUNCTION": {
        "FIELD_NAME": {
              "origin": ...,
              "scale": ...
        },
        "multi_value_mode": "avg"
    }
```

#### Detailed example

Suppose you are searching for a hotel in a certain town. Your budget is limited. Also, you would like the hotel to be close to the town center, so the farther the hotel is from the desired location the less likely you are to check in.

You would like the query results that match your criterion (for example, "hotel, Nancy, non-smoker") to be scored with respect to distance to the town center and also the price.

Intuitively, you would like to define the town center as the origin and maybe you are willing to walk 2km to the town center from the hotel.
In this case your **origin** for the location field is the town center and the **scale** is ~2km.

If your budget is low, you would probably prefer something cheap above something expensive. For the price field, the **origin** would be 0 Euros and the **scale** depends on how much you are willing to pay, for example 20 Euros.

In this example, the fields might be called "price" for the price of the hotel and "location" for the coordinates of this hotel.

The function for `price` in this case would be

```js
"gauss": { 
    "price": {
          "origin": "0",
          "scale": "20"
    }
}
```

|      | This decay function could also be `linear` or `exp`. |
| ---- | ---------------------------------------------------- |
|      |                                                      |

and for `location`:

```js
"gauss": { 
    "location": {
          "origin": "11, 12",
          "scale": "2km"
    }
}
```

|      | This decay function could also be `linear` or `exp`. |
| ---- | ---------------------------------------------------- |
|      |                                                      |

Suppose you want to multiply these two functions on the original score, the request would look like this:

```console
GET /_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "gauss": {
            "price": {
              "origin": "0",
              "scale": "20"
            }
          }
        },
        {
          "gauss": {
            "location": {
              "origin": "11, 12",
              "scale": "2km"
            }
          }
        }
      ],
      "query": {
        "match": {
          "properties": "balcony"
        }
      },
      "score_mode": "multiply"
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/860.console) 

Next, we show how the computed score looks like for each of the three possible decay functions.

##### Normal decay, keyword `gauss`

When choosing `gauss` as the decay function in the above example, the contour and surface plot of the multiplier looks like this:

![cd0e18a6 e898 11e2 9b3c f0145078bd6f](https://f.cloud.github.com/assets/4320215/768157/cd0e18a6-e898-11e2-9b3c-f0145078bd6f.png)

![ec43c928 e898 11e2 8e0d f3c4519dbd89](https://f.cloud.github.com/assets/4320215/768160/ec43c928-e898-11e2-8e0d-f3c4519dbd89.png)

Suppose your original search results matches three hotels :

- "Backback Nap"
- "Drink n Drive"
- "BnB Bellevue".

"Drink n Drive" is pretty far from your defined location (nearly 2 km) and is not too cheap (about 13 Euros) so it gets a low factor a factor of 0.56. "BnB Bellevue" and "Backback Nap" are both pretty close to the defined location but "BnB Bellevue" is cheaper, so it gets a multiplier of 0.86 whereas "Backpack Nap" gets a value of 0.66.

##### Exponential decay, keyword `exp`

When choosing `exp` as the decay function in the above example, the contour and surface plot of the multiplier looks like this:

![082975c0 e899 11e2 86f7 174c3a729d64](https://f.cloud.github.com/assets/4320215/768161/082975c0-e899-11e2-86f7-174c3a729d64.png)

![0b606884 e899 11e2 907b aefc77eefef6](https://f.cloud.github.com/assets/4320215/768162/0b606884-e899-11e2-907b-aefc77eefef6.png)

##### Linear decay, keyword `linear`

When choosing `linear` as the decay function in the above example, the contour and surface plot of the multiplier looks like this:

![1775b0ca e899 11e2 9f4a 776b406305c6](https://f.cloud.github.com/assets/4320215/768164/1775b0ca-e899-11e2-9f4a-776b406305c6.png)

![19d8b1aa e899 11e2 91bc 6b0553e8d722](https://f.cloud.github.com/assets/4320215/768165/19d8b1aa-e899-11e2-91bc-6b0553e8d722.png)

#### Supported fields for decay functions

Only numeric, date, and geopoint fields are supported.

#### What if a field is missing?

If the numeric field is missing in the document, the function will return 1.

## 全文检索

全文查询使您能够搜索[analyzed text fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)，例如电子邮件正文。 使用在索引期间应用于字段的相同分析器处理查询字符串。

他在这个组中的查询是：

- **[`intervals`查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html)**

  A full text query that allows fine-grained control of the ordering and proximity of matching terms.

  允许对匹配项的排序和接近度进行细粒度控制的全文查询。

- **[`match` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)**

  The standard query for performing full text queries, including fuzzy matching and phrase or proximity queries.

  用于执行全文查询的标准查询，包括模糊匹配和短语或邻近查询。

- **[`match_bool_prefix` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-bool-prefix-query.html)**

  Creates a `bool` query that matches each term as a `term` query, except for the last term, which is matched as a `prefix` query

  创建一个 `bool` 查询，将每个词匹配为一个 `term` 查询，最后一个词除外，它作为一个 `prefix` 查询匹配

- **[`match_phrase` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html)**

  Like the `match` query but used for matching exact phrases or word proximity matches.

  类似于 `match` 查询，但用于匹配精确的短语或单词接近匹配。

- **[`match_phrase_prefix` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase-prefix.html)**

  Like the `match_phrase` query, but does a wildcard search on the final word.

  类似于 `match_phrase` 查询，但对最后一个单词进行通配符搜索。

- **[`multi_match` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)**

  The multi-field version of the `match` query.

  `match` 查询的多字段版本。

- **[`combined_fields` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-combined-fields-query.html)**

  Matches over multiple fields as if they had been indexed into one combined field.

  匹配多个字段，就好像它们已被索引到一个组合字段中一样。

- **[`query_string` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)**

  Supports the compact Lucene [query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax), allowing you to specify AND|OR|NOT conditions and multi-field search within a single query string. For expert users only.

  支持紧凑的 Lucene [查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax)，允许您可以在单个查询字符串中指定 AND|OR|NOT 条件和多字段搜索。仅供专家用户使用。

- **[`simple_query_string` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-simple-query-string-query.html)**

  A simpler, more robust version of the `query_string` syntax suitable for exposing directly to users.
  
  一个更简单、更健壮的 `query_string` 语法版本，适合直接向用户公开。

### Intervals查询

Returns documents based on the order and proximity of matching terms.

The `intervals` query uses **matching rules**, constructed from a small set of definitions. These rules are then applied to terms from a specified `field`.

The definitions produce sequences of minimal intervals that span terms in a body of text. These intervals can be further combined and filtered by parent sources.

根据匹配项的顺序和接近程度返回文档。

`intervals` 查询使用**匹配规则**，由一小组定义构成。 然后将这些规则应用于来自指定“字段”的术语。

定义产生跨越文本正文中的术语的最小间隔序列。 这些间隔可以通过父源进一步组合和过滤。

#### 请求示例

The following `intervals` search returns documents containing `my favorite food` without any gap, followed by `hot water` or `cold porridge` in the `my_text` field.

This search would match a `my_text` value of `my favorite food is cold porridge` but not `when it's cold my favorite food is porridge`.



下面的 `intervals` 搜索返回包含 `my favorite food` 的文档，没有任何间隙，然后是 `my_text` 字段中的 `hot water` 或 `cold porridge`。

此搜索将匹配`my favorite food is cold porridge`的“my_text”值，但不会匹配`when it's cold my favorite food is porridge`的值。

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favorite food",
                "max_gaps" : 0,
                "ordered" : true
              }
            },
            {
              "any_of" : {
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

#### `intervals`的参数



- **`<field>`**

  (Required, rule object) Field you wish to search.The value of this parameter is a rule object used to match documents based on matching terms, order, and proximity.

  （必填，规则对象）您希望搜索的字段。此参数的值是一个规则对象，用于根据匹配项、顺序和接近度来匹配文档。

  Valid rules include:

  - [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-match)
  - [`prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-prefix)
  - [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-wildcard)
  - [`fuzzy`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-fuzzy)
  - [`all_of`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-all_of)
  - [`any_of`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-any_of)



##### `match` 规则的参数

The `match` rule matches analyzed text.

- **`query`**

  (Required, string) Text you wish to find in the provided `<field>`.

  （必需，字符串）您希望在提供的`<field>`.

- **`max_gaps`**

  (Optional, integer) Maximum number of positions between the matching terms. Terms further apart than this are not considered matches. Defaults to `-1`.If unspecified or set to `-1`, there is no width restriction on the match. If set to `0`, the terms must appear next to each other.

  （可选，整数）匹配项之间的最大位置数。比这更远的术语不被视为匹配。默认为 `-1`.

  如果未指定或设置为`-1`，则匹配没有宽度限制。如果设置为`0`，则术语必须彼此相邻显示。

- **`ordered`**

  (Optional, Boolean) If `true`, matching terms must appear in their specified order. Defaults to `false`.

- **`analyzer`**

  (Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) used to analyze terms in the `query`. Defaults to the top-level `<field>`'s analyzer.

  （可选，字符串）[分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)用于分析`query`. 默认为顶级`<field>`的分析器。

- **`filter`**

  (Optional, [interval filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#interval_filter) rule object) An optional interval filter.

  （可选，[间隔过滤](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#interval_filter)规则对象）一个可选的间隔过滤器。

- **`use_field`**

  (Optional, string) If specified, then match intervals from this field rather than the top-level `<field>`. Terms are analyzed using the search analyzer from this field. This allows you to search across multiple fields as if they were all the same field; for example, you could index the same text into stemmed and unstemmed fields, and search for stemmed tokens near unstemmed ones.

  （可选，字符串）如果指定，则匹配来自此字段而不是顶级 的间隔`<field>`。使用该字段中的搜索分析器分析术语。这允许您跨多个字段进行搜索，就好像它们都是相同的字段一样；例如，您可以将相同的文本索引到词干和非词干字段中，并在非词干标记附近搜索词干标记。

##### `prefix` 

The `prefix` rule matches terms that start with a specified set of characters. This prefix can expand to match at most 128 terms. If the prefix matches more than 128 terms, Elasticsearch returns an error. You can use the [`index-prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-prefixes.html) option in the field mapping to avoid this limit.

该`prefix`规则匹配以指定字符集开头的术语。此前缀可以扩展以匹配最多 128 个术语。如果前缀匹配超过 128 个术语，Elasticsearch 将返回错误。您可以使用[`index-prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-prefixes.html)字段映射中的 选项来避免此限制。

- **`prefix`**

  (Required, string) Beginning characters of terms you wish to find in the top-level `<field>`.

- **`analyzer`**

  (Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) used to normalize the `prefix`. Defaults to the top-level `<field>`'s analyzer.

- **`use_field`**

  (Optional, string) If specified, then match intervals from this field rather than the top-level `<field>`.The `prefix` is normalized using the search analyzer from this field, unless a separate `analyzer` is specified.

  （可选，字符串）如果指定，则匹配来自此字段而不是顶级 的间隔`<field>`。

  `prefix`使用此字段中的搜索分析器对进行标准化，除非`analyzer`指定了单独的。

##### `wildcard` 

The `wildcard` rule matches terms using a wildcard pattern. This pattern can expand to match at most 128 terms. If the pattern matches more than 128 terms, Elasticsearch returns an error.

该`wildcard`规则使用通配符模式匹配术语。此模式可以扩展以匹配最多 128 个术语。如果模式匹配超过 128 个术语，Elasticsearch 将返回错误。

- **`pattern`**

  (Required, string) Wildcard pattern used to find matching terms.

  This parameter supports two wildcard operators:

  ​	`?`, which matches any single character

  ​	`*`, which can match zero or more characters, including an empty one

  此参数支持两个通配符运算符：

  - `?`, 匹配任何单个字符
  - `*`, 可以匹配零个或多个字符，包括空字符

  >  Avoid beginning patterns with `*` or `?`. This can increase the iterations needed to find matching terms and slow search performance.
  >
  > 避免以`*`或开始模式`?`。这会增加查找匹配项所需的迭代次数并降低搜索性能。

- **`analyzer`**

  (Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) used to normalize the `pattern`. Defaults to the top-level `<field>`'s analyzer.

- **`use_field`**

  (Optional, string) If specified, match intervals from this field rather than the top-level `<field>`.

  The `pattern` is normalized using the search analyzer from this field, unless `analyzer` is specified separately.

##### `fuzzy`

The `fuzzy` rule matches terms that are similar to the provided term, within an edit distance defined by [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness). If the fuzzy expansion matches more than 128 terms, Elasticsearch returns an error.

`fuzzy` 规则匹配与提供的术语相似的术语，在 [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html) 定义的编辑距离内 #模糊性）。 如果模糊扩展匹配超过 128 个术语，Elasticsearch 将返回错误。

- **`term`**

  (Required, string) The term to match

- **`prefix_length`**

  (Optional, integer) Number of beginning characters left unchanged when creating expansions. Defaults to `0`.

  创建扩展时保持不变的起始字符数。默认为0

- **`transpositions`**

  (Optional, Boolean) Indicates whether edits include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

  （可选，布尔值）指示编辑是否包括两个相邻字符的换位（ab → ba）。默认为`true`.

- **`fuzziness`**

  (Optional, string) Maximum edit distance allowed for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness) for valid values and more information. Defaults to `auto`.

  （可选，字符串）允许匹配的最大编辑距离。有关 有效值和更多信息，请参阅[模糊度](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness)。默认为`auto`.

- **`analyzer`**

  (Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) used to normalize the `term`. Defaults to the top-level `<field>` 's analyzer.

- **`use_field`**

  (Optional, string) If specified, match intervals from this field rather than the top-level `<field>`.The `term` is normalized using the search analyzer from this field, unless `analyzer` is specified separately.

##### `all_of`

The `all_of` rule returns matches that span a combination of other rules.

- **`intervals`**

  (Required, array of rule objects) An array of rules to combine. All rules must produce a match in a document for the overall source to match.

  （必需，规则对象数组）要组合的规则数组。所有规则都必须在文档中生成匹配项，以便整体源进行匹配。

- **`max_gaps`**

  (Optional, integer) Maximum number of positions between the matching terms. Intervals produced by the rules further apart than this are not considered matches. Defaults to `-1`.If unspecified or set to `-1`, there is no width restriction on the match. If set to `0`, the terms must appear next to each other.

  （可选，整数）匹配项之间的最大位置数。规则产生的间隔比这更远不被视为匹配。默认为`-1`.

  如果未指定或设置为`-1`，则匹配没有宽度限制。如果设置为`0`，则术语必须彼此相邻显示。

- **`ordered`**

  (Optional, Boolean) If `true`, intervals produced by the rules should appear in the order in which they are specified. Defaults to `false`.

- **`filter`**

  (Optional, [interval filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#interval_filter) rule object) Rule used to filter returned intervals.

##### `any_of`

The `any_of` rule returns intervals produced by any of its sub-rules.

- **`intervals`**

  (Required, array of rule objects) An array of rules to match.

- **`filter`**

  (Optional, [interval filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#interval_filter) rule object) Rule used to filter returned intervals.

#### `filter`

The `filter` rule returns intervals based on a query. See [Filter example](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#interval-filter-rule-ex) for an example.

- **`after`**

  (Optional, query object) Query used to return intervals that follow an interval from the `filter` rule.

  （可选，查询对象）查询用于返回遵循`filter`规则间隔的间隔。

- **`before`**

  (Optional, query object) Query used to return intervals that occur before an interval from the `filter` rule.

  （可选，查询对象）用于返回`filter`规则中间隔之前发生的间隔的查询。

- **`contained_by`**

  (Optional, query object) Query used to return intervals contained by an interval from the `filter` rule.

- **`containing`**

  (Optional, query object) Query used to return intervals that contain an interval from the `filter` rule.

- **`not_contained_by`**

  (Optional, query object) Query used to return intervals that are **not** contained by an interval from the `filter` rule.

- **`not_containing`**

  (Optional, query object) Query used to return intervals that do **not** contain an interval from the `filter` rule.

- **`not_overlapping`**

  (Optional, query object) Query used to return intervals that do **not** overlap with an interval from the `filter` rule.

- **`overlapping`**

  (Optional, query object) Query used to return intervals that overlap with an interval from the `filter` rule.

- **`script`**

  (Optional, [script object](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html)) Script used to return matching documents. This script must return a boolean value, `true` or `false`. See [Script filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#interval-script-filter) for an example.

#### Notes

##### Filter示例

The following search includes a `filter` rule. It returns documents that have the words `hot` and `porridge` within 10 positions of each other, without the word `salty` in between:

间隔搜索，返回包含hot porridge的doc,(间隔小于10)，并且中间不包含salty

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "max_gaps" : 10,
          "filter" : {
            "not_containing" : {
              "match" : {
                "query" : "salty"
              }
            }
          }
        }
      }
    }
  }
}
```

##### Script filters

You can use a script to filter intervals based on their start position, end position, and internal gap count. The following `filter` script uses the `interval` variable with the `start`, `end`, and `gaps` methods:

下面的`filter`脚本使用 `interval`与变量`start`，`end`和`gaps`方法：

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "filter" : {
            "script" : {
              "source" : "interval.start > 10 && interval.end < 20 && interval.gaps == 0"
            }
          }
        }
      }
    }
  }
}
```

##### 最小化

The intervals query always minimizes intervals, to ensure that queries can run in linear time. This can sometimes cause surprising results, particularly when using `max_gaps` restrictions or filters. For example, take the following query, searching for `salty` contained within the phrase `hot porridge`:

intervals查询总是最小化间隔，来保证查询可以再线性时间内完成。

这有时会导致令人惊讶的结果，尤其是在使用`max_gaps`限制或过滤器时。例如，采用以下查询，搜索`salty`包含在短语中的内容`hot porridge`：

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "salty",
          "filter" : {
            "contained_by" : {
              "match" : {
                "query" : "hot porridge"
              }
            }
          }
        }
      }
    }
  }
}
```

This query does **not** match a document containing the phrase `hot porridge is salty porridge`, because the intervals returned by the match query for `hot porridge` only cover the initial two terms in this document, and these do not overlap the intervals covering `salty`.

Another restriction to be aware of is the case of `any_of` rules that contain sub-rules which overlap. In particular, if one of the rules is a strict prefix of the other, then the longer rule can never match, which can cause surprises when used in combination with `max_gaps`. Consider the following query, searching for `the` immediately followed by `big` or `big bad`, immediately followed by `wolf`:

该查询并**没有**匹配包含短语的文档`hot porridge is salty porridge`，因为间隔由匹配查询返回的`hot porridge`只涉及本文档中的最初两届，而这些不重叠覆盖的区间`salty`。

要注意的另一个限制是`any_of`包含重叠子规则的规则的情况。特别是，如果其中一个规则是另一个的严格前缀，那么更长的规则永远无法匹配，这在与`max_gaps`. 考虑下面的查询，搜索`the`紧跟`big`或`big bad`紧随其后，紧随其后`wolf`：

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "intervals" : [
            { "match" : { "query" : "the" } },
            { "any_of" : {
                "intervals" : [
                    { "match" : { "query" : "big" } },
                    { "match" : { "query" : "big bad" } }
                ] } },
            { "match" : { "query" : "wolf" } }
          ],
          "max_gaps" : 0,
          "ordered" : true
        }
      }
    }
  }
}
```

Counter-intuitively, this query does **not** match the document `the big bad wolf`, because the `any_of` rule in the middle only produces intervals for `big` - intervals for `big bad` being longer than those for `big`, while starting at the same position, and so being minimized away. In these cases, it’s better to rewrite the query so that all of the options are explicitly laid out at the top level:

与直觉相反，这个查询与文档**不**匹配`the big bad wolf`，因为`any_of`中间的规则只产生间隔 for `big`-`big bad`比 for 更长的间隔`big`，同时从相同的位置开始，因此被最小化。在这些情况下，最好重写查询，以便所有选项都在顶层显式布局：

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "any_of" : {
          "intervals" : [
            { "match" : {
                "query" : "the big bad wolf",
                "ordered" : true,
                "max_gaps" : 0 } },
            { "match" : {
                "query" : "the big wolf",
                "ordered" : true,
                "max_gaps" : 0 } }
           ]
        }
      }
    }
  }
}
```

### 匹配查询

Returns documents that match a provided text, number, date or boolean value. The provided text is analyzed before matching.

The `match` query is the standard query for performing a full-text search, including options for fuzzy matching.

返回与提供的文本、数字、日期或布尔值匹配的文档。在匹配之前分析提供的文本。

该`match`查询是执行全文搜索的标准查询，包括用于模糊匹配的选项。

#### 示例

```console
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test"
      }
    }
  }
}
```

####  `match`的参数

- **`<field>`**

  (Required, object) Field you wish to search.

#####  `<field>`的参数

- **`query`**

  (Required) Text, number, boolean value or date you wish to find in the provided `<field>`.The `match` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) any provided text before performing a search. This means the `match` query can search [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) fields for analyzed tokens rather than an exact term.

  在执行搜索之前，`match`查询会[分析](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)任何提供的文本。这意味着`match`查询可以搜索[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)分析标记的字段，而不是精确的术语。

- **`analyzer`**

  (Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) used to convert the text in the `query` value into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-time-analyzer) mapped for the `<field>`. If no analyzer is mapped, the index’s default analyzer is used.

  （可选，字符串）[分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)用于将`query` 值中的文本转换为标记。默认为`<field>`映射的[index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-time-analyzer)。如果没有映射分析器，则使用索引的默认分析器。

- **`auto_generate_synonyms_phrase_query`**

  (Optional, Boolean) If `true`, [match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html) queries are automatically created for multi-term synonyms. Defaults to `true`.

  See [Use synonyms with match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-synonyms) for an example.

  可选，布尔值）如果`true`， 则为多词同义词自动创建[匹配短语](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html)查询。默认为`true`.

- **`fuzziness`**

  (Optional, string) Maximum edit distance allowed for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness) for valid values and more information. See [Fuzziness in the match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-fuzziness) for an example.

  可选，字符串）允许匹配的最大编辑距离。有关有效值和更多信息，请参阅[模糊度](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness)。有关 示例，请参阅[匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-fuzziness)中的[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-fuzziness)。

- **`max_expansions`**

  (Optional, integer) Maximum number of terms to which the query will expand. Defaults to `50`.

  可选，整数）查询将扩展到的最大术语数。默认为`50`.

- **`prefix_length`**

  (Optional, integer) Number of beginning characters left unchanged for fuzzy matching. Defaults to `0`.

  （可选，整数）用于模糊匹配的起始字符数保持不变。默认为`0`.

- **`fuzzy_transpositions`**

  (Optional, Boolean) If `true`, edits for fuzzy matching include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

  （可选，布尔值）如果`true`，模糊匹配的编辑包括两个相邻字符的换位（ab → ba）。默认为`true`.

- **`fuzzy_rewrite`**

  (Optional, string) Method used to rewrite the query. See the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html) for valid values and more information.If the `fuzziness` parameter is not `0`, the `match` query uses a `fuzzy_rewrite` method of `top_terms_blended_freqs_${max_expansions}` by default.

  （可选，字符串）用于重写查询的方法。有关有效值和更多信息，请参阅 [`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)。

  如果`fuzziness`参数不是`0`，则`match`查询默认使用 的`fuzzy_rewrite` 方法`top_terms_blended_freqs_${max_expansions}`。

- **`lenient`**

  (Optional, Boolean) If `true`, format-based errors, such as providing a text `query` value for a [numeric](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html) field, are ignored. Defaults to `false`.

  （可选，布尔值）如果，则忽略`true`基于格式的错误，例如为[数字](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)字段提供文本 `query`值。默认为. `false`

- **`operator`**

  (Optional, string) Boolean logic used to interpret text in the `query` value. Valid values are:

  （可选，字符串）用于解释`query`值中文本的布尔逻辑。有效值为：

  **`OR` (Default)**

  For example, a `query` value of `capital of Hungary` is interpreted as `capital OR of OR Hungary`.

  **`AND`**

  For example, a `query` value of `capital of Hungary` is interpreted as `capital AND of AND Hungary`.

- **`minimum_should_match`**

  (Optional, string) Minimum number of clauses that must match for a document to be returned. See the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html) for valid values and more information.

  （可选，字符串）必须与要返回的文档匹配的最小子句数。有关有效值和更多信息，请参阅[`minimum_should_match` 参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html)。

- **`zero_terms_query`**

  (Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:

  （可选，字符串）指示如果`analyzer` 删除所有标记（例如使用`stop`过滤器时），是否不返回任何文档。有效值为：

  **`none` (Default)**

  No documents are returned if the `analyzer` removes all tokens.

  **`all`**

  Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html) query.See [Zero terms query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-zero) for an example.

#### 笔记

##### 简单请求

You can simplify the match query syntax by combining the `<field>` and `query` parameters. For example:

您可以通过组合 `<field>` 和 `query` 参数来简化匹配查询语法。 例如：

```console
GET /_search
{
  "query": {
    "match": {
      "message": "this is a test"
    }
  }
}
```

##### match查询如何工作

The `match` query is of type `boolean`. It means that the text provided is analyzed and the analysis process constructs a boolean query from the provided text. The `operator` parameter can be set to `or` or `and` to control the boolean clauses (defaults to `or`). The minimum number of optional `should` clauses to match can be set using the [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html) parameter.

Here is an example with the `operator` parameter:

`match` 查询的类型是 `boolean`。 这意味着对提供的文本进行分析，分析过程根据提供的文本构造一个布尔查询。 `operator` 参数可以设置为 `or` 或 `and` 来控制布尔子句（默认为 `or`）。 可以使用 [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should- match.html) 参数。 

这是一个带有 `operator` 参数的示例：

```console
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test",
        "operator": "and"
      }
    }
  }
}
```

The `analyzer` can be set to control which analyzer will perform the analysis process on the text. It defaults to the field explicit mapping definition, or the default search analyzer.

The `lenient` parameter can be set to `true` to ignore exceptions caused by data-type mismatches, such as trying to query a numeric field with a text query string. Defaults to `false`.

`analyzer` 可以设置为控制哪个分析器将对文本执行分析过程。 它默认为字段显式映射定义，或默认搜索分析器。

`lenient` 参数可以设置为 `true` 以忽略由数据类型不匹配引起的异常，例如尝试使用文本查询字符串查询数字字段。 默认为“假”。

##### 查询中的模糊检索

`fuzziness` allows *fuzzy matching* based on the type of field being queried. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness) for allowed settings.

`fuzziness` 允许基于被查询字段的类型*模糊匹配*。有关允许的设置，请参阅 [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness)。

The `prefix_length` and `max_expansions` can be set in this case to control the fuzzy process. If the fuzzy option is set the query will use `top_terms_blended_freqs_${max_expansions}` as its [rewrite method](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html) the `fuzzy_rewrite` parameter allows to control how the query will get rewritten.

在这种情况下可以设置`prefix_length`和`max_expansions`来控制模糊过程。如果设置了模糊选项，则查询将使用 `top_terms_blended_freqs_${max_expansions}` 作为其 [重写方法](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi- term-rewrite.html) `fuzzy_rewrite` 参数允许控制查询将如何被重写。

Fuzzy transpositions (`ab` → `ba`) are allowed by default but can be disabled by setting `fuzzy_transpositions` to `false`.

Fuzzy matching is not applied to terms with synonyms or in cases where the analysis process produces multiple tokens at the same position. Under the hood these terms are expanded to a special synonym query that blends term frequencies, which does not support fuzzy expansion.

默认情况下允许模糊转换（`ab` → `ba`），但可以通过将 `fuzzy_transpositions` 设置为 `false` 来禁用。

模糊匹配不适用于具有同义词的术语或分析过程在同一位置产生多个标记的情况。在幕后，这些术语被扩展为一个特殊的同义词查询，它混合了术语频率，不支持模糊扩展。

```console
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a testt",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

##### Zero terms query

If the analyzer used removes all tokens in a query like a `stop` filter does, the default behavior is to match no documents at all. In order to change that the `zero_terms_query` option can be used, which accepts `none` (default) and `all` which corresponds to a `match_all` query.

如果使用的分析器像 `stop` 过滤器那样删除查询中的所有标记，则默认行为是根本不匹配任何文档。 为了改变可以使用 `zero_terms_query` 选项，它接受 `none`（默认）和对应于 `match_all` 查询的 `all`。

```console
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "to be or not to be",
        "operator": "and",
        "zero_terms_query": "all"
      }
    }
  }
}
```

##### Cutoff frequency

###### Deprecated in 7.3.0.

This option can be omitted as the [Match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) can skip blocks of documents efficiently, without any configuration, provided that the total number of hits is not tracked.

The match query supports a `cutoff_frequency` that allows specifying an absolute or relative document frequency where high frequency terms are moved into an optional subquery and are only scored if one of the low frequency (below the cutoff) terms in the case of an `or` operator or all of the low frequency terms in the case of an `and` operator match.

This query allows handling `stopwords` dynamically at runtime, is domain independent and doesn’t require a stopword file. It prevents scoring / iterating high frequency terms and only takes the terms into account if a more significant / lower frequency term matches a document. Yet, if all of the query terms are above the given `cutoff_frequency` the query is automatically transformed into a pure conjunction (`and`) query to ensure fast execution.

The `cutoff_frequency` can either be relative to the total number of documents if in the range from 0 (inclusive) to 1 (exclusive) or absolute if greater or equal to `1.0`.

此选项可以省略，因为 [Match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) 可以有效地跳过文档块，而无需任何配置，前提是不跟踪总命中数。

匹配查询支持“cutoff_frequency”，它允许指定绝对或相对的文档频率，其中高频词被移动到可选的子查询中，并且只有在“或` 运算符或在 `and` 运算符匹配的情况下的所有低频术语。

此查询允许在运行时动态处理“停用词”，与域无关且不需要停用词文件。它可以防止对高频词进行评分/迭代，并且仅在更重要/较低频率的词与文档匹配时才考虑这些词。然而，如果所有查询词都高于给定的“cutoff_frequency”，则查询会自动转换为纯连接（“and”）查询以确保快速执行。

如果在 0（包含）到 1（不包含）的范围内，则 `cutoff_frequency` 可以相对于文档总数，或者如果大于或等于 `1.0`，则可以是绝对的。

Here is an example showing a query composed of stopwords exclusively:

```console
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "to be or not to be",
        "cutoff_frequency": 0.001
      }
    }
  }
}
```

The `cutoff_frequency` option operates on a per-shard-level. This means that when trying it out on test indexes with low document numbers you should follow the advice in [Relevance is broken](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/relevance-is-broken.html).

##### 同义词

The `match` query supports multi-terms synonym expansion with the [synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-graph-tokenfilter.html) token filter. When this filter is used, the parser creates a phrase query for each multi-terms synonyms. For example, the following synonym: `"ny, new york"` would produce:

`match` 查询支持使用 [synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-graph-tokenfilter.html) 标记过滤器进行多术语同义词扩展。 使用此过滤器时，解析器会为每个多术语同义词创建短语查询。 例如，以下同义词：`"ny, new york"` 将产生：

```
(ny OR ("new york"))
```

It is also possible to match multi terms synonyms with conjunctions instead:

```console
GET /_search
{
   "query": {
       "match" : {
           "message": {
               "query" : "ny city",
               "auto_generate_synonyms_phrase_query" : false
           }
       }
   }
}
```

The example above creates a boolean query:

```
(ny OR (new AND york)) city
```

that matches documents with the term `ny` or the conjunction `new AND york`. By default the parameter `auto_generate_synonyms_phrase_query` is set to `true`

### 匹配布尔前缀查询

A `match_bool_prefix` query analyzes its input and constructs a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) from the terms. Each term except the last is used in a `term` query. The last term is used in a `prefix` query. A `match_bool_prefix` query such as

`match_bool_prefix` 查询分析其输入并根据术语构造一个 `bool` 查询。

除了最后一个term外，每个术语都用于“term”查询。 最后一个术语用于“前缀”查询。 一个 `match_bool_prefix` 查询，例如

```console
GET /_search
{
  "query": {
    "match_bool_prefix" : {
      "message" : "quick brown f"
    }
  }
}
```

where analysis produces the terms `quick`, `brown`, and `f` is similar to the following `bool` query

```console
GET /_search
{
  "query": {
    "bool" : {
      "should": [
        { "term": { "message": "quick" }},
        { "term": { "message": "brown" }},
        { "prefix": { "message": "f"}}
      ]
    }
  }
}
```

An important difference between the `match_bool_prefix` query and [`match_phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase-prefix.html) is that the `match_phrase_prefix` query matches its terms as a phrase, but the `match_bool_prefix` query can match its terms in any position. 

The example `match_bool_prefix` query above could match a field containing `quick brown fox`, but it could also match `brown fox quick`. It could also match a field containing the term `quick`, the term `brown` and a term starting with `f`, appearing in any position.

#### 参数

By default, `match_bool_prefix` queries' input text will be analyzed using the analyzer from the queried field’s mapping. A different search analyzer can be configured with the `analyzer` parameter

默认情况下，“match_bool_prefix”查询的输入文本将使用来自查询字段映射的分析器进行分析。 可以使用 `analyzer` 参数配置不同的搜索分析器

```console
GET /_search
{
  "query": {
    "match_bool_prefix": {
      "message": {
        "query": "quick brown f",
        "analyzer": "keyword"
      }
    }
  }
}
```

`match_bool_prefix` queries support the [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html) and `operator` parameters as described for the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-boolean), applying the setting to the constructed `bool` query. The number of clauses in the constructed `bool` query will in most cases be the number of terms produced by analysis of the query text.

match_bool_prefix支持minimum_should_match 和 match查询中的opreator参数，将设置应用于构造的`bool`查询。`bool` 在大多数情况下，构造查询中的子句数量将是通过分析查询文本产生的术语数量。

The [`fuzziness`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-fuzziness), `prefix_length`, `max_expansions`, `fuzzy_transpositions`, and `fuzzy_rewrite` parameters can be applied to the `term` subqueries constructed for all terms but the final term. They do not have any effect on the prefix query constructed for the final term.

[`fuzziness`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-fuzziness)，`prefix_length`， `max_expansions`，`fuzzy_transpositions`，和`fuzzy_rewrite`参数可被应用到`term`所有术语，而是最终术语构建子查询。它们对为最终术语构造的前缀查询没有任何影响。

## 匹配短语查询

The `match_phrase` query analyzes the text and creates a `phrase` query out of the analyzed text. For example:

`match_phrase` 查询分析文本并从分析的文本中创建一个 `phrase` 查询。 例如：

```console
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

A phrase query matches terms up to a configurable `slop` (which defaults to 0) in any order. Transposed terms have a slop of 2.

The `analyzer` can be set to control which analyzer will perform the analysis process on the text. It defaults to the field explicit mapping definition, or the default search analyzer, for example:

短语查询以任何顺序匹配最多可配置的 `slop`（默认为 0）的术语。 转置项的slop为 2。

`analyzer` 可以设置为控制哪个分析器将对文本执行分析过程。 它默认为字段显式映射定义，或默认搜索分析器，例如：

```console
GET /_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "this is a test",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

This query also accepts `zero_terms_query`, as explained in [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html).

### 匹配短语前缀查询

Returns documents that contain the words of a provided text, in the **same order** as provided. The last term of the provided text is treated as a [prefix](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html), matching any words that begin with that term.

**以**与提供的**相同的顺序**返回包含提供的文本的单词的文档。提供的文本的最后一个术语被视为 [前缀](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html)，匹配以该术语开头的任何单词。

#### 示例

以下搜索返回包含字段`quick brown f`中以开头的短语的文档 `message`。

此搜索将匹配`quick brown fox`或者`two quick brown ferrets`但不匹配`the fox is quick and brown`

```console
GET /_search
{
  "query": {
    "match_phrase_prefix": {
      "message": {
        "query": "quick brown f"
      }
    }
  }
}
```

####  `match_phrase_prefix`的参数

- **`<field>`**

  (Required, object) Field you wish to search.

##### field的参数

- **`query`**

  (Required, string) Text you wish to find in the provided `<field>`.The `match_phrase_prefix` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) any provided text into tokens before performing a search. The last term of this text is treated as a [prefix](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html), matching any words that begin with that term.

- **`analyzer`**

  (Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) used to convert text in the `query` value into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-time-analyzer) mapped for the `<field>`. If no analyzer is mapped, the index’s default analyzer is used.

- **`max_expansions`**

  (Optional, integer) Maximum number of terms to which the last provided term of the `query` value will expand. Defaults to `50`.

  （可选，整数）“查询”值的最后提供的术语将扩展到的最大术语数。 默认为“50”。

- **`slop`**

  (Optional, integer) Maximum number of positions allowed between matching tokens. Defaults to `0`. Transposed terms have a slop of `2`.

- **`zero_terms_query`**

  (Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:**`none` (Default)**No documents are returned if the `analyzer` removes all tokens.**`all`**Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html) query.

#### 笔记

##### 使用匹配短语前缀查询进行搜索自动补全

While easy to set up, using the `match_phrase_prefix` query for search autocompletion can sometimes produce confusing results.

For example, consider the query string `quick brown f`. This query works by creating a phrase query out of `quick` and `brown` (i.e. the term `quick` must exist and must be followed by the term `brown`). Then it looks at the sorted term dictionary to find the first 50 terms that begin with `f`, and adds these terms to the phrase query.

The problem is that the first 50 terms may not include the term `fox` so the phrase `quick brown fox` will not be found. This usually isn’t a problem as the user will continue to type more letters until the word they are looking for appears.

虽然易于设置，但使用“match_phrase_prefix”查询进行搜索自动完成有时会产生令人困惑的结果。

例如，考虑查询字符串“quick brown f”。 这个查询的工作原理是从`quick` 和`brown` 创建一个短语查询（即术语`quick` 必须存在并且后面必须跟有术语`brown`）。 然后它查看已排序的术语字典以查找以“f”开头的前 50 个术语，并将这些术语添加到短语查询中。

问题是前 50 个术语可能不包含术语“fox”，因此不会找到短语“quick brown fox”。 这通常不是问题，因为用户会继续输入更多字母，直到出现他们要查找的单词。

For better solutions for *search-as-you-type* see the [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester) and the [`search_as_you_type` field type](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html).

有关 *search-as-you-type* 的更好解决方案，请参阅 [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester) 和 [`search_as_you_type` 字段类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html)。

## 字段组合查询

The `combined_fields` query supports searching multiple text fields as if their contents had been indexed into one combined field. It takes a term-centric view of the query: first it analyzes the query string into individual terms, then looks for each term in any of the fields. This query is particularly useful when a match could span multiple text fields, for example the `title`, `abstract` and `body` of an article:

该`combined_fields`查询支持搜索多个文本字段，就好像它们的内容已被索引到一个组合字段中一样。它采用以术语为中心的查询视图：首先将查询字符串分析为单独的术语，然后在任何字段中查找每个术语。当匹配可以跨越多个文本字段时，此查询特别有用，例如文章的`title`、 `abstract`和`body`：

```console
GET /_search
{
  "query": {
    "combined_fields" : {
      "query":      "database systems",
      "fields":     [ "title", "abstract", "body"],
      "operator":   "and"
    }
  }
}
```

The `combined_fields` query takes a principled approach to scoring based on the simple BM25F formula described in [The Probabilistic Relevance Framework: BM25 and Beyond](http://www.staff.city.ac.uk/~sb317/papers/foundations_bm25_review.pdf). When scoring matches, the query combines term and collection statistics across fields. This allows it to score each match as if the specified fields had been indexed into a single combined field. (Note that this is a best attempt — `combined_fields` makes some approximations and scores will not obey this model perfectly.)

该`combined_fields`查询采用基于[概率相关性框架：BM25 及其他中](http://www.staff.city.ac.uk/~sb317/papers/foundations_bm25_review.pdf)描述的简单 BM25F 公式的原则性方法进行评分 。当对匹配进行评分时，查询会跨字段组合术语和集合统计信息。这允许它对每个匹配进行评分，就好像指定的字段已被索引到单个组合字段中一样。（请注意，这是最好的尝试—— `combined_fields`做一些近似，分数不会完全服从这个模型。）

>  Field number limit
>
> There is a limit on the number of fields that can be queried at once. It is defined by the `indices.query.bool.max_clause_count` [Search settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-settings.html) which defaults to 1024.



### 字段的boosting

Individual fields can be boosted with the caret (`^`) notation:

```console
GET /_search
{
  "query": {
    "combined_fields" : {
      "query" : "distributed consensus",
      "fields" : [ "title^2", "body" ] 
    }
  }
}
```

Field boosts are interpreted according to the combined field model. For example, if the `title` field has a boost of 2, the score is calculated as if each term in the title appeared twice in the synthetic combined field.

The `combined_fields` query requires that field boosts are greater than or equal to 1.0. Field boosts are allowed to be fractional.

###  `combined_fields`的一级参数

- **`fields`**

  (Required, array of strings) List of fields to search. Field wildcard patterns are allowed. Only [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) fields are supported, and they must all have the same search [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html).

- **`query`**

  (Required, string) Text to search for in the provided `<fields>`.The `combined_fields` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) the provided text before performing a search.

- **`auto_generate_synonyms_phrase_query`**

  (Optional, Boolean) If `true`, [match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html) queries are automatically created for multi-term synonyms. Defaults to `true`.See [Use synonyms with match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-synonyms) for an example.

- **`operator`**

  (Optional, string) Boolean logic used to interpret text in the `query` value. Valid values are:**`or` (Default)**For example, a `query` value of `database systems` is interpreted as `database OR systems`.**`and`**For example, a `query` value of `database systems` is interpreted as `database AND systems`.

- **`minimum_should_match`**

  (Optional, string) Minimum number of clauses that must match for a document to be returned. See the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html) for valid values and more information.

- **`zero_terms_query`**

  (Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:**`none` (Default)**No documents are returned if the `analyzer` removes all tokens.**`all`**Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html) query.See [Zero terms query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-zero) for an example.

### 与multi_match比较

The `combined_fields` query provides a principled way of matching and scoring across multiple [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) fields. To support this, it requires that all fields have the same search [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html).

If you want a single query that handles fields of different types like keywords or numbers, then the [`multi_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html) query may be a better fit. It supports both text and non-text fields, and accepts text fields that do not share the same analyzer.

The main `multi_match` modes `best_fields` and `most_fields` take a field-centric view of the query. In contrast, `combined_fields` is term-centric: `operator` and `minimum_should_match` are applied per-term, instead of per-field. Concretely, a query like

该`combined_fields`查询提供了一种跨多个[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)字段进行匹配和评分的原则性方法。为了支持这一点，它要求所有字段都具有相同的 search [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html)。

如果您想要处理不同类型字段（如关键字或数字）的单个查询，那么该[`multi_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html) 查询可能更合适。它支持文本和非文本字段，并接受不共享相同分析器的文本字段。

主要`multi_match`模式`best_fields`和`most_fields`以字段为中心的查询视图。相反，`combined_fields`以术语为中心：`operator`并且`minimum_should_match`按术语应用，而不是按字段应用。具体来说，像这样的查询：

```console
GET /_search
{
  "query": {
    "combined_fields" : {
      "query":      "database systems",
      "fields":     [ "title", "abstract"],
      "operator":   "and"
    }
  }
}
```

is executed as

```
+(combined("database", fields:["title" "abstract"]))
+(combined("systems", fields:["title", "abstract"]))
```

In other words, each term must be present in at least one field for a document to match.

The `cross_fields` `multi_match` mode also takes a term-centric approach and applies `operator` and `minimum_should_match per-term`. The main advantage of `combined_fields` over `cross_fields` is its robust and interpretable approach to scoring based on the BM25F algorithm.

换句话说，每个术语必须至少出现在一个字段中才能匹配文档。

该`cross_fields` `multi_match`模式还采用以术语为中心的方法并应用`operator`和`minimum_should_match per-term`。`combined_fields`over的主要优点 `cross_fields`是其基于 BM25F 算法的稳健且可解释的评分方法。

> Custom similarities
>
> The `combined_fields` query currently only supports the `BM25` similarity (which is the default unless a [custom similarity](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-similarity.html) is configured). [Per-field similarities](https://www.elastic.co/guide/en/elasticsearch/reference/current/similarity.html) are also not allowed. Using `combined_fields` in either of these cases will result in an error.

## Multi-match查询

The `multi_match` query builds on the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) to allow multi-field queries:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
```

|      | The query string.         |
| ---- | ------------------------- |
|      | The fields to be queried. |

### `fields` and per-field加权

Fields can be specified with wildcards, eg:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] 
    }
  }
}
```

|      | Query the `title`, `first_name` and `last_name` fields. |
| ---- | ------------------------------------------------------- |
|      |                                                         |

Individual fields can be boosted with the caret (`^`) notation:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ] 
    }
  }
}
```

|      | The query multiplies the `subject` field’s score by three but leaves the `message` field’s score unchanged. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

If no `fields` are provided, the `multi_match` query defaults to the `index.query.default_field` index settings, which in turn defaults to `*`. `*` extracts all fields in the mapping that are eligible to term queries and filters the metadata fields. All extracted fields are then combined to build a query.

There is a limit on the number of fields that can be queried at once. It is defined by the `indices.query.bool.max_clause_count` [Search settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-settings.html) which defaults to 1024.

#### Types of `multi_match` query:

The way the `multi_match` query is executed internally depends on the `type` parameter, which can be set to:

| `best_fields`   | (**default**) Finds documents which match any field, but uses the `_score` from the best field. See [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-best-fields). |
| --------------- | ------------------------------------------------------------ |
| `most_fields`   | Finds documents which match any field and combines the `_score` from each field. See [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-most-fields). |
| `cross_fields`  | Treats fields with the same `analyzer` as though they were one big field. Looks for each word in **any** field. See [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-cross-fields). |
| `phrase`        | Runs a `match_phrase` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-phrase). |
| `phrase_prefix` | Runs a `match_phrase_prefix` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-phrase). |
| `bool_prefix`   | Creates a `match_bool_prefix` query on each field and combines the `_score` from each field. See [`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-bool-prefix). |

### `best_fields`[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

The `best_fields` type is most useful when you are searching for multiple words best found in the same field. For instance “brown fox” in a single field is more meaningful than “brown” in one field and “fox” in the other.

The `best_fields` type generates a [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) for each field and wraps them in a [`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-dis-max-query.html) query, to find the single best matching field. For instance, this query:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/886.console) 

would be executed as:

```console
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "subject": "brown fox" }},
        { "match": { "message": "brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/887.console) 

Normally the `best_fields` type uses the score of the **single** best matching field, but if `tie_breaker` is specified, then it calculates the score as follows:

- the score from the best matching field
- plus `tie_breaker * _score` for all other matching fields

Also, accepts `analyzer`, `boost`, `operator`, `minimum_should_match`, `fuzziness`, `lenient`, `prefix_length`, `max_expansions`, `fuzzy_rewrite`, `zero_terms_query`, `cutoff_frequency`, `auto_generate_synonyms_phrase_query` and `fuzzy_transpositions`, as explained in [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html).

### `operator` and `minimum_should_match`

The `best_fields` and `most_fields` types are *field-centric* — they generate a `match` query **per field**. This means that the `operator` and `minimum_should_match` parameters are applied to each field individually, which is probably not what you want.

Take this query for example:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "best_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and" 
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/888.console) 

|      | All terms must be present. |
| ---- | -------------------------- |
|      |                            |

This query is executed as:

```
  (+first_name:will +first_name:smith)
| (+last_name:will  +last_name:smith)
```

In other words, **all terms** must be present **in a single field** for a document to match.

The [`combined_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-combined-fields-query.html) query offers a term-centric approach that handles `operator` and `minimum_should_match` on a per-term basis. The other multi-match mode [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-cross-fields) also addresses this issue.

### `most_fields`[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

The `most_fields` type is most useful when querying multiple fields that contain the same text analyzed in different ways. For instance, the main field may contain synonyms, stemming and terms without diacritics. A second field may contain the original terms, and a third field might contain shingles. By combining scores from all three fields we can match as many documents as possible with the main field, but use the second and third fields to push the most similar results to the top of the list.

This query:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown fox",
      "type":       "most_fields",
      "fields":     [ "title", "title.original", "title.shingles" ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/889.console) 

would be executed as:

```console
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":          "quick brown fox" }},
        { "match": { "title.original": "quick brown fox" }},
        { "match": { "title.shingles": "quick brown fox" }}
      ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/890.console) 

The score from each `match` clause is added together, then divided by the number of `match` clauses.

Also, accepts `analyzer`, `boost`, `operator`, `minimum_should_match`, `fuzziness`, `lenient`, `prefix_length`, `max_expansions`, `fuzzy_rewrite`, `zero_terms_query` and `cutoff_frequency`, as explained in [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html), but **see [`operator` and `minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#operator-min)**.

### `phrase` and `phrase_prefix`[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

The `phrase` and `phrase_prefix` types behave just like [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-best-fields), but they use a `match_phrase` or `match_phrase_prefix` query instead of a `match` query.

This query:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "phrase_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/891.console) 

would be executed as:

```console
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase_prefix": { "subject": "quick brown f" }},
        { "match_phrase_prefix": { "message": "quick brown f" }}
      ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/892.console) 

Also, accepts `analyzer`, [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html), `lenient` and `zero_terms_query` as explained in [Match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html), as well as `slop` which is explained in [Match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html). Type `phrase_prefix` additionally accepts `max_expansions`.

### `phrase`, `phrase_prefix` and `fuzziness`

The `fuzziness` parameter cannot be used with the `phrase` or `phrase_prefix` type.

### `cross_fields`[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

The `cross_fields` type is particularly useful with structured documents where multiple fields **should** match. For instance, when querying the `first_name` and `last_name` fields for “Will Smith”, the best match is likely to have “Will” in one field and “Smith” in the other.

This sounds like a job for [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-most-fields) but there are two problems with that approach. The first problem is that `operator` and `minimum_should_match` are applied per-field, instead of per-term (see [explanation above](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#operator-min)).

The second problem is to do with relevance: the different term frequencies in the `first_name` and `last_name` fields can produce unexpected results.

For instance, imagine we have two people: “Will Smith” and “Smith Jones”. “Smith” as a last name is very common (and so is of low importance) but “Smith” as a first name is very uncommon (and so is of great importance).

If we do a search for “Will Smith”, the “Smith Jones” document will probably appear above the better matching “Will Smith” because the score of `first_name:smith` has trumped the combined scores of `first_name:will` plus `last_name:smith`.

One way of dealing with these types of queries is simply to index the `first_name` and `last_name` fields into a single `full_name` field. Of course, this can only be done at index time.

The `cross_field` type tries to solve these problems at query time by taking a *term-centric* approach. It first analyzes the query string into individual terms, then looks for each term in any of the fields, as though they were one big field.

A query like:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "cross_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and"
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/893.console) 

is executed as:

```
+(first_name:will last_name:will)
+(first_name:smith last_name:smith)
```

In other words, **all terms** must be present **in at least one field** for a document to match. (Compare this to [the logic used for `best_fields` and `most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#operator-min).)

That solves one of the two problems. The problem of differing term frequencies is solved by *blending* the term frequencies for all fields in order to even out the differences.

In practice, `first_name:smith` will be treated as though it has the same frequencies as `last_name:smith`, plus one. This will make matches on `first_name` and `last_name` have comparable scores, with a tiny advantage for `last_name` since it is the most likely field that contains `smith`.

Note that `cross_fields` is usually only useful on short string fields that all have a `boost` of `1`. Otherwise boosts, term freqs and length normalization contribute to the score in such a way that the blending of term statistics is not meaningful anymore.

If you run the above query through the [Validate](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-validate.html), it returns this explanation:

```
+blended("will",  fields: [first_name, last_name])
+blended("smith", fields: [first_name, last_name])
```

Also, accepts `analyzer`, `boost`, `operator`, `minimum_should_match`, `lenient`, `zero_terms_query` and `cutoff_frequency`, as explained in [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html).

The `cross_fields` type blends field statistics in a way that does not always produce well-formed scores (for example scores can become negative). As an alternative, you can consider the [`combined_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-combined-fields-query.html) query, which is also term-centric but combines field statistics in a more robust way.

#### `cross_field` and analysis[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

The `cross_field` type can only work in term-centric mode on fields that have the same analyzer. Fields with the same analyzer are grouped together as in the example above. If there are multiple groups, the query will use the best score from any group.

For instance, if we have a `first` and `last` field which have the same analyzer, plus a `first.edge` and `last.edge` which both use an `edge_ngram` analyzer, this query:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "fields":     [
        "first", "first.edge",
        "last",  "last.edge"
      ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/894.console) 

would be executed as:

```
    blended("jon", fields: [first, last])
| (
    blended("j",   fields: [first.edge, last.edge])
    blended("jo",  fields: [first.edge, last.edge])
    blended("jon", fields: [first.edge, last.edge])
)
```

In other words, `first` and `last` would be grouped together and treated as a single field, and `first.edge` and `last.edge` would be grouped together and treated as a single field.

Having multiple groups is fine, but when combined with `operator` or `minimum_should_match`, it can suffer from the [same problem](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#operator-min) as `most_fields` or `best_fields`.

You can easily rewrite this query yourself as two separate `cross_fields` queries combined with a `dis_max` query, and apply the `minimum_should_match` parameter to just one of them:

```console
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        {
          "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "first", "last" ],
            "minimum_should_match": "50%" 
          }
        },
        {
          "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "*.edge" ]
          }
        }
      ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/895.console) 

|      | Either `will` or `smith` must be present in either of the `first` or `last` fields |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

You can force all fields into the same group by specifying the `analyzer` parameter in the query.

```console
GET /_search
{
  "query": {
   "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "analyzer":   "standard", 
      "fields":     [ "first", "last", "*.edge" ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/896.console) 

|      | Use the `standard` analyzer for all fields. |
| ---- | ------------------------------------------- |
|      |                                             |

which will be executed as:

```
blended("will",  fields: [first, first.edge, last.edge, last])
blended("smith", fields: [first, first.edge, last.edge, last])
```

#### `tie_breaker`[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

By default, each per-term `blended` query will use the best score returned by any field in a group. Then when combining scores across groups, the query uses the best score from any group. The `tie_breaker` parameter can change the behavior for both of these steps:

| `0.0`           | Take the single best score out of (eg) `first_name:will` and `last_name:will` (default) |
| --------------- | ------------------------------------------------------------ |
| `1.0`           | Add together the scores for (eg) `first_name:will` and `last_name:will` |
| `0.0 < n < 1.0` | Take the single best score plus `tie_breaker` multiplied by each of the scores from other matching fields/ groups |

### `cross_fields` and `fuzziness`

The `fuzziness` parameter cannot be used with the `cross_fields` type.

### `bool_prefix`[edit](https://github.com/elastic/elasticsearch/edit/7.14/docs/reference/query-dsl/multi-match-query.asciidoc)

The `bool_prefix` type’s scoring behaves like [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-most-fields), but using a [`match_bool_prefix` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-bool-prefix-query.html) instead of a `match` query.

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "bool_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}
```

Copy as curl[View in Console](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/897.console) 

The `analyzer`, `boost`, `operator`, `minimum_should_match`, `lenient`, `zero_terms_query`, and `auto_generate_synonyms_phrase_query` parameters as explained in [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) are supported. The `fuzziness`, `prefix_length`, `max_expansions`, `fuzzy_rewrite`, and `fuzzy_transpositions` parameters are supported for the terms that are used to construct term queries, but do not have an effect on the prefix query constructed from the final term.

The `slop` and `cutoff_frequency` parameters are not supported by this query type.