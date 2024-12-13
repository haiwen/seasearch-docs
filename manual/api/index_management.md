## Index Management
In SeaSearch, users can create any number of indexes. An index is a collection of documents that can be searched, and a document can contain multiple searchable fields. Users specify the fields contained in the index via mappings and can customize the analyzers available to the index through settings. Each field can specify either a built-in or custom analyzer. The analyzer is used to split the content of a field into searchable tokens.

### Create Index
To create a SeaSearch index, you can configure the mappings and settings at the same time. For more details about mappings and settings, refer to the following sections.

ElasticSearch API: [Create Index](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)

### Configure Mappings
Mappings define the types and attributes of fields in a document. Users can configure the mapping via the API.

SeaSearch supports the following field types:

- text
- keyword
- numeric
- bool
- date
- vector

Other types, such as flattened, object, nested, etc., are not supported, and mappings do not support modifying existing fields (new fields can be added).

ElasticSearch Mappings API: [Put Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html)

ElasticSearch Mappings Explanation: [Mapping Types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

### Configure Settings
Index settings control the properties of the index. The most commonly used property is `analysis`, which allows you to customize the analyzers for the index. The analyzers defined here can be used by fields in the mappings.

ElasticSearch Settings API: [Update Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html)

ElasticSearch related explanation:
- [Analyzer Concepts](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-concepts.html)
- [Specifying Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html)

### Analyzer Support
Analyzers can be configured as default when creating an index, or they can be set for specific fields. (See the previous section for related concepts from the ES documentation.)

SeaSearch supports the following analyzers, which can be found here: [ZincSearch Documentation](https://zincsearch-docs.zinc.dev/api/index/analyze/). The concepts such as tokenization and token filters are consistent with ES and support most of the commonly used analyzers and tokenizers in ES.

### Chinese Analyzer
To enable the Chinese analyzer in the system, set the environment variable `ZINC_PLUGIN_GSE_ENABLE=true`.

If you need more comprehensive support for Chinese word dictionaries, set `ZINC_PLUGIN_GSE_DICT_EMBED = BIG`.

`GSE` is a standard analyzer, so you can directly assign the Chinese analyzer to fields in the mappings:
```
PUT /es/my-index/_mappings
{
  "properties": {
    "content": { 
        "type": "text",
        "analyzer": "gse_standard"
      }
  }
}
```
If users have custom tokenization habits, they can specify their dictionary files by setting the environment variable `ZINC_PLUGIN_GSE_DICT_PATH=${DICT_PATH}`, where `DICT_PATH` is the actual path to the dictionary files. The `user.txt` file contains the dictionary, and the `stop.txt` file contains stop words. Each line contains a single word.

GSE will load the dictionary and stop words from this path and use the user-defined dictionary to segment Chinese sentences.
