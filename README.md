# elastic-phonatic-comparsion
comparison between elasticsearch phonatic algorithms

TLDR;

 [click me](#results) to see the comparison.

### start elasticsearch 5 and kibana
```bash
  docker-compose up -d
```

### elasticsearch plugings needed `docker run -it elasticsearch bash` then:
```bash
  # provide char_filter icu_normalizer and filter icu_folding
  bin/elasticsearch-plugin install analysis-icu
  # provide phonatic algorithms
  bin/elasticsearch-plugin install analysis-phonetic

  #### restart elasticsearch => docker-compose restart
```

#### Test:
 * load kibana console with test date by clicking [here](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://raw.githubusercontent.com/waleedsamy/elastic-phonatic-comparsion/master/kibana.json)

#### Notes:
 * if you are using elasticsearch older than 5, you should check changes happened to [suggester](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/breaking_50_suggester.html)
 * autocomplete filed does not need `payload` attribute any more, `_source` is used as a replacement
 * autocomplete filed does not need `output` attribute, you could use `_source` to simulate it.
 * consider using `index/_search` instead of the deprecated `index/_suggest`
 * [Fuzzy Matching to Search by Sound](http://www.informit.com/articles/article.aspx?p=1848528)

#### Issues:
 * suggester context do `OR`ed thing not `AND` [21291](https://github.com/elastic/elasticsearch/issues/21291)

#### How to custom analyzers
 * char_filters: Character filters are used to preprocess the stream of characters before it is passed to the tokenizer.
 * tokenizer: A tokenizer receives a stream of characters, breaks it up into individual tokens (usually individual words), and outputs a stream of tokens.
 * filter: Token filters accept a stream of tokens from a tokenizer and can modify tokens (eg lowercasing), delete tokens (eg remove stopwords) or add tokens (eg synonyms).

   ```
      {
         "analyzer":{
            "my_custom_analyzer":{
               "type":"custom",
               "tokenizer":"standard",
               "char_filter":[
                  "html_strip"
               ],
               "filter":[
                  "lowercase",
                  "asciifolding"
               ]
            }
         }
      }
   ```

#### results
 * ✓ means match
 * ☓ means doesn't match

| cities/algorithms | cologne | soundex | metaphone | doublemetaphone|
| --- |--- | --- | --- | --- |
| Kairo vs Cairo            | ✓       | ☓        | ✓         | ✓              |
| Kairo vs Qairo            | ✓       | ☓        | ✓         | ✓              |
| Koln vs Köln              | ✓       | ✓        | ✓         | ✓              |
| Paris vs Biarritz         | ✓       | ☓        | ☓         | ☓              |
| Berlin vs Paralimni       | ✓       | ☓        | ☓         | ☓              |

#### suggestion
 * `Metaphone` algorithm produce the most expected result for my test
 * Don't depend only on the phonatic algorithm, you should have a `Weight` for every suggestion
