# QBiC Maven Repository

A small, static Maven repository for artifacts that QBiC projects need but that are not
available on Maven Central. It is a plain HTTPS file tree — no credentials, no accounts,
no server to maintain.

Served from GitHub Pages: **https://qbicsoftware.github.io/maven-repository/releases**

## What is hosted

| Coordinate | Files | SHA-1 |
|---|---|---|
| `life.qbic:zzzz-openbis-api:20.10.7.3` (pom) | `zzzz-openbis-api-20.10.7.3.pom` | `26d3f5a64c9830733afdbb326612ec10bb517cef` |
| `life.qbic:zzzz-openbis-api:20.10.7.3:r1700646105` (jar) | `zzzz-openbis-api-20.10.7.3-r1700646105.jar` | `5c7350674e178b6c19a0c4946cc7d97470693001` |

Each artifact ships a `.sha1` sidecar in Maven's format (40 hex characters), so a consumer
can keep `<checksumPolicy>fail</checksumPolicy>` and have it mean something.

Layout:

```
releases/
  life/qbic/zzzz-openbis-api/20.10.7.3/
    zzzz-openbis-api-20.10.7.3.pom
    zzzz-openbis-api-20.10.7.3.pom.sha1
    zzzz-openbis-api-20.10.7.3-r1700646105.jar
    zzzz-openbis-api-20.10.7.3-r1700646105.jar.sha1
```

## Why it exists

`zzzz-openbis-api` was previously resolved from the QBiC Nexus instance at
`qbic-repo.qbic.uni-tuebingen.de`. That host is not reachable from GitHub Actions egress,
which broke every CI build. The artifact's POM declares no parent, no dependencies and no
plugins, so as a *Maven coordinate* it needs no dependency graph — mirroring it is two files
and two checksums. Hosting it here removes the only external dependency the QBiC Java reactor
had on that Nexus.

The `zzzz-` prefix is deliberate upstream: openBIS uses `zzzz*` artifactIds so its own
modules sort last in a reactor build. It is not a placeholder or a typo.

## Licensing

openBIS is **Apache 2.0**, per the vendor's own statement: *"openBIS is an open-source
software distributed under the Apache Licence 2.0"* —
<https://openbis.ch/index.php/downloads/>. That permits redistribution.

**What the jar actually contains matters here.** The POM is empty, but the jar is a *shaded
uber-jar*: openBIS API classes plus 13 bundled libraries, each carrying its own
`META-INF/LICENSE` and POM inside the archive.

- It is **not** the unmodified openBIS binary, and the POM is **not** byte-identical to an
  upstream one. It is a QBiC repackaging published under a `life.qbic` coordinate.
- The SHA-1 values above are recorded so a *transfer* can be verified against what was
  mirrored. They are not a claim of upstream provenance.

The 13 bundled libraries, all Apache 2.0 — inventory taken from the `pom.properties`
entries inside the jar:

```
com.fasterxml.jackson.core:jackson-annotations:2.9.7
com.fasterxml.jackson.core:jackson-core:2.9.7
com.fasterxml.jackson.core:jackson-databind:2.9.7
commons-io:commons-io:2.6
commons-logging:commons-logging:1.1.1
org.apache.commons:commons-lang3:3.11
org.apache.httpcomponents:httpclient:4.3.6
org.apache.httpcomponents:httpcore:4.3.3
org.eclipse.jetty:jetty-client:9.4.44.v20210927
org.eclipse.jetty:jetty-http:9.4.44.v20210927
org.eclipse.jetty:jetty-io:9.4.44.v20210927
org.eclipse.jetty:jetty-util:9.4.44.v20210927
org.slf4j:slf4j-api:1.7.7
```

No license or notice files are added or removed — the full texts travel inside the archive
under `META-INF/`. See `THIRD-PARTY.md` for the per-library detail and the method used to
derive it.

## Consuming it

Add to the `<repositories>` block of your POM:

```xml
<repository>
  <id>qbic-maven-repository</id>
  <name>QBiC Maven Repository</name>
  <url>https://qbicsoftware.github.io/maven-repository/releases</url>
  <releases>
    <checksumPolicy>fail</checksumPolicy>
    <enabled>true</enabled>
  </releases>
  <snapshots>
    <enabled>false</enabled>
  </snapshots>
</repository>
```

Then declare the dependency as before — no coordinates change, only the host:

```xml
<dependency>
  <groupId>life.qbic</groupId>
  <artifactId>zzzz-openbis-api</artifactId>
  <version>20.10.7.3</version>
  <classifier>r1700646105</classifier>
</dependency>
```

Do **not** add `<updatePolicy>always</updatePolicy>`. This is a static repository; an
`always` policy turns an unreachable host from a one-time failed lookup into a per-build
stall, and it is what made the old Nexus entries so painful.

## Current consumers

- [qbicsoftware/data-manager-app](https://github.com/qbicsoftware/data-manager-app)

`data-download-server` and `openbis20-java-dropboxes` also consume openBIS and are
encouraged to point their own POMs here.

## Operations

This repository is **static**. Nothing publishes to it, and there is no
`maven-metadata.xml` — nothing here is a `-SNAPSHOT` and nothing uses version ranges.
Adding an artifact is a commit of the files plus their `.sha1` sidecars to the
`gh-pages` branch.

Because it is static, it is cheap to trust: the content of a coordinate never changes
underneath a build. If a coordinate must be replaced, publish a new version instead.

### Verifying a file by hand

```bash
curl -sSLO https://qbicsoftware.github.io/maven-repository/releases/life/qbic/zzzz-openbis-api/20.10.7.3/zzzz-openbis-api-20.10.7.3-r1700646105.jar
shasum -a 1 zzzz-openbis-api-20.10.7.3-r1700646105.jar   # expect 5c7350674e178b6c19a0c4946cc7d97470693001
```
