# elasticsearch-kuromoji-docker
ElasticSearch（kuromoji）で日本語検索周りをチェックするためのdocker関連ファイル群

## usage
```sh
docker-compose up --build
```

kiberaからコマンドを叩いてテストしました。  
access: http://localhost:5601
Menu > Management > Dev Tools

## sample command
[【Elasticsearch】kuromoji analyzerで出来ることと設定の解説 \- JPDEBUG\.COM](https://jpdebug.com/p/941707)  
参考サイトを見ながら叩いてみたコマンド
```
GET /_nodes

PUT my_ja_map2
{
  "settings": { 
    "analysis": {
      "analyzer": {
        "my_ja_analyzer": {
          "type": "custom",
          "tokenizer": "kuromoji_tokenizer",
          "char_filter": [
            "icu_normalizer",
            "kuromoji_iteration_mark"
          ],
          "filter": [
            "kuromoji_baseform",
            "kuromoji_part_of_speech",
            "ja_stop",
            "kuromoji_number",
            "kuromoji_stemmer",
            "kuromoji_readingform"
          ]
        }
      }
    }
  }
}

GET my_ja_map/_analyze
{
  "tokenizer": {
    "ja_kuromoji_tokenizer": {
      "mode": "search",
      "type": "kuromoji_tokenizer",
      "discard_compound_token": true,
      "user_dictionary_rules": [
        "唐揚げ,唐揚げ,カラアゲ,カスタム名詞"
      ]
    }
  },
  "filter": [
    "kuromoji_readingform"
  ],
  "text": "唐揚げ弁当"
}

GET my_ja_map/_search
{
  "query": {
    "match_all": {}
  }
}

GET my_ja_map2/_search
{
  "query": {
    "match":{"name": "カラアゲ"}
  }
}


PUT my_ja_map2/_doc/1
{
  "id": 1,
  "slug": "item1",
  "name": "カラアゲ定食"
}

PUT my_ja_map2/_doc/2
{
  "id": 2,
  "slug": "item2",
  "name": "からあげ弁当"
}

PUT my_ja_map2/_doc/3
{
  "id": 3,
  "slug": "item3",
  "name": "唐揚げ定食"
}

PUT my_ja_map2/_doc/4
{
  "id": 4,
  "slug": "item4",
  "name": "から揚げべんとう"
}



PUT my_fulltext_search
{
  "settings": {
    "analysis": {
      "char_filter": {
        "normalize": {
          "type": "icu_normalizer",
          "name": "nfkc",
          "mode": "compose"
        }
      },
      "tokenizer": {
        "ja_kuromoji_tokenizer": {
          "mode": "search",
          "type": "kuromoji_tokenizer",
          "discard_compound_token": true,
          "user_dictionary_rules": [
            "東京スカイツリー,東京 スカイツリー,トウキョウ スカイツリー,カスタム名詞"
          ]
        },
        "ja_ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 2,
          "max_gram": 2,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      },
      "filter": {
        "ja_index_synonym": {
          "type": "synonym",
          "lenient": false,
          "synonyms": [
            
          ]
        },
        "ja_search_synonym": {
          "type": "synonym_graph",
          "lenient": false,
          "synonyms": [
            "米国, アメリカ",
            "東京大学, 東大"
          ]
        }
      },
      "analyzer": {
        "ja_kuromoji_index_analyzer": {
          "type": "custom",
          "char_filter": [
            "normalize"
          ],
          "tokenizer": "ja_kuromoji_tokenizer",
          "filter": [
            "kuromoji_baseform",
            "kuromoji_part_of_speech",
            "ja_index_synonym",
            "cjk_width",
            "ja_stop",
            "kuromoji_stemmer",
            "lowercase"
          ]
        },
        "ja_kuromoji_search_analyzer": {
          "type": "custom",
          "char_filter": [
            "normalize"
          ],
          "tokenizer": "ja_kuromoji_tokenizer",
          "filter": [
            "kuromoji_baseform",
            "kuromoji_part_of_speech",
            "ja_search_synonym",
            "cjk_width",
            "ja_stop",
            "kuromoji_stemmer",
            "lowercase"
          ]
        },
        "ja_ngram_index_analyzer": {
          "type": "custom",
          "char_filter": [
            "normalize"
          ],
          "tokenizer": "ja_ngram_tokenizer",
          "filter": [
            "lowercase"
          ]
        },
        "ja_ngram_search_analyzer": {
          "type": "custom",
          "char_filter": [
            "normalize"
          ],
          "tokenizer": "ja_ngram_tokenizer",
          "filter": [
            "ja_search_synonym",
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "search_analyzer": "ja_kuromoji_search_analyzer",
        "analyzer": "ja_kuromoji_index_analyzer",
        "fields": {
          "ngram": {
            "type": "text",
            "search_analyzer": "ja_ngram_search_analyzer",
            "analyzer": "ja_ngram_index_analyzer"
          }
        }
      }
    }
  }
}

POST _bulk
{"index": {"_index": "my_fulltext_search", "_id": 1}}
{"my_field": "アメリカ"}
{"index": {"_index": "my_fulltext_search", "_id": 2}}
{"my_field": "米国"}
{"index": {"_index": "my_fulltext_search", "_id": 3}}
{"my_field": "アメリカの大学"}
{"index": {"_index": "my_fulltext_search", "_id": 4}}
{"my_field": "東京大学"}
{"index": {"_index": "my_fulltext_search", "_id": 5}}
{"my_field": "帝京大学"}
{"index": {"_index": "my_fulltext_search", "_id": 6}}
{"my_field": "東京で夢の大学生活"}
{"index": {"_index": "my_fulltext_search", "_id": 7}}
{"my_field": "東京大学で夢の生活"}
{"index": {"_index": "my_fulltext_search", "_id": 8}}
{"my_field": "東大で夢の生活"}
{"index": {"_index": "my_fulltext_search", "_id": 9}}
{"my_field": "首都圏の大学　東京"}

GET my_fulltext_search/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "生活",
            "fields": [
              "my_field.ngram^1"
            ],
            "type": "phrase"
          }
        }
      ],
      "should": [
        {
          "multi_match": {
            "query": "とうきょう",
            "fields": [
              "my_field^1"
            ],
            "type": "phrase"
          }
        }
      ]
    }
  }
}
```