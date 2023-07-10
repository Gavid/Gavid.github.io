[TOC]



# ES中文文档

https://doc.codingdict.com/elasticsearch/435/

# ElasticSearch权威指南：深入搜索

https://cloud.tencent.com/developer/article/1803780?areaSource=104001.150&traceId=9cx6haVjdymgc9ypEFzY4

# ES文本分析

## 概述

> 文本分析使 Elasticsearch 能够执行全文搜索，其中搜索返回所有*相关*结果，而不仅仅是完全匹配。

### ES 的分词器组成

1、character filter：在一段文本进行分词之前，先进行预处理，比如说最常见的就是，过滤html标签（<span>hello<span> --> hello），& --> and（I&you --> I and you）

> * html_strip : 预处理HTML文本。

2、tokenizer：分词，hello you and me --> hello, you, and, me

> * `标准分词器` 
> *  `icu_分词器`
>

3、token filter：lowercase，stop word，synonymom，dogs --> dog，liked --> like，Tom --> tom，a/the/an --> 干掉，mother --> mom，small --> little

> * asciifolding: 不仅仅能去掉变音符号。它会把Unicode字符转化为ASCII来表示;
> * lowercase: 将token中的字符都转换成小写形式。

stop word 停用词： 了 的 呢。

### 索引与搜索分析

1. 文本分析发生在以下两次时间：

- **索引时间**

  索引文档时，任何[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)字段值都会被分析。

- **搜索时间**

  在`text`字段上运行[全文搜索](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html)时，查询字符串（用户正在搜索的文本）会被分析。搜索时间也称为*查询时间*。

每次使用的分析器或分析规则集分别称为*索引分析器*或*搜索分析器*。

2. 索引和搜索分析器如何协同工作

   在大多数情况下，应该在索引和搜索时使用相同的分析器。这可确保字段的值和查询字符串更改为相同形式的标记。反过来，这可以确保令牌在搜索过程中按预期匹配。

3. 何时使用不同的搜索分析器

   虽然不太常见，但有时在索引和搜索时使用不同的分析器是有意义的。为了实现这一点，Elasticsearch 允许您 [指定一个单独的搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-analyzer)。

   通常，仅当对字段值和查询字符串使用相同形式的标记会创建意外或不相关的搜索匹配时，才应指定单独的搜索分析器。

### 词干（*Stemming*）

*词干提取*是将单词简化为其词根形式的过程。这确保了搜索期间单词匹配的变体。

例如，`walking`和`walked`可以词干到相同的词根： `walk`。一旦词干化，出现的任何一个词都会在搜索中与另一个词匹配。

词干与语言有关，但通常涉及从单词中删除前缀和后缀。

在某些情况下，词干词的词根形式可能不是真正的词。例如，`jumping`and`jumpiness`都可以被词干化为`jumpi`. 虽然`jumpi` 不是真正的英文单词，但对搜索没有影响；如果一个词的所有变体都简化为相同的词根形式，它们将正确匹配。

因为词干会改变标记，我们建议在[索引和搜索分析](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-index-search-time.html)期间使用相同的词干标记过滤器。

