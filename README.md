# GLAD Platform

The GLAD Platform provides a centralised Maven dependency and build management solution for GLAD services.

It provides:

- A central dependency management Maven Bill of Materials (BOM) (`glad-bom`)
- A standard Maven parent POM (`glad-parent`)
- A single place to manage dependency versions, CVE fixes, build standards, and common tooling

The goal is to allow consuming applications (GLAD services) to inherit a consistent platform configuration rather than
individually managing dependency versions across multiple repositories.

---

## Architecture

The project is structured as a Maven multi-module project:

```
glad-platform
│
├── glad-bom
│   └── Dependency version management
│
└── glad-parent
    └── Maven build configuration and conventions
```

The relationship between the modules is:

```
Consumer GLAD service(s)
        |
        v
   glad-parent
        |
        +-- spring-boot-starter-parent
        |
        +-- imports glad-bom
                    |
                    +-- Spring ecosystem versions (via spring-boot-starter-parent)
                    +-- AWS SDK versions
                    +-- Azure versions
                    +-- Cucumber versions
                    +-- Internal library versions
                    +-- CVE/Snyk overrides
```

---

## Modules

### glad-bom

`glad-bom` is a Maven Bill of Materials (BOM).

Its sole responsibility is dependency version management. It defines versions for dependencies used across GLAD services.

Examples:

```xml
<logback.version>1.5.36</logback.version>
<tomcat.version>11.0.23</tomcat.version>
<poi.version>5.5.1</poi.version>
```

and manages dependencies through:

```xml
<dependencyManagement>
    <dependencies>
        ...
    </dependencies>
</dependencyManagement>
```

A consuming application does not inherit from the BOM directly. It is imported by `glad-parent`.

---

### glad-parent

`glad-parent` is the standard Maven parent for GLAD services.

It provides:

- Spring Boot parent configuration
- Java version configuration
- Maven plugin defaults
- Common build conventions
- Import of `glad-bom`

Example relationship:

```xml
<parent>
    <groupId>uk.gov.laa</groupId>
    <artifactId>glad-parent</artifactId>
    <version>0.1.0</version>
    <relativePath/>
</parent>
```

A consuming GLAD service should inherit from `glad-parent` instead of directly inheriting from:

```xml
org.springframework.boot:spring-boot-starter-parent
```

---

## Using GLAD Platform in a Consumer Repository

## 1. Configure the parent

Replace the Spring Boot parent:

Before:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.1.0</version>
</parent>
```

After:

```xml
<parent>
    <groupId>uk.gov.laa</groupId>
    <artifactId>glad-parent</artifactId>
    <version>0.1.0</version>
    <relativePath/>
</parent>
```

---

## 2. Keep service dependencies in the application

The platform manages versions, but services still declare the dependencies they require.

For example:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

The service decides what functionality it needs.

The platform decides which versions are used.

---

## Dependency Updates and CVE Management

The point of the platform is that dependency versions are updated in one place.

For example:

A vulnerability is identified in Logback.

Update:

```xml
<logback.version>1.5.36</logback.version>
```

in:

```
glad-bom/pom.xml
```

Release a new BOM version:

```
uk.gov.laa:glad-bom:0.1.1
```

Update the BOM version referenced by `glad-parent`.

Release:

```
uk.gov.laa:glad-parent:0.1.1
```

Consumer applications then only need to update:

```xml
<version>0.1.1</version>
```

in their parent declaration.

No individual repository needs to manage the dependency version.

---

## Building GLAD Platform Locally

From the root directory:

```bash
mvn clean install
```

This builds:

```
glad-bom
glad-parent
```

**The root glad-platform POM is the Maven reactor aggregator for the platform modules. 
It coordinates building glad-bom and glad-parent together but is not intended to be consumed directly as a 
dependency management BOM or application parent.**

---

## Module Responsibilities

| Module | Responsibility |
|---|---|
| `glad-platform` | Maven aggregator |
| `glad-bom` | Dependency versions |
| `glad-parent` | Build configuration |

---

## Design Principles

## Services own dependencies

Services decide:

- Which Spring Boot starters they use
- Which libraries they require
- Which features they enable

The platform does not force unnecessary dependencies.

---

## The platform owns versions

The platform decides:

- Dependency versions
- Supported Spring Boot version
- Security fixes
- Common library versions

---

## Future Extensions

The platform can be extended to include:

- Common Maven plugin configuration
- Security scanning configuration
- Code quality tooling
- Formatting rules
- Test conventions
- Organisation-wide build standards