---
{"dg-publish":true,"permalink":"/4-projects/41-learning-scala/fix-error-semanticdb-scalac-not-found/"}
---

[[4_Projects/41-LearningScala/Scala with Metals + Bloop + VSCode\|Scala with Metals + Bloop + VSCode]]; 



# Fix error semanticdb-scalac not found

## Error

The error is like this:
```
2022.10.25 21:05:19 INFO [error] not found: https://maven.localproxy.sc:42042/sc-repo/org/scalameta/semanticdb-scalac_2.13.7/4.4.28/semanticdb-scalac_2.13.7-4.4.28.pom
```

## Problem 

- The reason is sbtkit is using sbt 1.6.1
- And sbt 1.6.1 was release on Dec 29, 2021, and it uses semanticdb-scalac version 4.4.28 ([code](https://github.com/sbt/sbt/blob/v1.6.1/build.sbt#L49)).
- And semanticdb-scalac version 4.4.28 was released on Sep 15, 2021 ([code](https://github.com/scalameta/scalameta/releases/tag/v4.4.28))
- Scala 2.13.7 however was only released on 1 November 2021 [here](https://www.scala-lang.org/news/2.13.7)\
- Hence semanticdb-scalac 4.4.28 was only built for Scala 2.13.6 at that time.
- Some useful link
    - Maven repository: https://mvnrepository.com/artifact/org.scalameta/semanticdb-scalac

## Solution

I need to change `build.sbt` so that it explicitly use Scala "2.13.6" instead of `scala213`