* 算法词干过滤器

  算法词干过滤器对每个单词应用一系列规则以将其简化为词根形式。例如，英语的算法词干分析器可能会删除复数词末尾的`-s` 和`-es`后缀。

  算法词干分析器有几个优点：

  - 它们几乎不需要设置，通常开箱即用。
  - 他们使用很少的内存。
  - 它们通常比[字典词干分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#dictionary-stemmers)更快。

  但是，大多数算法词干分析器仅更改单词的现有文本。这意味着它们可能不适用于不包含其词根形式的不规则单词

  - [`stemmer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html)，它为多种语言提供算法词干，其中一些具有附加变体。
  - [`kstem`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-kstem-tokenfilter.html)，一种结合了算法词干和内置词典的英语词干分析器。
  - [`porter_stem`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-porterstem-tokenfilter.html)，我们推荐的英语算法词干分析器。
  - [`snowball`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-tokenfilter.html)，它对多种语言使用 基于[Snowball](https://snowballstem.org/)的词干提取规则。

* 字典词干过滤器

  词典词干过滤器在提供的词典中查找词，用词典中的词干替换未词干的词变体。

  理论上，字典词干分析器非常适合：

  - 词干不规则词
  - 区分拼写相似但概念上不相关的单词，例如：
    - `organ` 和 `organization`
    - `broker` 和 `broken`

  在实践中，算法词干分析器通常优于字典词干分析器。这是因为字典词干分析器具有以下缺点：

  - **字典质量**
    字典词干分析器的**好坏**取决于它的字典。为了运行良好，这些词典必须包含大量单词、定期更新并随着语言趋势而变化。通常，当一本字典可用时，它已经不完整，其中的一些条目已经过时了。
  - **大小和性能**
    字典词干分析器必须将其字典中的所有单词、前缀和后缀加载到内存中。这可能会使用大量 RAM。去除前缀和后缀的低质量词典也可能效率较低，这会显着减慢词干提取过程。

  可以使用[`hunspell`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-hunspell-tokenfilter.html)标记过滤器来执行字典词干提取。

  > Note:如果可用，我们建议在使用[`hunspell`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-hunspell-tokenfilter.html)标记过滤器之前尝试针对您的语言使用算法词干分析器。

* 控制词干过滤器

  有时词干提取可以产生拼写相似但概念上不相关的共享词根。例如，词干分析器可以将`skies`和 都减少`skiing`到相同的词根：`ski`。

  为了防止这种情况并更好地控制词干提取，您可以使用以下标记过滤器：

  - [`stemmer_override`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-override-tokenfilter.html)，它允许您定义用于提取特定标记的规则。
  - [`keyword_marker`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-marker-tokenfilter.html)，将指定的标记标记为关键字。关键字标记不会被后续的词干分析器标记过滤器阻止。
  - [`conditional`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-condition-tokenfilter.html)，可用于将标记标记为关键字，类似于`keyword_marker`过滤器。

  对于内置[语言分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html)，您还可以使用该 [`stem_exclusion`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html#_excluding_words_from_stemming)参数来指定一个不会被词干化的单词列表。

### Token graphs

当[标记器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html#analyzer-anatomy-tokenizer)将文本转换为标记流时，它还会记录以下内容：

- 的`position`流中的每个标记的
- 的`positionLength`，位置是令牌的跨距数

使用这些，您可以为流创建一个有 [向无环图](https://en.wikipedia.org/wiki/Directed_acyclic_graph)，称为*标记图*。在令牌图中，每个位置代表一个节点。每个标记代表一条边或弧，指向下一个位置。

![令牌图 qbf ex](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/analysis/token-graph-qbf-ex.svg)

## 配置文本分析器

### 测试文本分析器

### 配置ES内置分析器

### 创建自定义分析器

### 指定ES分析器（ES分析器的执行选取）

 * Elasticsearch 如何确定索引分析器

   Elasticsearch 通过按顺序检查以下参数来确定要使用的索引分析器：

   1. [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html)字段 的映射参数。请参阅[为字段指定分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-field-analyzer)。
   2. 该`analysis.analyzer.default`指数设置。请参阅[为索引指定默认分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-time-default-analyzer)。

   如果没有指定这些参数， 则使用[`standard`分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html)。

 * Elasticsearch 如何确定搜索分析器

   在搜索时，Elasticsearch 通过按顺序检查以下参数来确定要使用的分析器：

   1. [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html)搜索查询中 的参数。请参阅[为查询指定搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-query-analyzer)。
   2. [`search_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-analyzer.html)字段 的映射参数。请参阅[为字段指定搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-field-analyzer)。
   3. 该`analysis.analyzer.default_search`指数设置。请参阅[为索引指定默认搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-default-analyzer)。
   4. [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html)字段 的映射参数。请参阅[为字段指定分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-field-analyzer)。

   如果没有指定这些参数， 则使用[`standard`分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html)。

   > Note: 
   > 在大多数情况下，不需要指定不同的搜索分析器。这样做可能会对相关性产生负面影响并导致意外的搜索结果。
   >
   > 如果您选择指定单独的搜索分析器，我们建议您在部署到生产环境之前彻底 [测试您的分析配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/test-analyzer.html)。

## ES内置Analyzer介绍（共8+1个）

### **[标准分词器（standard）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-standard-analyzer.html)**

所述`standard`分析器将文本分为在字边界条件，如通过Unicode文本分割算法定义。它删除了大多数标点符号、小写术语，并支持删除停用词。

* 举例

```json
POST _analyze
{
  "analyzer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

[ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog's, bone ]
```

* 参数配置

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english_analyzer": {
          "type": "standard",
          "max_token_length": 5,
          "stopwords": "_english_"
        }
      }
    }
  }
}
```

| 参数名             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `max_token_length` | The maximum token length. If a token is seen that exceeds this length then it is split at `max_token_length` intervals. Defaults to `255`. |
| `stopwords`        | A pre-defined stop words list like `_english_` or an array containing a list of stop words. Defaults to `_none_`. |
| `stopwords_path`   | The path to a file containing stop words.                    |

* 自定义
```json
PUT /standard_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_standard": {
          "tokenizer": "standard",
          "filter": [
            "lowercase"       
          ]
        }
      }
    }
  }
}
```

### **[简单分词器（simple）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-simple-analyzer.html)**

该`simple`分析仪将文本分为方面每当遇到一个字符是不是字母。它小写所有术语。

* 举例

```json
POST _analyze
{
  "analyzer": "simple",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

[ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```

* 无配置项
* 自定义

```json
PUT /simple_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_simple": {
          "tokenizer": "lowercase",
          "filter": [         
          ]
        }
      }
    }
  }
}
```

### **[空白分词器（whitespace）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-whitespace-analyzer.html)**

该`whitespace`分析器是将文本使用“空白段”进行分词。

* 举例

```json
POST _analyze
{
  "analyzer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

[ The, 2, QUICK, Brown-Foxes, jumped, over, the, lazy, dog's, bone. ]
```

* 无配置项
* 自定义

```json
PUT /whitespace_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_whitespace": {
          "tokenizer": "whitespace",
          "filter": [         
          ]
        }
      }
    }
  }
}
```

### **[停止分词器（stop）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-stop-analyzer.html)**

该`stop`分析仪是像`simple`仪，而且还支持去除停止词。

* 举例

```json
POST _analyze
{
  "analyzer": "stop",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

[ quick, brown, foxes, jumped, over, lazy, dog, s, bone ]
```

* 配置项

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_stop_analyzer": {
          "type": "stop",
          "stopwords": ["the", "over"]
        }
      }
    }
  }
}
```

| 参数名           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `stopwords`      | A pre-defined stop words list like `_english_` or an array containing a list of stop words. Defaults to `_english_`. |
| `stopwords_path` | The path to a file containing stop words. This path is relative to the Elasticsearch `config` directory. |

* 自定义

```json
PUT /stop_example
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stop": {
          "type":       "stop",
          "stopwords":  "_english_" 
        }
      },
      "analyzer": {
        "rebuilt_stop": {
          "tokenizer": "lowercase",
          "filter": [
            "english_stop"          
          ]
        }
      }
    }
  }
}
```

### **[关键字分词器（keyword）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-keyword-analyzer.html)**

所述`keyword`分析器是一个“空操作”分析器接受任何文本它被赋予并输出完全相同的文本作为一个单一的术语。

* 举例

```json
POST _analyze
{
  "analyzer": "keyword",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

[ The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. ]
```

* 无配置项
* 自定义

```json
PUT /keyword_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_keyword": {
          "tokenizer": "keyword",
          "filter": [         
          ]
        }
      }
    }
  }
}
```

### **[模式分词器（pattern）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-pattern-analyzer.html)**

所述`pattern`分析仪使用一个正则表达式的文本分成条款。它支持小写和停用词。正则表达式默认为`\W+`（或所有非单词字符）。

* 举例

```json
POST _analyze
{
  "analyzer": "pattern",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
[ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```

* 配置项

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_email_analyzer": {
          "type":      "pattern",
          "pattern":   "\\W|_", 
          "lowercase": true
        }
      }
    }
  }
}
```

| 参数名           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `pattern`        | A [Java regular expression](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html), defaults to `\W+`. |
| `flags`          | Java regular expression [flags](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html#field.summary). Flags should be pipe-separated, eg `"CASE_INSENSITIVE|COMMENTS"`. |
| `lowercase`      | Should terms be lowercased or not. Defaults to `true`.       |
| `stopwords`      | A pre-defined stop words list like `_english_` or an array containing a list of stop words. Defaults to `_none_`. |
| `stopwords_path` | The path to a file containing stop words.                    |

* 自定义

```json
PUT /pattern_example
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "split_on_non_word": {
          "type":       "pattern",
          "pattern":    "\\W+" 
        }
      },
      "analyzer": {
        "rebuilt_pattern": {
          "tokenizer": "split_on_non_word",
          "filter": [
            "lowercase"       
          ]
        }
      }
    }
  }
}
```

### **[语言分词器（`english` or `french`or ·····）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-lang-analyzer.html)**

Elasticsearch 提供了许多特定于语言的分析器，例如`english`或 `french`。



### **[唯一排序token分词器（fingerprint）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-fingerprint-analyzer.html)**

所述`fingerprint`分词器是一个专业的分词器，它产生单个标记。

* 举例

```json
POST _analyze
{
  "analyzer": "fingerprint",
  "text": "Yes yes, Gödel said this sentence is consistent and."
}

[ and consistent godel is said sentence this yes ]
```

* 配置项

```console
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_fingerprint_analyzer": {
          "type": "fingerprint",
          "stopwords": "_english_"
        }
      }
    }
  }
}
```

| 参数名            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `separator`       | The character to use to concatenate the terms. Defaults to a space. |
| `max_output_size` | The maximum token size to emit. Defaults to `255`. Tokens larger than this size will be discarded. |
| `stopwords`       | A pre-defined stop words list like `_english_` or an array containing a list of stop words. Defaults to `_none_`. |
| `stopwords_path`  | The path to a file containing stop words.                    |

* 自定义

```json
PUT /fingerprint_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_fingerprint": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding",
            "fingerprint"
          ]
        }
      }
    }
  }
}
```

### [自定义分词器（`custom`）](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-custom-analyzer.html) 

> 当内置分析器不能满足您的需求时，您可以创建一个 `custom`使用以下适当组合的分析器：
>
> - 零个或多个[char_filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-charfilters.html)
> - 一个[tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-tokenizers.html)
> - 零个或多个[token_filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-tokenfilters.html)。

* 配置项


| 参数名                   | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `tokenizer`              | A built-in or customised [tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-tokenizers.html). (Required) |
| `char_filter`            | An optional array of built-in or customised [character filters](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-charfilters.html). |
| `filter`                 | An optional array of built-in or customised [token filters](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-tokenfilters.html). |
| `position_increment_gap` | When indexing an array of text values, Elasticsearch inserts a fake "gap" between the last term of one value and the first term of the next value to ensure that a phrase query doesn’t match two terms from different array elements. Defaults to `100`. See [`position_increment_gap`](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/position-increment-gap.html) for more.（在索引文本值数组时，Elasticsearch 在一个值的最后一项和下一个值的第一项之间插入一个假的“间隙”，以确保短语查询不匹配来自不同数组元素的两个项。默认为 100。） |

* 配置举例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": { 
          "type": "custom", # Setting type to custom tells Elasticsearch that we are defining a custom analyzer.
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "punctuation",
          "filter": [
            "lowercase",
            "english_stop"
          ]
        }
      },
      "tokenizer": {
        "punctuation": { 
          "type": "pattern",
          "pattern": "[ .,!?]"
        }
      },
      "char_filter": {
        "emoticons": { 
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
      "filter": {
        "english_stop": { 
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I'm a :) person, and you?"
}
```

