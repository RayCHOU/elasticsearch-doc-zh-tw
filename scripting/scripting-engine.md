# [Advanced scripts using script engines](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-engine.html)

A ScriptEngine is a backend for implementing a scripting language. 
It may also be used to write scripts that need to use advanced internals of scripting. 
For example, a script that wants to use term frequencies while scoring.

ScriptEngine 是實現腳本語言的後端。  
它還可用於編寫需要使用高級腳本內部機制的腳本。  
例如，想要在評分時使用 term frequencies 的腳本。

The plugin documentation has more information on how to write a plugin so that Elasticsearch will properly load it. 
To register the `ScriptEngine`, your plugin should implement the `ScriptPlugin` interface and override the `getScriptEngine(Settings settings)` method.

Plugin [documentatin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.5/plugin-authors.html) 包含有關如何編寫 plugin 插件以便 Elasticsearch 正確加載它的更多信息。  
要註冊 `ScriptEngine`，您的 plugin 應該實現 `ScriptPlugin` interface 並覆蓋 `getScriptEngine(Settings settings)` 方法。

The following is an example of a custom `ScriptEngine` which uses the language name `expert_scripts`. 
It implements a single script called `pure_df` which may be used as a search script to override each document’s score as the document frequency of a provided term.

以下是使用語言名稱 `expert_scripts` 的自定義 `ScriptEngine` 示例。  
它實現了一個名為 `pure_df` 的腳本，它可以用作搜索腳本來覆蓋每個文檔的分數作為提供的 term 的 document frequency。

```java
private static class MyExpertScriptEngine implements ScriptEngine {
    @Override
    public String getType() {
        return "expert_scripts";
    }

    @Override
    public <T> T compile(
        String scriptName,
        String scriptSource,
        ScriptContext<T> context,
        Map<String, String> params
    ) {
        if (context.equals(ScoreScript.CONTEXT) == false) {
            throw new IllegalArgumentException(getType()
                    + " scripts cannot be used for context ["
                    + context.name + "]");
        }
        // we use the script "source" as the script identifier
        if ("pure_df".equals(scriptSource)) {
            ScoreScript.Factory factory = new PureDfFactory();
            return context.factoryClazz.cast(factory);
        }
        throw new IllegalArgumentException("Unknown script name "
                + scriptSource);
    }

    @Override
    public void close() {
        // optionally close resources
    }

    @Override
    public Set<ScriptContext<?>> getSupportedContexts() {
        return Set.of(ScoreScript.CONTEXT);
    }

    private static class PureDfFactory implements ScoreScript.Factory,
                                                  ScriptFactory {
        @Override
        public boolean isResultDeterministic() {
            // PureDfLeafFactory only uses deterministic APIs, this
            // implies the results are cacheable.
            return true;
        }

        @Override
        public LeafFactory newFactory(
            Map<String, Object> params,
            SearchLookup lookup
        ) {
            return new PureDfLeafFactory(params, lookup);
        }
    }

    private static class PureDfLeafFactory implements LeafFactory {
        private final Map<String, Object> params;
        private final SearchLookup lookup;
        private final String field;
        private final String term;

        private PureDfLeafFactory(
                    Map<String, Object> params, SearchLookup lookup) {
            if (params.containsKey("field") == false) {
                throw new IllegalArgumentException(
                        "Missing parameter [field]");
            }
            if (params.containsKey("term") == false) {
                throw new IllegalArgumentException(
                        "Missing parameter [term]");
            }
            this.params = params;
            this.lookup = lookup;
            field = params.get("field").toString();
            term = params.get("term").toString();
        }

        @Override
        public boolean needs_score() {
            return false;  // Return true if the script needs the score
        }

        @Override
        public ScoreScript newInstance(DocReader docReader)
                throws IOException {
            DocValuesDocReader dvReader = DocValuesDocReader) docReader);             PostingsEnum postings = dvReader.getLeafReaderContext()                     .reader().postings(new Term(field, term;
            if (postings == null) {
                /*
                 * the field and/or term don't exist in this segment,
                 * so always return 0
                 */
                return new ScoreScript(params, lookup, docReader) {
                    @Override
                    public double execute(
                        ExplanationHolder explanation
                    ) {
                        return 0.0d;
                    }
                };
            }
            return new ScoreScript(params, lookup, docReader) {
                int currentDocid = -1;
                @Override
                public void setDocument(int docid) {
                    /*
                     * advance has undefined behavior calling with
                     * a docid <= its current docid
                     */
                    if (postings.docID() < docid) {
                        try {
                            postings.advance(docid);
                        } catch (IOException e) {
                            throw new UncheckedIOException(e);
                        }
                    }
                    currentDocid = docid;
                }
                @Override
                public double execute(ExplanationHolder explanation) {
                    if (postings.docID() != currentDocid) {
                        /*
                         * advance moved past the current doc, so this
                         * doc has no occurrences of the term
                         */
                        return 0.0d;
                    }
                    try {
                        return postings.freq();
                    } catch (IOException e) {
                        throw new UncheckedIOException(e);
                    }
                }
            };
        }
    }
}
```

You can execute the script by specifying its lang as expert_scripts, and the name of the script as the script source:

您可以通過將其 `lang` 指定為 `expert_scripts` 並將腳本名稱指定為腳本源來執行腳本：

```http
POST /_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "body": "foo"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": {
                "source": "pure_df",
                "lang" : "expert_scripts",
                "params": {
                    "field": "body",
                    "term": "foo"
                }
            }
          }
        }
      ]
    }
  }
}
```
