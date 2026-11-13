# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a personal **template/starter repo** for scaffolding new Quarkus projects, not a finished application. The submodules (`quarkus-template-backend-api`, `quarkus-template-backend-persistence`, `quarkus-template-backend-app`) start out as **byte-for-byte identical, unmodified Quarkus quickstart scaffolding** (same `net.benfro.lab` package, same `ExampleResource`/`MyEntity` classes, same Dockerfiles) — new modules are added by copying an existing one and renaming only the `artifactId`. When starting real work from this template, expect to diverge the modules by purpose (`-api` toward REST/presentation, `-persistence` toward data access, `-app` toward whatever the new module is for) and delete whichever boilerplate doesn't apply.

The root `src/main/java/net/benfro/lab/App.java` and `src/test/.../AppTest.java` are leftover `maven-archetype-quickstart` files unrelated to the Quarkus modules; the root `pom.xml` is a plain aggregator (packaging `pom`) that just declares the `<modules>`.

## Build / test / run commands

Each Quarkus module is built independently with its own Maven wrapper. Run commands from inside the module directory, e.g.:

```shell
cd quarkus-template-backend-api

./mvnw quarkus:dev          # live-coding dev mode; Dev UI at http://localhost:8080/q/dev/
./mvnw package              # build target/quarkus-app/quarkus-run.jar
./mvnw package -Dquarkus.package.jar.type=uber-jar   # über-jar build
./mvnw package -Dnative     # native executable (requires GraalVM)
./mvnw package -Dnative -Dquarkus.native.container-build=true   # native build in container, no local GraalVM needed

./mvnw test                                  # unit tests (QuarkusTest)
./mvnw verify                                # also runs IT tests (QuarkusIntegrationTest), activated by -Dnative profile skipITs flag
./mvnw test -Dtest=ExampleResourceTest       # single unit test class
```

From the repo root, `mvn -pl quarkus-template-backend-api -am <goal>` (or the equivalent for another module) builds just that module plus its dependencies via the reactor.

Java target is 21 (`maven.compiler.release`), Quarkus platform version is pinned in the root `pom.xml` (`quarkus.platform.version`).

## Architecture notes

- Each Quarkus module is self-contained (REST resource + Panache entity + Postgres JDBC datasource); modules do not currently depend on each other despite the `-api`/`-persistence`/`-app` naming split.
- Persistence uses **Hibernate ORM with Panache**, active-record style (`MyEntity extends PanacheEntity`). `src/main/resources/import.sql` seeds dev/test data (commented out by default).
- Integration tests follow the standard Quarkus pattern: a `@QuarkusTest` class for in-process tests, paired with a `@QuarkusIntegrationTest` subclass (`*IT.java`) that re-runs the same tests against the packaged artifact (wired via `maven-failsafe-plugin`'s `integration-test`/`verify` goals).
- `src/main/docker/` in each module ships the four standard Quarkus Dockerfiles (`jvm`, `legacy-jar`, `native`, `native-micro`) — pick the one matching the target packaging.
