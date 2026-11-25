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
which broke every CI build. Because the artifact's POM declares no parent, no dependencies
and no plugins, mirroring it is two files and two checksums — no dependency graph to
reproduce. Hosting it here removes the only external dependency the QBiC Java reactor had
on that Nexus.

The `zzzz-` prefix is deliberate upstream: openBIS uses `zzzz*` artifactIds so its own
modules sort last in a reactor build. It is not a placeholder or a typo.

## Licensing

openBIS is licensed under the **Apache License 2.0**, which permits redistribution of the
unmodified binary. The jar and POM here are byte-identical to the ones published by the
openBIS project; their SHA-1 values are recorded above so any copy can be checked against
the original.

No license files are added or removed. See the upstream project for the authoritative
licence text: <https://openbis.ch/>

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