## ES中 Normalizers （规范化器）介绍

规范器类似于分析器，不同之处在于它们可能只发出一个标记。因此，它们没有分词器，只接受可用字符过滤器和标记过滤器的子集。仅允许基于每个字符工作的过滤器。例如，允许使用小写过滤器，但不允许使用词干过滤器，它需要将关键字作为一个整体来查看。可用于规范化器的当前过滤器列表如下：`arabic_normalization`, `asciifolding`, `bengali_normalization`, `cjk_width`, `decimal_digit`, `elision`, `german_normalization`, `hindi_normalization`, `indic_normalization`, `lowercase`, `persian_normalization`, `scandinavian_folding`, `serbian_normalization`, `sorani_normalization`, `uppercase`. 

> 简单来说：在 [Elasticsearch ]中处理字符串类型的数据时，如果我们想把整个字符串作为一个完整的 term 存储，我们通常会将其类型 `type` 设定为 `keyword`。但有时这种设定又会给我们带来麻烦，比如同一个数据再写入时由于没有做好清洗，导致大小写不一致，比如 `apple`、`Apple`两个实际都是 `apple`，但当我们去搜索 `apple`时却无法返回 `Apple`的文档。要解决这个问题，就需要 `Normalizer`出场了。

```json
DELETE test_normalizer# 自定义 normalizer
PUT test_normalizer
{
  "settings": {
    "analysis": {
      "normalizer": {
        "lowercase": {
          "type": "custom",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "type": {
        "type": "keyword"
      },
      "type_normalizer": {
        "type": "keyword",
        "normalizer": "lowercase"
      }
    }
  }
}
PUT test_normalizer/_doc/1
{
  "type": "apple",
  "type_normalizer": "apple"
}
PUT test_normalizer/_doc/2
{
  "type": "Apple",
  "type_normalizer": "Apple"
}
# 查询三
GET test_normalizer/_search
{
  "query": {
    "term":{
      "type":"aPple"
    }
  }
}
# 查询四
GET test_normalizer/_search
{
  "query": {
    "term":{
      "type_normalizer":"aPple"
    }
  }
}
```

我们第一步是自定义了名为 `lowercase`的 normalizer，其中`filter` 类似自定义分词器中的 `filter` ，但是可用的种类很少，详情大家可以查看官方文档。然后通过 `normalizer`属性设定到字段`type_normalizer`中，然后插入相同的2条文档。执行发现，`查询三`无结果返回，`查询四`返回2条文档。

1.文档写入时由于加入了 `normalizer`,所有的 `term`都会被做小写处理；
2.查询时搜索词同样采用有 `normalizer`的配置，因此处理后的 `term`也是小写的；
3.两边分词匹对，就得到了我们上面的结果。

## ES内置 Character Filter  介绍（共3个）

> *Character Filter*用于在将字符流传[递给标记器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)之前对其进行预处理。
>
> 字符过滤器以字符流的形式接收原始文本，并可以通过添加、删除或更改字符来转换流。例如，字符过滤器可用于将印度-阿拉伯数字 (٠ ١٢٣٤٥٦٧٨ ٩ ) 转换为它们的阿拉伯-拉丁数字 (0123456789)，或`<b>`从流中去除 HTML 元素等。
>
> Elasticsearch 有许多内置的字符过滤器，可用于构建 [自定义分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html)。
>
> * **[HTML条形字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-htmlstrip-charfilter.html)**
>
>   `html_strip`字符过滤器剥离了`<b>`等HTML元素，并解码了`&amp;`等HTML实体。
>
> * **[映射字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-mapping-charfilter.html)**
>
>   `mapping`字符过滤器将指定字符串的任何出现替换为指定的替换。
>
> * **[模式替换字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-replace-charfilter.html)**
>
>   `pattern_replace`字符过滤器将与正则表达式匹配的任何字符替换为指定的替换。

### HTML strip character

