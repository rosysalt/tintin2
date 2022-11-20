---
{"dg-publish":true,"permalink":"/4-projects/41-learning-scala/scala2plantuml-generate-uml-diagrams/"}
---

- Action:: #to_publish  

# scala2plantuml - generate UML diagrams for Scala

Links: scal
1. scala2plantuml: https://index.scala-lang.org/bottech/scala2plantuml
2. sbt-assembly: https://github.com/sbt/sbt-assembly
3. sbtkit Packaging.scala: https://github.com/soundcloud/sbtkit/blob/1b198b9fbe05338f08aa102422220da82f64234d/src/main/scala/com/soundcloud/sbtkit/modules/Packaging.scala
4. SemanticDB guide: https://olafurpg.github.io/scalameta/docs/semanticdb/guide.html
5. SemanticDB specification: https://scalameta.org/docs/semanticdb/specification.html#symbol-1

## Setup

I add this to `~/.sbt/1.0/plugins/scala2plantuml.sbt` so that this plugin is available for all projects so I can always generate diagrams when I need to.
```
addSbtPlugin("nz.co.bottech" % "sbt-scala2plantuml" % "0.3.0")
```

If we only need to generate via sbt console, then we need to only enable semanticDB
```
semanticdbEnabled := true
```

If we only need to generate diagrams with the jar package, then we need to add these 2 lines in settings. It's not my prefered approach as generating a jar file takes a long time. However, it can be useful to include the semanticdb in the jar, so everyone can generate the diagrams with the CLI.
```
semanticdbEnabled := true
semanticdbIncludeInJar := true
```

## Generate diagrams

🚧 We need to use **slash** and not ~~dot~~ for the symbol. 🚧

✅ Correct command 
```
scala2plantuml --dir 'target/scala-2.13/classes' "com/soundcloud/kotti/Api#" -v 
```

❌ Incorrect command
```
scala2plantuml --dir 'target/scala-2.13/classes' "com.soundcloud.kotti.Api#" -v

WARN  Missing symbol: com/soundcloud/kotti/Api#
```

### For kotti

It's a project without sub-project, hence we do not need to provide `--project` flag in the CLI
 
With CLI + jar file
```bash
scala2plantuml \
    --jar ./target/scala-2.13/kotti-assembly.jar \
    "com/soundcloud/kotti/Api#armsApiClient."
```

With CLI + dir
```
scala2plantuml --dir 'target/scala-2.13/classes' "com/soundcloud/kotti/Api#" -v --output ./quynh/test.puml
```

With sbt console
```sbt
scala2PlantUML "com/soundcloud/kotti/Api#"
```

### For SCI

SCI project has multiple sub-projects. Hence we **have to** provide `--project` flag.

With CLI + dir
```
scala2plantuml --dir './xml-processor/target/scala-2.12/meta' --project xml-processor "com/soundcloud/supplychainingestion/xmlprocessor/IndividualDeliveriesMergerApp#"


scala2plantuml --dir './xml-processor/target/scala-2.12/meta' --project xml-processor "com/soundcloud/supplychainingestion/xmlprocessor/XmlProcessorApp#"
```

With sbt console. Do this first
```
sbt
...
project xml-processor
```

With sbt console and `exclude`.
```
scala2PlantUML "com/soundcloud/supplychainingestion/xmlprocessor/XmlProcessorApp#" --exclude "com/soundcloud/supplychainingestion/xmlprocessor/model/*" --exclude "org/slf4j/*" --exclude "com/soundcloud/jvmkit/*" --exclude "monix/*" --exclude "com/amazonaws/*"
```

With sbt console and `include` for `IndividualDeliveriesMerger`
```
scala2PlantUML "com/soundcloud/supplychainingestion/xmlprocessor/merger/IndividualDeliveriesMerger#" --include "com/soundcloud/supplychainingestion/*/*/*"

scala2PlantUML "com/soundcloud/supplychainingestion/xmlprocessor/merger/IndividualDeliveriesMerger#" --include "com/soundcloud/supplychainingestion/xmlprocessor/*/*"
```

With sbt console and `include` for `XmlProcessorApp`
```
scala2PlantUML "com/soundcloud/supplychainingestion/xmlprocessor/XmlProcessorApp#" --include
"com/soundcloud/supplychainingestion/*/*"
```

---

## Extra nodes
Build the package
```
 sbt xml-processor/packageSC
```

And then 
```
scala2plantuml \
    --jar xml-processor/target/scala-2.12/supply-chain-ingestion-xml-processor-assembly.jar \
    --project xml-processor \
    "com.soundcloud.supplychainingestion.xmlprocessor.merger.IndividualDeliveriesMergerApp"
```

I would need this [setting](https://github.com/BotTech/scala2plantuml/blob/9207682253433096fa57fe356aaae7b1c33e01ce/build.sbt#L244) to include semanticdb in a packaged jar


