# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Ikasan Enterprise Integration Platform (EIP): an open-source Java/Spring Boot framework for building integration modules out of flows and EAI-pattern components. Maven multi-module build, Java 25 source/target (CI runs on OpenJDK 26, Maven 3.9.x). Release branches are named `X.Y.x` (current: `5.1.x`); version is `5.1.1-SNAPSHOT`.

## Build & Test

The real reactor root is `ikasaneip/pom.xml`. The top-level `pom.xml` only wraps it.

```bash
# Full build as CI runs it (from repo root)
mvn --file ikasaneip/pom.xml --settings .settings.xml clean install -Dmaven.javadoc.skip=true -Pci -B

# Build a single module plus the modules it depends on (from ikasaneip/)
mvn -pl component/endpoint/jms-spring -am install -DskipTests

# Run a single test class / method in one module
mvn -pl wiretap test -Dtest=SomeTest
mvn -pl wiretap test -Dtest=SomeTest#someMethod
```

Maven profiles in `ikasaneip/pom.xml` choose which modules are built:
- `all` / `ci` (both active by default): everything, including `sample`. `ci` excludes the flaky `MultiThreadedJmsSampleFlowTest` and `DefaultOperationImplTest` from surefire.
- `core-and-components`: core libraries and components only (adds `component-factory`, leaves out the UI apps and samples).
- `top-level-services-and-applications`: platform, hospital, webconsole, topology, developer, sample, transaction.
- `release-sign-artifacts`: for Maven Central releases. It leaves out `sample`.

Surefire runs with `forkCount=1`, `reuseForks=false` and a long list of `--add-opens` flags, which are set in the parent pom. If you run tests outside Maven, you need the same JVM args. Tests use a mix of JUnit 4 and JUnit 5.

There is no lint or formatter step in the build. `.editorconfig` sets 4-space indentation (2 for JSON, YAML and SQL) and LF line endings.

## Contributor rules (from README)

- Keep logging during `mvn clean install` very low, because CI has a 4MB log limit. The maximum log level is INFO, and `hibernate.show_sql` must always be `false`.
- A new component needs a `README.md` that explains its configuration, plus a sample module or flow that uses it.
- New source files carry the standard Modified BSD licence header, the same one at the top of existing files.

## Architecture

**Core model:** Module → Flows → Components. A *Module* is a deployable integration point (a Spring Boot app) that contains one or more *Flows*. A Flow is a chain of components: a Consumer, then any number of converters, translators, brokers, filters, routers, splitters or sequencers, then Producers. Flows carry events through lightweight event containers.

**Spec vs. implementation:** `ikasaneip/spec/*` holds only interfaces (`org.ikasan.spec.flow`, `.component`, `.module`, `.event`, `.recovery`, `.monitor`, …). Every other module codes against these interfaces. When you change a contract, start in `spec`, then update the implementations.

**Key modules:**
- `flow/visitorPatternFlow`: the runtime flow engine. Flow elements are invoked through the visitor pattern, using per-component-type invokers in `org.ikasan.flow.visitorPattern.invoker`.
- `builder/spring`: the fluent API applications use (`BuilderFactory` → `ModuleBuilder` → `FlowBuilder`, plus `getComponentBuilder()` for off-the-shelf components) and the Spring Boot auto-configuration that wires in core services. `builder/jar` holds the component builder implementations (e.g. `XmlStringToObjectConverterBuilderImpl`).
- `component/*`: reusable components. Endpoints (JMS, Kafka, Pulsar, file transfer/FTP/SFTP, Mongo, S3, email, quartz scheduler, BigQueue…) live under `component/endpoint`, and the generic converters, filters, splitters, routers and validators sit next to them.
- Core services that are bound to each flow automatically, each in its own module: `wiretap`, `replay`, `hospital`, `exclusion`, `error-reporting`, `recovery-manager`, `configuration-service`, `monitor`, `system-event`, `housekeeping`, `harvesting`, `serialiser`, `persistence`, `security`, `transaction`.
- `rest`, `webconsole`, `visualisation`, `topology`, `cli`: management and UI layers over running modules.
- `test`: test harness (`FlowTestHarness`, flow observers and expectations, embedded JMS/FTP/SFTP/DB helpers) for flow-level tests.
- `sample/spring-boot/*`: runnable reference modules. `sample/spring-boot/jms/.../ModuleConfig.java` is the shortest example of how a module is assembled with the builder. `sample/non-functional` holds transaction-failure and performance scenarios.

Some services and components have their own `Readme.md` (for example `hospital`, `replay`, `wiretap`, `configuration-service`, `monitor`, `component`). Read it before changing that area. `UpgradePath.md` lists breaking changes between versions.

## Team conventions
@docs-claude/java-style.md
