

# Advanced scripts using script engines

ScriptEngine是实现脚本语言的后端。它还可以用于编写需要使用高级脚本内部结构的脚本。例如，一个脚本希望在评分时使用术语频率。

[插件文档]([documentation](https://www.elastic.co/guide/en/elasticsearch/plugins/7.11/plugin-authors.html) )中有更多关于如何编写插件的信息，这样Elasticsearch就可以正确地加载插件。要注册ScriptEngine，你的插件应该实现ScriptPlugin接口并覆盖getScriptEngine(Settings Settings)方法。

下面是一个自定义ScriptEngine的示例，它使用了名为expert_scripts。它实现了一个名为pure_df的单一脚本，可以用作搜索脚本，以覆盖每个文档的得分作为提供的术语的文档频率。



```
public class ExpertScriptPlugin extends Plugin implements ScriptPlugin {

    @Override
    public ScriptEngine getScriptEngine(Settings settings, Collection<ScriptContext<?>> contexts) {
        return new MyExpertScriptEngine();
    }

    /**
     * An example {@link ScriptEngine} that uses Lucene segment details to
     * implement pure document frequency scoring.
     */
    // tag::expert_engine
    private static class MyExpertScriptEngine implements ScriptEngine {
        @Override
        public String getType() {
            return "expert_scripts";
        }

        @Override
        public <T> T compile(String scriptName, String scriptSource, ScriptContext<T> context, Map<String, String> params) {
            if (context.equals(ScoreScript.CONTEXT) == false) {
                throw new IllegalArgumentException(getType() + " scripts cannot be used for context [" + context.name + "]");
            }
            // we use the script "source" as the script identifier
            if ("pure_df".equals(scriptSource)) {
                ScoreScript.Factory factory = new PureDfFactory();
                return context.factoryClazz.cast(factory);
            }
            throw new IllegalArgumentException("Unknown script name " + scriptSource);
        }

        @Override
        public void close() {
            // optionally close resources
        }

        @Override
        public Set<ScriptContext<?>> getSupportedContexts() {
            return Set.of(ScoreScript.CONTEXT);
        }

        private static class PureDfFactory implements ScoreScript.Factory, ScriptFactory {
            @Override
            public boolean isResultDeterministic() {
                // PureDfLeafFactory only uses deterministic APIs, this
                // implies the results are cacheable.
                return true;
            }

            @Override
            public LeafFactory newFactory(Map<String, Object> params, SearchLookup lookup) {
                return new PureDfLeafFactory(params, lookup);
            }
        }

        private static class PureDfLeafFactory implements LeafFactory {
            private final Map<String, Object> params;
            private final SearchLookup lookup;
            private final String field;
            private final String term;

            private PureDfLeafFactory(Map<String, Object> params, SearchLookup lookup) {
                if (params.containsKey("field") == false) {
                    throw new IllegalArgumentException("Missing parameter [field]");
                }
                if (params.containsKey("term") == false) {
                    throw new IllegalArgumentException("Missing parameter [term]");
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
            public ScoreScript newInstance(LeafReaderContext context) throws IOException {
                PostingsEnum postings = context.reader().postings(new Term(field, term));
                if (postings == null) {
                    /*
                     * the field and/or term don't exist in this segment,
                     * so always return 0
                     */
                    return new ScoreScript(params, lookup, context) {
                        @Override
                        public double execute(ExplanationHolder explanation) {
                            return 0.0d;
                        }
                    };
                }
                return new ScoreScript(params, lookup, context) {
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
    // end::expert_engine
}
```



您可以通过将其lang指定为expert_scripts，并将脚本的名称指定为脚本源来执行脚本

```console
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