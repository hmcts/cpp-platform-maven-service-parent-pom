# cpp-platform-maven-service-parent-pom

`uk.gov.moj.cpp.common:service-parent-pom`

The Maven parent POM for every `cpp-context-*` bounded-context service. It wires together the full dependency stack (framework BOM + platform BOM) and configures all build plugins needed to build a CPP service, including RAML code generation, Liquibase migrations, annotation validation, and WildFly deployment.

## Position in the hierarchy

```
cpp-platform-maven-parent-pom
└── cpp-platform-libraries  (platform-libraries-parent-pom)
    └── cpp-platform-maven-service-parent-pom  ← this project
        └── [all cpp-context-* bounded-context services]
```

## Maven coordinates

| Property | Value |
|---|---|
| `groupId` | `uk.gov.moj.cpp.common` |
| `artifactId` | `service-parent-pom` |
| Parent | `uk.gov.moj.platform.libraries:platform-libraries-parent-pom` (cpp-platform-libraries) |

## What this POM provides

**Dependency imports**
- `cpp-platform-maven-common-bom` (`uk.gov.moj.cpp.common:common-bom`) — all platform + framework dependency versions
- `cpp-platform-libraries` BOM (`platform-libraries-bom`) — platform library artifacts at a consistent version

**Code generation plugins** (version-managed; activated per module as needed)
- `messaging-adapter-generator-plugin` — generates JMS message listener adapters from messaging RAML
- `messaging-client-generator-plugin` — generates JMS message sender clients
- `direct-client-generator-plugin` — generates direct (in-process) command clients
- `rest-client-generator-plugin` — generates REST client stubs from RAML
- `rest-adapter-generator-plugin` — generates JAX-RS REST resource classes from RAML
- `unifiedsearch-client-generator-plugin` — generates unified-search query clients
- `pojo-generation-plugin` — generates Java POJOs from JSON Schema
- `catalog-generation-plugin` — generates JSON Schema catalog files

**Build conventions for service modules**
- `cpp.service-component` property — identifies which framework service component type a module provides (`COMMAND_API`, `COMMAND_HANDLER`, `EVENT_LISTENER`, `QUERY_API`, `QUERY_VIEW`, etc.)
- `cpp.service.name` — service identifier used for JNDI namespace (`java:app/<service.name>/<key>`)
- Schema generation source directories: `src/raml/json/schema` and `src/main/resources/json/schema`
- Annotation validation plugin (`annotation-validator-maven-plugin`) — validates `@Handles`, `@ServiceComponent`, and related annotations

**Enforcer rules**
- Inherits `enforce-moj-latest-interfaces` from `cpp-platform-maven-parent-pom` (cannot be skipped)
- `enforcer.skipLatestCoreDomainRule` and `enforcer.skipLatestServiceParentPomRule` flags available to temporarily bypass version-freshness checks during development

## Usage

All `cpp-context-*` service modules declare this as their parent:

```xml
<parent>
    <groupId>uk.gov.moj.cpp.common</groupId>
    <artifactId>service-parent-pom</artifactId>
    <version>${service-parent-pom.version}</version>
</parent>
```

The root `pom.xml` of each context project also declares `<cpp.service.name>` to set the JNDI application namespace for that service.