> 从文本中去除 HTML 元素并用它们的解码值替换 HTML 实体（例如，替换`&amp;`为`&`）。
>
> 该`html_strip`过滤器使用Lucene的 [HTMLStripCharFilter](https://lucene.apache.org/core/8_9_0/analyzers-common/org/apache/lucene/analysis/charfilter/HTMLStripCharFilter.html)。

* 举例

  ```bash
  GET /_analyze
  {
    "tokenizer": "keyword",
    "char_filter": [
      "html_strip"
    ],
    "text": "<p>I&apos;m so <b>happy</b>!</p>"
  }
  
  # [ \nI'm so happy!\n ]
  ```

* 配置

  **escaped_tags** ：（可选，字符串数组）不包含尖括号 ( `< >`)的 HTML 元素数组。当从文本中去除 HTML 时，过滤器会跳过这些 HTML 元素。例如，值`[ "p" ]`跳过`<p>`HTML 元素。

* 配置举例

  该`my_custom_html_strip_char_filter`过滤器跳过去除的`<b>` HTML元素。

  ```bash
  PUT my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "keyword",
            "char_filter": [
              "my_custom_html_strip_char_filter"
            ]
          }
        },
        "char_filter": {
          "my_custom_html_strip_char_filter": {
            "type": "html_strip",
            "escaped_tags": [
              "b"
            ]
          }
        }
      }
    }
  }
  ```

### Mapping character filter

>每当遇到与键相同的字符串时，它就会将它们替换为与该键关联的值。
>匹配是贪婪的；在某一点上匹配最长的模式获胜。替换允许为空字符串。
>该`mapping`过滤器使用Lucene的 [MappingCharFilter](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/charfilter/MappingCharFilter.html)。

* 举例

  使用`mapping`过滤器将印度-阿拉伯数字 (٠ ١٢٣٤٥٦٧٨ ٩ ) 转换为其阿拉伯-拉丁等价物 (0123456789)，将文本更改`My license plate is ٢٥٠١٥`为 `My license plate is 25015`.

  ```bash
  GET /_analyze
  {
    "tokenizer": "keyword",
    "char_filter": [
      {
        "type": "mapping",
        "mappings": [
          "٠ => 0",
          "١ => 1",
          "٢ => 2",
          "٣ => 3",
          "٤ => 4",
          "٥ => 5",
          "٦ => 6",
          "٧ => 7",
          "٨ => 8",
          "٩ => 9"
        ]
      }
    ],
    "text": "My license plate is ٢٥٠١٥"
  }
  
  # [ My license plate is 25015 ]
  ```

* 配置

  **mappings**：（必需*，字符串数组）映射数组，每个元素的形式为`key => value`.

  必须指定此参数或 mappings_path 参数。

  **mappings_path**：

  （必需*，字符串）包含`key => value`映射的文件的路径。

  此路径必须是绝对或相对于`config`位置的路径，并且文件必须是 UTF-8 编码的。文件中的每个映射必须由换行符分隔。

  必须指定此参数 或 mappings 参数。

* 配置举例

  该`my_mappings_char_filter`过滤器替换`:)`和`:(`表情符号与对应的文字。

  ```bash
  PUT /my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "char_filter": [
              "my_mappings_char_filter"
            ]
          }
        },
        "char_filter": {
          "my_mappings_char_filter": {
            "type": "mapping",
            "mappings": [
              ":) => _happy_",
              ":( => _sad_"
            ]
          }
        }
      }
    }
  }
  ```

### Pattern replace character filter

> pattern_replace 字符过滤器使用正则表达式来匹配应该用指定的替换字符串替换的字符。替换字符串可以引用正则表达式中的捕获组。
>
> Note: 
> 模式替换字符过滤器使用 [Java 正则表达式](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)。
> 写得不好的正则表达式可能会运行得很慢，甚至会抛出 StackOverflowError 并导致它正在运行的节点突然退出。
> 阅读有关[病态正则表达式以及如何避免它们的更多信息](https://www.regular-expressions.info/catastrophic.html)。

* 配置：

  **pattern**：一个[Java 正则表达式](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)。必需的。

  **replacement** ：替换字符串，可以参考使用捕获组 `$1`..`$9`语法，说明 [这里](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Matcher.html#appendReplacement-java.lang.StringBuffer-java.lang.String-)。

  **flags**：Java 正则表达式[标志](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html#field.summary)。标志应该用管道分隔，例如`"CASE_INSENSITIVE|COMMENTS"`.

* 配置举例

  我们配置了`pattern_replace`字符过滤器，用下划线替换数字中的任何嵌入破折号，即`123-456-789`→ `123_456_789`：

  ```bash
  PUT my-index-00001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "char_filter": [
              "my_char_filter"
            ]
          }
        },
        "char_filter": {
          "my_char_filter": {
            "type": "pattern_replace",
            "pattern": "(\\d+)-(?=\\d)",
            "replacement": "$1_"
          }
        }
      }
    }
  }
  
  POST my-index-00001/_analyze
  {
    "analyzer": "my_analyzer",
    "text": "My credit card is 123-456-789"
  }
  # [ My, credit, card, is, 123_456_789 ]
  ```

  > Note:使用更改原始文本长度的替换字符串可用于搜索，但会导致不正确的突出显示，如下例所示。

  此示例在遇到小写字母后跟大写字母（即`fooBarBaz`→ `foo Bar Baz`）时插入一个空格，允许单独查询驼峰式单词：

  ```bash
  PUT my-index-00001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "char_filter": [
              "my_char_filter"
            ],
            "filter": [
              "lowercase"
            ]
          }
        },
        "char_filter": {
          "my_char_filter": {
            "type": "pattern_replace",
            "pattern": "(?<=\\p{Lower})(?=\\p{Upper})",
            "replacement": " "
          }
        }
      }
    },
    "mappings": {
      "properties": {
        "text": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
  
  POST my-index-00001/_analyze
  {
    "analyzer": "my_analyzer",
    "text": "The fooBarBaz method"
  }
  
  # [ the, foo, bar, baz, method ]
  ```

  查询 for`bar`会正确找到文档，但是在结果上高亮会产生不正确的高亮，因为我们的字符过滤器改变了原始文本的长度：

  ```bash
  PUT my-index-00001/_doc/1?refresh
  {
    "text": "The fooBarBaz method"
  }
  
  GET my-index-00001/_search
  {
    "query": {
      "match": {
        "text": "bar"
      }
    },
    "highlight": {
      "fields": {
        "text": {}
      }
    }
  }
  
  # 返回结果
  {
    "timed_out": false,
    "took": $body.took,
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
      "max_score": 0.2876821,
      "hits": [
        {
          "_index": "my-index-00001",
          "_type": "_doc",
          "_id": "1",
          "_score": 0.2876821,
          "_source": {
            "text": "The fooBarBaz method"
          },
          "highlight": {
            "text": [
              "The foo<em>Ba</em>rBaz method" 
            ]
          }
        }
      ]
    }
  }
  ```

  

## ES内置 Tokenizer 介绍（共15个）

> 分*词器*接收字符流，将其分解为单独的 *标记*（通常是单个单词），并输出*标记*流。例如，[`whitespace`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-tokenizer.html)标记器在看到任何空格时将文本分解为标记。它会将文本 `"Quick brown fox!"`转换为`[Quick, brown, fox!]`。
>
> 分词器还负责记录以下内容：
>
> - 每个术语的 顺序或*位置*（用于短语和单词邻近查询）
> - 该术语所代表的原始单词的 开始和结束*字符偏移*（用于突出显示搜索片段）。
> - *令牌类型*，产生的每个术语的分类，例如`<ALPHANUM>`、 `<HANGUL>`、 或`<NUM>`。更简单的分析器只生成`word`令牌类型。
>
> Elasticsearch 有许多内置的分词器，可用于构建 [自定义分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html)。

### 以全文字为导向的 Tokenizer（7）

> 以下标记器通常用于将全文标记为单个词。

#### [标准分词器(standard)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-tokenizer.html)

> 标准标记器按照Unicode文本分割算法的定义，将文本划分为字界上的术语。它删除了大多数标点符号。它是大多数语言的最佳选择。

#### [字母标记器(letter)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-letter-tokenizer.html)

> 只要遇到非字母的字符，字母标记器就会将文本分成若干个术语。

#### [小写分词器(lowercase)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html)

> 小写标示器和字母标示器一样，只要遇到不是字母的字符，就会将文本分为术语，但它也会将所有术语小写。

#### [空白标记器(whitespace)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-tokenizer.html)

> 每当遇到任何空白字符时，空白标记器都会将文本划分为术语。

#### [UAX URL 电子邮件标记器(uax_url_email)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-uaxurlemail-tokenizer.html)

> uax_url_email标记器与标准标记器一样，只是它将URL和电子邮件地址识别为单一标记。

#### [经典分词器(classic)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-classic-tokenizer.html)

> 经典标记器是一种基于语法的英语标记器。

#### [泰国分词器(thai)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-thai-tokenizer.html)

> 泰语标记器将泰语文本分割成单词。

略

### 以部分文字为导向的 Tokenizer(2)

> 这些标记器将文本或单词分解成小片段，用于部分单词的匹配。

#### [N-Gram 分词器(ngram)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html)

> ngram tokenizer可以在遇到指定的字符列表（如空白或标点符号）中的任何一个时，将文本分解成单词，然后返回每个单词的n-grams：一个连续字母的滑动窗口，例如quick→[qu, ui, ic, ck]。

#### [边缘 N-Gram 分词器(edge_ngram)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html)

> Edge_ngram标记器在遇到指定的字符列表（如空白或标点符号）中的任何一个时，可以将文本分解成单词，然后返回每个单词的n个grams，这些grams被锚定在单词的开头，例如，quick → [q, qu, qui, quic, quick]。

### 以结构化文字为导向的 Tokenizer(6)

> 以下标记器通常用于结构化文本，如标识符、电子邮件地址、邮政编码和路径，而不是用于全文。

#### [关键字标记器(keyword)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-tokenizer.html)

> 关键词标记器是一个 "noop "标记器，它接受给定的任何文本，并输出与单个术语完全相同的文本。它可以与小写字母等标记过滤器结合起来，使分析出来的术语正常化。

#### [模式标记器(pattern)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-tokenizer.html)

> 模式标记器使用正则表达式，只要文本与单词分隔符匹配，就将其分割成术语，或者将匹配的文本作为术语捕获。

#### [简单模式标记器(simple_pattern)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-simplepattern-tokenizer.html)

> simple_pattern标记器使用正则表达式来捕获匹配的文本作为术语。它使用一个受限制的正则表达式特征子集，通常比模式标记器快。

#### [字符组标记器(char_group)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-chargroup-tokenizer.html)

>  char_group tokenizer可以通过字符集来配置，这通常比运行正则表达式的成本要低。

#### [简单模式拆分标记器(simple_pattern_split)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-simplepatternsplit-tokenizer.html)

> simple_pattern_split标记器使用与simple_pattern标记器相同的限制性正则表达式子集，但在匹配处对输入进行分割，而不是将匹配处作为术语返回。

#### [路径标记器(path_hierarchy)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html)

> path_hierarchy标记器接受一个像文件系统路径一样的分层值，在路径分隔符上进行分割，并为树中的每个组成部分发出一个术语，例如，/foo/bar/baz → [/foo, /foo/bar, /foo/bar/baz ]。



## ES内置 Token Filters (共48个)介绍

> 标记过滤器接受来自[标记](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)器的标记流， 并且可以修改标记（例如小写）、删除标记（例如删除停用词）或添加标记（例如同义词）。
>
> Elasticsearch 有许多内置的令牌过滤器，可用于构建[自定义分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html)。

#### 与词干相关的 Token Filters

##### 算法词干过滤器

>  算法词干处理器对每个单词应用一系列规则以将其简化为词根形式。例如，英语的算法词干分析器可能会删除复数词末尾的`-s` 和`-es`后缀。

###### Stemmer 标记过滤器(stemmer)

> 为多种语言提供[算法词干](https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#algorithmic-stemmers)，其中一些具有附加变体。有关支持的语言列表，请参阅 [`language`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html#analysis-stemmer-tokenfilter-language-parm)参数。
>
> 未指定时，过滤器使用英语的 [porter 词干提取算法](https://snowballstem.org/algorithms/porter/stemmer.html)。

* 举例

  ```bash
  GET /_analyze
  {
    "tokenizer": "standard",
    "filter": [ "stemmer" ],
    "text": "the foxes jumping quickly"
  }
  
  # [ the, fox, jump, quickli ]
  ```

* 配置

  **language**：(可选的，字符串）与语言有关的词干化算法，用于词干化标记。如果同时指定`language`参数和`name`参数，则使用`language`参数。

  **name**:[`language`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html#analysis-stemmer-tokenfilter-language-parm) 参数的别名。如果同时指定`language`参数和`name`参数，则使用`language`参数。

  

  language参数可选值（默认为 [**`english`**](https://snowballstem.org/algorithms/porter/stemmer.html)）：

  **阿拉伯**[**`arabic`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicStemmer.html)

  **亚美尼亚语**[**`armenian`**](https://snowballstem.org/algorithms/armenian/stemmer.html)

  **巴斯克**[**`basque`**](https://snowballstem.org/algorithms/basque/stemmer.html)

  **孟加拉**[**`bengali`**](https://www.tandfonline.com/doi/abs/10.1080/02564602.1993.11437284)

  **巴西葡萄牙语**[**`brazilian`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/br/BrazilianStemmer.html)

  **保加利亚语**[**`bulgarian`**](http://members.unine.ch/jacques.savoy/Papers/BUIR.pdf)

  **加泰罗尼亚语**[**`catalan`**](https://snowballstem.org/algorithms/catalan/stemmer.html)

  **捷克语**[**`czech`**](https://dl.acm.org/doi/10.1016/j.ipm.2009.06.001)

  **丹麦语**[**`danish`**](https://snowballstem.org/algorithms/danish/stemmer.html)

  **荷兰语**[**`dutch`**](https://snowballstem.org/algorithms/dutch/stemmer.html), [`dutch_kp`](https://snowballstem.org/algorithms/kraaij_pohlmann/stemmer.html)

  **英语**[**`english`**](https://snowballstem.org/algorithms/porter/stemmer.html), [`light_english`](https://ciir.cs.umass.edu/pubfiles/ir-35.pdf), [`lovins`](https://snowballstem.org/algorithms/lovins/stemmer.html), [`minimal_english`](https://www.researchgate.net/publication/220433848_How_effective_is_suffixing), [`porter2`](https://snowballstem.org/algorithms/english/stemmer.html), [`possessive_english`](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/en/EnglishPossessiveFilter.html)

  **爱沙尼亚语**[**`estonian`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/tartarus/snowball/ext/EstonianStemmer.html)

  **芬兰**[**`finnish`**](https://snowballstem.org/algorithms/finnish/stemmer.html), [`light_finnish`](http://clef.isti.cnr.it/2003/WN_web/22.pdf)

  **法语**[**`light_french`**](https://dl.acm.org/citation.cfm?id=1141523), [`french`](https://snowballstem.org/algorithms/french/stemmer.html), [`minimal_french`](https://dl.acm.org/citation.cfm?id=318984)

  **加利西亚语**[**`galician`**](http://bvg.udc.es/recursos_lingua/stemming.jsp), [`minimal_galician`](http://bvg.udc.es/recursos_lingua/stemming.jsp)（仅限复数步骤）

  **德语**[**`light_german`**](https://dl.acm.org/citation.cfm?id=1141523), [`german`](https://snowballstem.org/algorithms/german/stemmer.html), [`german2`](https://snowballstem.org/algorithms/german2/stemmer.html), [`minimal_german`](http://members.unine.ch/jacques.savoy/clef/morpho.pdf)

  **希腊语**[**`greek`**](https://sais.se/mthprize/2007/ntais2007.pdf)

  **印地语**[**`hindi`**](http://computing.open.ac.uk/Sites/EACLSouthAsia/Papers/p6-Ramanathan.pdf)

  **匈牙利**[**`hungarian`**](https://snowballstem.org/algorithms/hungarian/stemmer.html), [`light_hungarian`](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)

  **印度尼西亚**[**`indonesian`**](http://www.illc.uva.nl/Publications/ResearchReports/MoL-2003-02.text.pdf)

  **爱尔兰语**[**`irish`**](https://snowballstem.org/otherapps/oregan/)

  **意大利语**[**`light_italian`**](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf), [`italian`](https://snowballstem.org/algorithms/italian/stemmer.html)

  **库尔德语（索拉尼语）**[**`sorani`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniStemmer.html)

  **拉脱维亚语**[**`latvian`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/lv/LatvianStemmer.html)

  **立陶宛语**[**`lithuanian`**](https://svn.apache.org/viewvc/lucene/dev/branches/lucene_solr_5_3/lucene/analysis/common/src/java/org/apache/lucene/analysis/lt/stem_ISO_8859_1.sbl?view=markup)

  **挪威语（博克马尔语）**[**`norwegian`**](https://snowballstem.org/algorithms/norwegian/stemmer.html), [**`light_norwegian`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html), [`minimal_norwegian`](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)

  **挪威语（尼诺斯克）**[**`light_nynorsk`**](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html), [`minimal_nynorsk`](https://lucene.apache.org/core/8_10_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)

  **葡萄牙语**[**`light_portuguese`**](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181), [`minimal_portuguese`](http://www.inf.ufrgs.br/~buriol/papers/Orengo_CLEF07.pdf), [`portuguese`](https://snowballstem.org/algorithms/portuguese/stemmer.html), [`portuguese_rslp`](https://www.inf.ufrgs.br//~viviane/rslp/index.htm)

  **罗马尼亚语**[**`romanian`**](https://snowballstem.org/algorithms/romanian/stemmer.html)

  **俄语**[**`russian`**](https://snowballstem.org/algorithms/russian/stemmer.html), [`light_russian`](https://doc.rero.ch/lm.php?url=1000%2C43%2C4%2C20091209094227-CA%2FDolamic_Ljiljana_-_Indexing_and_Searching_Strategies_for_the_Russian_20091209.pdf)

  **西班牙语**[**`light_spanish`**](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf), [`spanish`](https://snowballstem.org/algorithms/spanish/stemmer.html)

  **瑞典**[**`swedish`**](https://snowballstem.org/algorithms/swedish/stemmer.html), [`light_swedish`](http://clef.isti.cnr.it/2003/WN_web/22.pdf)

  **土耳其**[**`turkish`**](https://snowballstem.org/algorithms/turkish/stemmer.html)

* 配置举例

  ```bash
  PUT /my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "filter": [
              "lowercase",
              "my_stemmer"
            ]
          }
        },
        "filter": {
          "my_stemmer": {
            "type": "stemmer",
            "language": "light_german"
          }
        }
      }
    }
  }
  ```

###### KStem 标记过滤器(kstem)

> 为英语提供基于[KStem](https://ciir.cs.umass.edu/pubfiles/ir-35.pdf)的词干提取。该`kstem`过滤器将[算法词干](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/stemming.html#algorithmic-stemmers)与内置 [字典](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/stemming.html#dictionary-stemmers)相结合 。
>
> kstem过滤器比其他英语词干过滤器（如porter_stem过滤器）消极。
>
> 该`kstem`滤波器相当于 [`stemmer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-stemmer-tokenfilter.html)过滤器的 [`light_english`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-stemmer-tokenfilter.html#analysis-stemmer-tokenfilter-language-parm)变种。
>
> 此过滤器使用 Lucene 的 [KStemFilter](https://lucene.apache.org/core/8_9_0/analyzers-common/org/apache/lucene/analysis/en/KStemFilter.html)。

* 举例

  ```bash
  GET /_analyze
  {
    "tokenizer": "standard",
    "filter": [ "kstem" ],
    "text": "the foxes jumping quickly"
  }
  
  # [ the, fox, jump, quick ]
  ```

* 自定义

  > Note:为了正常工作，`kstem`过滤器需要小写标记。为确保标记为小写，请在分析器配置中的`kstem`过滤器之前添加[`lowercase`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-lowercase-tokenfilter.html)过滤器。

  ```bash
  PUT /my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "whitespace",
            "filter": [
              "lowercase",
              "kstem"
            ]
          }
        }
      }
    }
  }
  ```

###### Porter stem标记过滤器(porter_stem)

> 基于[Porter stem算法](https://snowballstem.org/algorithms/porter/stemmer.html),为英语提供[算法词干](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/stemming.html#algorithmic-stemmers)。
>
> 与其他英语词干过滤器（例如[`kstem`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-kstem-tokenfilter.html)过滤器）相比，此过滤器倾向于更积极地进行词干化。
>
> 该`porter_stem`滤波器相当于 [`stemmer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-stemmer-tokenfilter.html)过滤器的 [`english`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-stemmer-tokenfilter.html#analysis-stemmer-tokenfilter-language-parm)变种。
>
> 该`porter_stem`过滤器使用Lucene的 [PorterStemFilter](https://lucene.apache.org/core/8_9_0/analyzers-common/org/apache/lucene/analysis/en/PorterStemFilter.html)。

* 举例

  ```bash
  GET /_analyze
  {
    "tokenizer": "standard",
    "filter": [ "porter_stem" ],
    "text": "the foxes jumping quickly"
  }
  # [ the, fox, jump, quickli ]
  ```

* 自定义

  > Note:为了正常工作，`porter_stem`过滤器需要小写标记。为确保标记为小写，请在分析器配置中的`porter_stem`过滤器之前添加[`lowercase`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-lowercase-tokenfilter.html)过滤器。

  ```bash
  PUT /my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "whitespace",
            "filter": [
              "lowercase",
              "porter_stem"
            ]
          }
        }
      }
    }
  }
  ```

###### Snowball 标记过滤器(snowball)

> 使用 Snowball 生成的词干分析器来提取词干的过滤器。 `language`参数可用值：`Arabic`，`Armenian`，`Basque`，`Catalan`，`Danish`，`Dutch`，`English`， `Estonian`，`Finnish`，`French`，`German`，`German2`，`Hungarian`，`Italian`，`Irish`，`Kp`， `Lithuanian`，`Lovins`，`Norwegian`，`Porter`，`Portuguese`，`Romanian`， `Russian`，`Spanish`，`Swedish`，`Turkish`。

* 举例：

  ```bash
  PUT /my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "filter": [ "lowercase", "my_snow" ]
          }
        },
        "filter": {
          "my_snow": {
            "type": "snowball",
            "language": "Lovins"
          }
        }
      }
    }
  }
  ```

  

##### 字典词干过滤器

> 词典词干过滤器在提供的词典中查找词，用词典中的词干替换未词干的词变体。

###### Hunspell 标记过滤器(hunspell)

> 根据提供的 [Hunspell 字典](https://en.wikipedia.org/wiki/Hunspell)提供[字典词干](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/stemming.html#dictionary-stemmers)。该 过滤器需要 [配置](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-hunspell-tokenfilter.html#analysis-hunspell-tokenfilter-dictionary-config)一个或多个语言特定的hunspell字典。`hunspell`
>
> 此过滤器使用 Lucene 的 [HunspellStemFilter](https://lucene.apache.org/core/8_9_0/analyzers-common/org/apache/lucene/analysis/hunspell/HunspellStemFilter.html)。

##### 控制词干过滤器

> 有时词干提取可以产生拼写相似但概念上不相关的共享词根。例如，词干分析器可以将`skies`和 都减少`skiing`到相同的词根：`ski`。

###### Stemmer 覆盖标记过滤器

###### 关键字标记过滤器(keyword_marker)

###### [条件token filter(condition) ](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-condition-tokenfilter.html)

> 将一组标记过滤器应用于与提供的谓词脚本中的条件匹配的标记。
>
> This filter uses Lucene’s [ConditionalTokenFilter](https://lucene.apache.org/core/8_2_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ConditionalTokenFilter.html).

* 举例

  ``` bash
  GET /_analyze
  {
    "tokenizer": "standard",
    "filter": [
      {
        "type": "condition",
        "filter": [ "lowercase" ],
        "script": {
          "source": "token.getTerm().length() < 5"
        }
      }
    ],
    "text": "THE QUICK BROWN FOX"
  }
  
  [ the, QUICK, BROWN, fox ]
  ```

* 配置参数

  **`filter`**

  (Required, array of token filters) Array of token filters. If a token matches the predicate script in the `script` parameter, these filters are applied to the token in the order provided.

  These filters can include custom token filters defined in the index mapping.

  **`script`**

  (Required, [script object](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/modules-scripting-using.html)) Predicate script used to apply token filters. If a token matches this script, the filters in the `filter` parameter are applied to the token.

  For valid parameters, see [Script parameters](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/modules-scripting-using.html#_script_parameters). Only inline scripts are supported. Painless scripts are executed in the [analysis predicate context](https://www.elastic.co/guide/en/elasticsearch/painless/7.4/painless-analysis-predicate-context.html) and require a `token` property.

* 配置参数举例

  ``` bash
  PUT /palindrome_list
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "whitespace_reverse_first_token": {
            "tokenizer": "whitespace",
            "filter": [ "reverse_first_token" ]
          }
        },
        "filter": {
          "reverse_first_token": {
            "type": "condition",
            "filter": [ "reverse" ],
            "script": {
              "source": "token.getPosition() === 0"
            }
          }
        }
      }
    }
  }
  ```

  

#### 与标记相关的 Token Filters

##### 图标记过滤器

（可以准确记录`positionLength`多位置标记。）



##### 无效图标过滤器

（可以添加跨越多个位置，但只能记录一个默认标记`positionLength`的`1`）



#### 其他 Token Filters

##### [撇号标记过滤器(apostrophe)](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-apostrophe-tokenfilter.html#analysis-apostrophe-tokenfilter)

> 去除撇号后的所有字符，包括撇号本身。 
>
> 此过滤器包含在 Elasticsearch 的内置土耳其语语言分析器中。它使用 Lucene 的 ApostropheFilter，它是为土耳其语构建的。

* 举例

```json
GET /_analyze
{
  "tokenizer" : "standard",
  "filter" : ["apostrophe"],
  "text" : "Istanbul'a veya Istanbul'dan"
}

[ Istanbul, veya, Istanbul ]
```

##### [ASCII 折叠标记过滤器(asciifolding)](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-asciifolding-tokenfilter.html#analysis-asciifolding-tokenfilter)

> 将不在前 127 个 ASCII 字符中的字母、数字和符号字符转换为它们的 ASCII 等效字符（如果存在）。例如，过滤器将 à 更改为 a。 这个过滤器使用 Lucene 的 ASCIIFoldingFilter。

* 举例

```bash
GET /_analyze
{
  "tokenizer" : "standard",
  "filter" : ["asciifolding"],
  "text" : "açaí à la carte"
}

[ acai, a, la, carte ]
```

* 可配置参数

  **`preserve_original`**：(Optional, boolean) If `true`, emit both original tokens and folded tokens. Defaults to `false`.

* 配置举例

  ```json
  PUT /asciifold_example
  {
      "settings" : {
          "analysis" : {
              "analyzer" : {
                  "standard_asciifolding" : {
                      "tokenizer" : "standard",
                      "filter" : ["my_ascii_folding"]
                  }
              },
              "filter" : {
                  "my_ascii_folding" : {
                      "type" : "asciifolding",
                      "preserve_original" : true
                  }
              }
          }
      }
  }
  ```

##### [CJK bigram 标记过滤器(cjk_bigram)](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-cjk-bigram-tokenfilter.html)

> 用 CJK（中文、日文和韩文）标记形成tokens。 此过滤器包含在 Elasticsearch 的内置 CJK 语言分析器中。它使用 Lucene 的 CJKBigramFilter。

* 举例

```json
GET /_analyze
{
  "tokenizer" : "standard",
  "filter" : ["cjk_bigram"],
  "text" : "東京都は、日本の首都であり"
}

[ 東京, 京都, 都は, 日本, 本の, の首, 首都, 都で, であ, あり ]
```

* 可配置参数

  **`ignored_scripts`**：(Optional, array of character scripts) Array of character scripts for which to disable bigrams. Possible values:`han`，`hangul`，`hiragana`，`katakana`All non-CJK input is passed through unmodified.

  **`output_unigrams` **(Optional, boolean) If `true`, emit tokens in both bigram and [unigram](https://en.wikipedia.org/wiki/N-gram) form. If `false`, a CJK character is output in unigram form when it has no adjacent characters. Defaults to `false`.

* 配置举例

  ```json
  PUT /cjk_bigram_example
  {
      "settings" : {
          "analysis" : {
              "analyzer" : {
                  "han_bigrams" : {
                      "tokenizer" : "standard",
                      "filter" : ["han_bigrams_filter"]
                  }
              },
              "filter" : {
                  "han_bigrams_filter" : {
                      "type" : "cjk_bigram",
                      "ignored_scripts": [
                          "hangul",
                          "hiragana",
                          "katakana"
                      ],
                      "output_unigrams" : true
                  }
              }
          }
      }
  }
  ```

##### [CJK 宽度标记过滤器(cjk_width)](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-cjk-width-tokenfilter.html)

> 将 CJK（中文、日文和韩文）字符的宽度差异标准化如下：
>
> - 将全角 ASCII 字符变体折叠成等效的基本Latin字符
> - 将半角片假名字符变体折叠成等效的Kana字符
>
> 此过滤器包含在 Elasticsearch 的内置[CJK 语言分析器中](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-lang-analyzer.html#cjk-analyzer)。它使用 Lucene 的 [CJKWidthFilter](https://lucene.apache.org/core/8_2_0/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html)。
>
> NOTE:这个标记过滤器可以被看作是 NFKC/NFKD Unicode 规范化的一个子集。请参阅 [`analysis-icu`插件](https://www.elastic.co/guide/en/elasticsearch/plugins/7.4/analysis-icu-normalization-charfilter.html)以获取完整的规范化支持。

* 举例

```console
GET /_analyze
{
  "tokenizer" : "standard",
  "filter" : ["cjk_width"],
  "text" : "ｼｰｻｲﾄﾞﾗｲﾅｰ"
}

シーサイドライナー
```

##### [经典标记过滤器(classic)](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-classic-tokenfilter.html)

> 对经典分词器生成的术语执行可选的后处理。 此过滤器从单词末尾删除英语所有格 ('s) 并从首字母缩略词中删除点。它使用 Lucene 的 ClassicFilter。

* 举例

```console
GET /_analyze
{
  "tokenizer" : "classic",
  "filter" : ["classic"],
  "text" : "The 2 Q.U.I.C.K. Brown-Foxes jumped over the lazy dog's bone."
}

[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog, bone ]
```

##### [常见的grams标记过滤器(common_grams)](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-common-grams-tokenfilter.html)

> 为一组指定的常用词生成二元组。
>
> 例如，您可以指定 is 和 the 作为常用词。然后，此过滤器将标记 [the, quick, fox, is, brown] 转换为[the, the_quick, quick, fox, fox_is, is, is_brown, brown]。
>
> 当您不想完全忽略常用词时，您可以使用 common_grams 过滤器代替 [stop token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-stop-tokenfilter.html) 过滤器。
>
> This filter uses Lucene’s [CommonGramsFilter](https://lucene.apache.org/core/8_2_0/analyzers-common/org/apache/lucene/analysis/commongrams/CommonGramsFilter.html).

* 举例

``` bash
GET /_analyze
{
  "tokenizer" : "whitespace",
  "filter" : [
    {
      "type": "common_grams",
      "common_words": ["is", "the"]
    }
  ],
  "text" : "the quick fox is brown"
}

[ the, the_quick, quick, fox, fox_is, is, is_brown, brown ]
```

* 可配置参数
  * **`common_words`**

    (Required*, array of strings) A list of tokens. 过滤器为这些标记生成二元组。 需要此参数或 common_words_path 参数。

  * **`common_words_path`**

    (Required*, string) Path to a file containing a list of tokens. 过滤器为这些标记生成二元组。

    此路径必须是绝对的或相对于config位置的。该文件必须采用 UTF-8 编码。文件中的每个标记必须由换行符分隔。

    需要此参数或 common_words 参数。

  * **`ignore_case`**

    (Optional, boolean) If `true`, 常用词匹配的匹配不区分大小写. Defaults to `false`.

  * **`query_mode`**

    (Optional, boolean) If `true`, 过滤器从输出中排除以下标记:

    	- 常用词的一元组
    	- 术语后跟常用词的一元组

    Defaults to `false`.

    我们建议为[search analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/search-analyzer.html)启用此参数。

    For example, you can enable this parameter and specify `is` and `the` as common words. This filter converts the tokens `[the, quick, fox, is, brown]` to `[the_quick, quick, fox_is, is_brown,]`.

* 配置举例

  ``` bash
  PUT /common_grams_example
  {
      "settings": {
          "analysis": {
              "analyzer": {
                "index_grams": {
                    "tokenizer": "whitespace",
                    "filter": ["common_grams_query"]
                }
              },
              "filter": {
                "common_grams_query": {
                    "type": "common_grams",
                    "common_words": ["a", "is", "the"],
                    "ignore_case": true,
                    "query_mode": true
                }
              }
          }
      }
  }
  ```



## ES 中分词器的设置

```bash
{
  "settings": {
    "analysis": {
      "char_filter": { 
        "quotes": {
          "type": "mapping",
          "mappings": [ 
            "\\u0091=>\\u0027",
            "\\u0092=>\\u0027",
            "\\u2018=>\\u0027",
            "\\u2019=>\\u0027",
            "\\u201B=>\\u0027"
          ]
        }，
      "filter": {
        "nfkc_normalizer": { 
          "type": "icu_normalizer",
          "name": "nfkc"
        },
        "english_stop": {
          "type":       "stop",
          "stopwords":  "_english_"
        },
        "english_keywords": {
          "type":       "keyword_marker", 
          "keywords":   []
        },
        "english_stemmer": {
          "type":       "stemmer",
          "language":   "english"  # light_english 非激进的词干提取器
        },
        "english_possessive_stemmer": {
          "type":       "stemmer",
          "language":   "possessive_english" 
        }，
        # hunspell 语汇单元过滤器
        "en_US": {
          "type":     "hunspell",
          "language": "en_US" 
        }，
        # 阻止词干提取
        "no_stem": {
          "type": "keyword_marker",
          "keywords": [ "skies" ] 
        },
        # 自定义词干提取
        "custom_stem": {
          "type": "stemmer_override",
          "rules": [ 
            "skies=>sky",
            "mice=>mouse",
            "feet=>foot"
          ]
        },
        # 防止提取和未提取词干形式相同的词项中的无意义重复
        "unique_stem": {
          "type": "unique",
          "only_on_same_position": true 
        }

      },
      "analyzer": {
        "quotes_analyzer": {
        	"char_filter": [ "quotes" ] ,
          "tokenizer": "icu_tokenizer",
          "filter":  [ "lowercase","nfkc_normalizer" ]
          # icu_normalizer 默认是 nfkc_cf 模式(lowercase 语汇单元过滤器(token filters)).
        }
      }
    }
  }
}
```

# ES - Painless 脚本语言

## 概述

Painless 是 Elasticsearch 中一种用于编写脚本的语言，它可以在查询、聚合、索引等各个环节中被使用。以下是一些使用 Painless 脚本的场景：

1. 查询：在 Elasticsearch 中，可以使用 Painless 脚本来编写自定义查询语句，以便在查询时进行更加精确的计算和筛选。
2. 聚合：在聚合查询中，Painless 脚本可以用来进行计算、转换、过滤等操作，从而实现更加灵活的聚合分析。
3. 映射：在索引文档时，Painless 脚本可以用来进行文档的字段映射，以及对字段进行计算、转换等操作。
4. 处理器：在使用 Elasticsearch 中的处理器时，可以使用 Painless 脚本来编写自定义的处理逻辑，以便对文档进行处理和转换。

总之，Painless 脚本可以被用于 Elasticsearch 的各个环节中，它可以让用户编写更加灵活和高效的代码，从而实现更加强大的功能。

## 查询场景下的使用方法

在 Elasticsearch 源码中，Painless 脚本主要是在查询场景中被使用的。具体来说，在查询过程中，Elasticsearch 使用了 QueryBuilder 和 SearchSourceBuilder 这两个类来构建查询请求，并且可以通过这两个类的方法来指定 Painless 脚本的使用。

其中，QueryBuilder 类的源代码位于 `core/src/main/java/org/elasticsearch/index/query/QueryBuilder.java` 文件中，而 SearchSourceBuilder 类的源代码位于 `core/src/main/java/org/elasticsearch/search/builder/SearchSourceBuilder.java` 文件中。在这两个类的源代码中，可以看到有许多方法可以使用 Painless 脚本，比如：

- `ScriptQueryBuilder.script(Script script)` 方法：可以用 Painless 脚本作为查询条件。
- `AggregationBuilder.script(Script script)` 方法：可以使用 Painless 脚本作为聚合查询的计算逻辑。
- `FieldSortBuilder.setScript(Script script)` 方法：可以使用 Painless 脚本进行排序。
- `ScriptField.add(Script script)` 方法：可以使用 Painless 脚本计算新的字段值。
- `ScriptTransform.Builder.setScript(Script script)` 方法：可以使用 Painless 脚本进行文档的转换。

当然，还有其他的类和方法也可以使用 Painless 脚本，在 Elasticsearch 的源代码中可以通过搜索 "painless" 来查找相关的代码片段。

## Debug调试ES Painless脚本

以下是一些常见的涉及查询解析器中 painless 脚本的源码文件和方法，您可以根据需要在相应的类文件和方法上设置断点：

1. ScriptQueryParser 类的 parseInnerQueryBuilder 方法：这个方法用于解析查询请求中的 painless 脚本查询语句，并将其转换成 Elasticsearch 内部的查询结构。
2. PainlessScriptEngineService 类的 compile 方法：这个方法用于编译 painless 脚本，并将其转换成 Elasticsearch 内部的执行结构。
3. SearchScript 类的 compile 方法：这个方法用于编译 painless 脚本，并将其转换成 Elasticsearch 内部的执行结构。
4. ScriptedMetricAggregationBuilder 类的 buildScript 方法：这个方法用于构建 painless 脚本聚合操作，并将其转换成 Elasticsearch 内部的聚合结构。
5. AbstractSearchScript 类的 runAsFloat/Double/Long/Boolean/String 方法：这个类是一个抽象类，用于实现 painless 脚本的执行逻辑。其中的 runAsFloat/Double/Long/Boolean/String 方法分别用于将 painless 脚本的执行结果转换成不同类型的值。
6. PainlessKeywordMarkerFilter 类和 PainlessAnalysisPlugin 类：这两个类分别用于实现 painless 脚本中关键词的过滤和分词等分析操作。
7. PainlessQueryBuilder 类的 doToQuery 方法：这个方法用于将 painless 脚本转换成 Elasticsearch 内部的查询对象，以便进行查询操作。
8. PainlessScript 类的 compile 方法和 runAs* 方法：这个类用于执行 painless 脚本，并将其结果返回给 Elasticsearch。
9. PainlessContext 类和 PainlessInterpreter 类：这两个类分别用于管理 painless 脚本的上下文和执行 painless 脚本的解释器。
10. PainlessScriptEngine 类和 PainlessScriptEngineService 类：这两个类分别用于注册和管理 Elasticsearch 中的 painless 脚本引擎。
11. PainlessContextFactory 类和 PainlessScriptEngineService.ScriptContext 类：这两个类分别用于创建 painless 脚本执行的上下文和保存 painless 脚本执行的上下文信息。
12. ScriptFieldsVisitor 类的 process 方法：这个方法用于解析 painless 脚本中的 script_fields 字段，并将其结果添加到搜索结果中。
13. SearchSourceBuilder 类的 scriptFields 方法：这个方法用于在搜索请求中指定 painless 脚本，以便在搜索结果中生成 script_fields 字段。
14. ScoreScript 类和 AbstractFloatSearchScript 类：这两个类分别用于实现 painless 脚本中的得分脚本和搜索脚本。
15. PainlessFieldComparatorSource 类和 PainlessFieldComparatorSourceParser 类：这两个类分别用于实现 painless 脚本中的字段排序功能和解析排序脚本。
16. PainlessFieldDataType 类和 PainlessFieldDataTypeLookup 类：这两个类分别用于管理 painless 脚本中的字段数据类型，并将其转换为 Elasticsearch 内部的数据类型。
17. PainlessAggregationBuilder 类和 PainlessAggregator 类：这两个类分别用于实现 painless 脚本中的聚合操作，并将其转换为 Elasticsearch 内部的聚合操作。
18. PainlessScriptQuery 类和 PainlessScriptQueryBuilder 类：这两个类分别用于实现 painless 脚本中的查询操作，并将其转换为 Elasticsearch 内部的查询操作。
19. PainlessSortBuilder 类和 PainlessSortParser 类：这两个类分别用于实现 painless 脚本中的排序操作，并将其转换为 Elasticsearch 内部的排序操作。
20. PainlessQueryBuilder 类和 PainlessQueryBuilderParser 类：这两个类分别用于实现 painless 脚本中的查询操作，并将其转换为 Elasticsearch 内部的查询操作。
21. SearchScript 类和 AbstractSearchScript 类：这两个类分别用于实现 painless 脚本中的搜索脚本，并将其转换为 Elasticsearch 内部的搜索脚本。

> 需要注意的是，在涉及到 painless 脚本的源码调试时，建议使用 Elasticsearch 的内置调试工具，例如在 `elasticsearch.yml` 文件中设置 `logger.org.elasticsearch.script.painless=TRACE`，可以输出 painless 脚本的编译和执行过程中的详细日志信息，从而更好地帮助调试。
>
> 设置调试器：在您的 IDE 中设置断点，或者使用命令行启动 Elasticsearch 并设置调试器。例如，在启动 Elasticsearch 前加上如下的 JVM 参数：
>
> ```
> -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005
> ```
>
> 这将会在端口号 5005 上启动一个调试器，并在 Elasticsearch 启动时暂停，等待调试器连接。
