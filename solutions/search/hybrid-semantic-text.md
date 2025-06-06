---
navigation_title: "Hybrid search with `semantic_text`"
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/semantic-text-hybrid-search.html
applies_to:
  stack:
  serverless:
---

# Hybrid search with `semantic_text` [semantic-text-hybrid-search]


This tutorial demonstrates how to perform hybrid search, combining semantic search with traditional full-text search.

In hybrid search, semantic search retrieves results based on the meaning of the text, while full-text search focuses on exact word matches. By combining both methods, hybrid search delivers more relevant results, particularly in cases where relying on a single approach may not be sufficient.

The recommended way to use hybrid search in the {{stack}} is following the `semantic_text` workflow. This tutorial uses the [`elasticsearch` service](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-inference-put-elasticsearch) for demonstration, but you can use any service and their supported models offered by the {{infer-cap}} API.

## Create an index mapping [hybrid-search-create-index-mapping]

The destination index will contain both the embeddings for semantic search and the original text field for full-text search. This structure enables the combination of semantic search and full-text search.

```console
PUT semantic-embeddings
{
  "mappings": {
    "properties": {
      "semantic_text": { <1>
        "type": "semantic_text"
      },
      "content": { <2>
        "type": "text",
        "copy_to": "semantic_text" <3>
      }
    }
  }
}
```

1. The name of the field to contain the generated embeddings for semantic search.
2. The name of the field to contain the original text for lexical search.
3. The textual data stored in the `content` field will be copied to `semantic_text` and processed by the {{infer}} endpoint.


::::{note}
If you want to run a search on indices that were populated by web crawlers or connectors, you have to [update the index mappings](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-put-mapping) for these indices to include the `semantic_text` field. Once the mapping is updated, you’ll need to run a full web crawl or a full connector sync. This ensures that all existing documents are reprocessed and updated with the new semantic embeddings, enabling hybrid search on the updated data.

::::



## Load data [semantic-text-hybrid-load-data]

In this step, you load the data that you later use to create embeddings from.

