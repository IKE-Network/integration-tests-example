---
date_published: 2026-08-10
date_modified: 2026-08-10
canonical_url: https://ike.network/integration-tests-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>integration-tests-example</artifactId>
  <version>26</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="integration-tests-example" rev="26">
  <artifact name="integration-tests-example" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='integration-tests-example', version='26')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:integration-tests-example:26'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "integration-tests-example" % "26"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/integration-tests-example "26"]
```
