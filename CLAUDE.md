# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build (produces target/spring-petclinic-*.jar)
./mvnw package

# Run (starts on port 8085)
./mvnw spring-boot:run

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=OwnerControllerTests

# Build without tests
./mvnw package -DskipTests

# Generate CSS from LESS sources (required before first run in IDE)
./mvnw generate-resources
```

## Architecture

**Package layout** under `org.springframework.samples.petclinic`:

- `model` — JPA base classes only: `BaseEntity` (id), `NamedEntity` (name), `Person` (firstName/lastName). All domain entities extend these.
- `owner` — Core domain: `Owner`, `Pet`, `PetType`, `Visit` entities with their Spring Data `Repository` interfaces and `@Controller` classes. `PetTypeFormatter` bridges string↔`PetType` for form binding. `PetValidator` handles custom validation.
- `vet` — `Vet` and `Specialty` entities, `VetRepository`, `VetController`. The `/vets` endpoint returns JSON; `/vets.html` returns a Thymeleaf view. Vet list is cached (JCache/Ehcache) under the `"vets"` cache name.
- `visit` — `Visit` entity and `VisitRepository`; visit creation is handled by `VisitController` in the `owner` package.
- `system` — `CacheConfiguration` (Ehcache wiring), `WelcomeController` (`/`), `CrashController` (`/oops` for error page testing).

**Data layer**: Spring Data JPA repositories extend `Repository<T, Integer>` directly (not `CrudRepository`), keeping the public API minimal. Custom JPQL queries use `@Query` + `@Param`. No service layer — controllers inject repositories directly.

**Views**: Thymeleaf templates under `src/main/resources/templates/`. Shared layout is `fragments/layout.html`; reusable form fragments are `inputField.html` and `selectField.html`. CSS is compiled from LESS via WRO4J at build time into `target/classes/static/resources/css/`.

**Database profiles**:
- Default: HSQLDB in-memory, auto-initialized from `src/main/resources/db/hsqldb/schema.sql` + `data.sql`
- MySQL: activate with `-Dspring.profiles.active=mysql`; see `application-mysql.properties` and `src/main/resources/db/mysql/`

**CI/CD**: `Jenkinsfile` defines a four-stage pipeline (checkout → build → test → deploy). Deploy runs the jar via `nohup` and logs to `/tmp/petclinic.log`. Maven tool must be configured as `M3` in Jenkins.