Use the `msmarco-passagetest2019-top1000` data set, which is a subset of the MS MARCO Passage Ranking data set. It consists of 200 queries, each accompanied by a list of relevant text passages. All unique passages, along with their IDs, have been extracted from that data set and compiled into a [tsv file](https://github.com/elastic/stack-docs/blob/main/docs/en/stack/ml/nlp/data/msmarco-passagetest2019-unique.tsv).

Download the file and upload it to your cluster using the [Data Visualizer](../../manage-data/ingest/upload-data-files.md) in the {{ml-app}} UI. After your data is analyzed, click **Override settings**. Under **Edit field names**, assign `id` to the first column and `content` to the second. Click **Apply**, then **Import**. Name the index `test-data`, and click **Import**. After the upload is complete, you will see an index named `test-data` with 182,469 documents.


## Reindex the data for hybrid search [hybrid-search-reindex-data]

Reindex the data from the `test-data` index into the `semantic-embeddings` index. The data in the `content` field of the source index is copied into the `content` field of the destination index. The `copy_to` parameter set in the index mapping creation ensures that the content is copied into the `semantic_text` field. The data is processed by the {{infer}} endpoint at ingest time to generate embeddings.

::::{note}
This step uses the reindex API to simulate data ingestion. If you are working with data that has already been indexed, rather than using the `test-data` set, reindexing is still required to ensure that the data is processed by the {{infer}} endpoint and the necessary embeddings are generated.

::::


```console
POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "test-data",
    "size": 10 <1>
  },
  "dest": {
    "index": "semantic-embeddings"
  }
}
```

1. The default batch size for reindexing is 1000. Reducing size to a smaller number makes the update of the reindexing process quicker which enables you to follow the progress closely and detect errors early.


The call returns a task ID to monitor the progress:

```console
GET _tasks/<task_id>
```

Reindexing large datasets can take a long time. You can test this workflow using only a subset of the dataset.

To cancel the reindexing process and generate embeddings for the subset that was reindexed:

```console
POST _tasks/<task_id>/_cancel
```


## Perform hybrid search [hybrid-search-perform-search]

After reindexing the data into the `semantic-embeddings` index, you can perform hybrid search to combine semantic and lexical search results. Choose between [retrievers](retrievers-overview.md) or [{{esql}}](/explore-analyze/query-filter/languages/esql.md) syntax to execute the query.

::::{tab-set}
:group: query-type

:::{tab-item} Query DSL
:sync: retrievers

This example uses [retrievers syntax](retrievers-overview.md) with [reciprocal rank fusion (RRF)](elasticsearch://reference/elasticsearch/rest-apis/reciprocal-rank-fusion.md). RRF is a technique that merges the rankings from both semantic and lexical queries, giving more weight to results that rank high in either search. This ensures that the final results are balanced and relevant.

```console
GET semantic-embeddings/_search
{
  "retriever": {
    "rrf": {
      "retrievers": [
        {
          "standard": { <1>
            "query": {
              "match": {
                "content": "How to avoid muscle soreness while running?" <2>
              }
            }
          }
        },
        {
          "standard": { <3>
            "query": {
              "semantic": {
                "field": "semantic_text", <4>
                "query": "How to avoid muscle soreness while running?"
              }
            }
          }
        }
      ]
    }
  }
}
```

1. The first `standard` retriever represents the traditional lexical search.
2. Lexical search is performed on the `content` field using the specified phrase.
3. The second `standard` retriever refers to the semantic search.
4. The `semantic_text` field is used to perform the semantic search.


After performing the hybrid search, the query will return the combined top 10 documents for both semantic and lexical search criteria. The results include detailed information about each document.

```console-result
{
  "took": 107,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 473,
      "relation": "eq"
    },
    "max_score": null,
    "hits": [
      {
        "_index": "semantic-embeddings",
        "_id": "wv65epIBEMBRnhfTsOFM",
        "_score": 0.032786883,
        "_rank": 1,
        "_source": {
          "semantic_text": {
            "inference": {
              "inference_id": "my-elser-endpoint",
              "model_settings": {
                "task_type": "sparse_embedding"
              },
              "chunks": [
                {
                  "text": "What so many out there do not realize is the importance of what you do after you work out. You may have done the majority of the work, but how you treat your body in the minutes and hours after you exercise has a direct effect on muscle soreness, muscle strength and growth, and staying hydrated. Cool Down. After your last exercise, your workout is not over. The first thing you need to do is cool down. Even if running was all that you did, you still should do light cardio for a few minutes. This brings your heart rate down at a slow and steady pace, which helps you avoid feeling sick after a workout.",
                  "embeddings": {
                    "exercise": 1.571044,
                    "after": 1.3603843,
                    "sick": 1.3281639,
                    "cool": 1.3227621,
                    "muscle": 1.2645415,
                    "sore": 1.2561599,
                    "cooling": 1.2335974,
                    "running": 1.1750668,
                    "hours": 1.1104802,
                    "out": 1.0991782,
                    "##io": 1.0794281,
                    "last": 1.0474665,
                   (...)
                  }
                }
              ]
            }
          },
          "id": 8408852,
          "content": "What so many out there do not realize is the importance of (...)"
        }
      }
    ]
  }
}
```
:::

:::{tab-item} ES|QL
:sync: esql

The ES|QL approach uses a combination of the match operator `:` and the match function `match()` to perform hybrid search.

```console
POST /_query?format=txt
{
  "query": """
    FROM semantic-embeddings METADATA _score <1>
    | WHERE content: "muscle soreness running?" OR match(semantic_text, "How to avoid muscle soreness while running?", { "boost": 0.75 }) <2> <3>
    | SORT _score DESC <4>
    | LIMIT 1000
  """
}
```
1. The `METADATA _score` clause is used to return the score of each document
2. The [match (`:`) operator](elasticsearch://reference/query-languages/esql/functions-operators/operators.md#esql-match-operator) is used on the `content` field for standard keyword matching
3. Semantic search using the `match()` function on the `semantic_text` field with a boost of `0.75`
4. Sorts by descending score and limits to 1000 results
:::
::::



