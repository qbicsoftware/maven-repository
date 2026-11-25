# Third-party contents of `zzzz-openbis-api`

This repository redistributes a single binary, `zzzz-openbis-api-20.10.7.3-r1700646105.jar`,
which is a **shaded uber-jar**. It is not a plain build of the openBIS API: it bundles 13
third-party libraries into one archive.

Recorded here because the artifact's POM is empty (no `<licenses>`, no `<dependencies>`), so
nothing in the Maven metadata discloses what the jar contains.

## Why this file exists

An earlier version of `README.md` claimed the jar was *"byte-identical to the ones published
by the openBIS project"* and that the artifact had *"no dependencies"*. The second claim was
true of the POM and false of the jar. Anyone can disprove it with `unzip -l`. The corrected
wording is above; this file records what is actually inside.

## Method

Studio, reproducible — no manual inventory:

```bash
JAR=releases/life/qbic/zzzz-openbis-api/20.10.7.3/zzzz-openbis-api-20.10.7.3-r1700646105.jar

# Bundled coordinates, read from the POMs shading left behind in the archive.
# Field order in pom.properties is not stable, so read each key explicitly - piping all
# three through one cut/paste emits version:groupId:artifactId for some entries.
unzip -Z1 "$JAR" | grep -E '^META-INF/maven/[^/]+/[^/]+/pom.properties$' | while read -r p; do
  g=$(unzip -p "$JAR" "$p" | sed -n 's/^groupId=//p'    | tr -d '\r')
  a=$(unzip -p "$JAR" "$p" | sed -n 's/^artifactId=//p' | tr -d '\r')
  v=$(unzip -p "$JAR" "$p" | sed -n 's/^version=//p'    | tr -d '\r')
  printf '%s:%s:%s\n' "$g" "$a" "$v"
done | sort -u

# Licence and notice files carried in the archive
unzip -Z1 "$JAR" | grep -iE '^META-INF/(LICENSE|NOTICE)'
```

## Bundled libraries

All 13 are Apache 2.0:

| Coordinate |
|---|
| `com.fasterxml.jackson.core:jackson-annotations:2.9.7` |
| `com.fasterxml.jackson.core:jackson-core:2.9.7` |
| `com.fasterxml.jackson.core:jackson-databind:2.9.7` |
| `commons-io:commons-io:2.6` |
| `commons-logging:commons-logging:1.1.1` |
| `org.apache.commons:commons-lang3:3.11` |
| `org.apache.httpcomponents:httpclient:4.3.6` |
| `org.apache.httpcomponents:httpcore:4.3.3` |
| `org.eclipse.jetty:jetty-client:9.4.44.v20210927` |
| `org.eclipse.jetty:jetty-http:9.4.44.v20210927` |
| `org.eclipse.jetty:jetty-io:9.4.44.v20210927` |
| `org.eclipse.jetty:jetty-util:9.4.44.v20210927` |
| `org.slf4j:slf4j-api:1.7.7` |

Rough size share, by top-level package: `org/*` 6,537 entries, `ch/*` (openBIS + CISD) 1,544,
`com/*` 851. The bundled libraries are the majority of the archive by entry count.

## openBIS itself

Apache 2.0, stated by the vendor: *"openBIS is an open-source software distributed under the
Apache Licence 2.0"* — <https://openbis.ch/index.php/downloads/>.

openBIS is developed by the Scientific IT Services group at ETH Zurich.

## What is not claimed

- **Not verified** that this jar equals any upstream openBIS binary. It carries a `life.qbic`
  coordinate and an empty POM, so no upstream equivalent is identifiable from the metadata.
- The SHA-1 recorded in `README.md` verifies a **transfer**, not provenance.
- The licence scan is per-library from packaged `LICENSE` files, not a legally reviewed audit.
  If a bundled library ever gains a non-Apache dependency, this file will not catch it.
