---
date_published: 2026-05-12
date_modified: 2026-05-12
canonical_url: https://ike.network/ike-example-its/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>ike-example-its</artifactId>
  <version>19</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="ike-example-its" rev="19">
  <artifact name="ike-example-its" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='ike-example-its', version='19')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:ike-example-its:19'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "ike-example-its" % "19"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/ike-example-its "19"]
```
