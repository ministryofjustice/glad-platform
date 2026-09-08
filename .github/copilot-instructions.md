# Copilot Instructions for GLAD Platform

See [README.md](../README.md) for project overview, architecture, and module responsibilities.

## Build Commands

### Build the entire platform
```bash
mvn clean install
```

### Build a specific module
```bash
mvn clean install -f glad-bom/pom.xml
# or
mvn clean install -f glad-parent/pom.xml
```

### Run the full build (CI equivalent)
```bash
mvn clean verify
```

### Skip tests
```bash
mvn clean install -DskipTests
```

## Key Conventions

### Version Independence
- `glad-bom` version is independent from `glad-parent` version (see `<glad-bom.version>` property in glad-parent/pom.xml)
- This allows BOM releases for dependency updates without requiring parent releases
- Both are released separately under their own tags (e.g., `glad-bom-2.1.2`, `glad-parent-2.1.1`)

See [README.md](../README.md#module-responsibilities) for module responsibilities and design principles.

### Pull Request Title Format
PR titles must follow Conventional Commits with Jira ticket references (enforced by `pr-title-validation.yml`):
```
type(PROJECT-123): description
```
Valid types: `feat`, `fix`, `docs`, `refactor`, `test`, `ci`, `build`, `perf`, `chore`

Examples:
- `feat(LAA-123): add Spring Boot 4.1 compatibility`
- `fix(LAA-456): update Logback CVE patch`
- `chore: update README`
- `fix(bot): automated dependency update`

### Pre-commit Validation
The repository uses `conventional-pre-commit` (v3.2.0) to validate commits locally. Install pre-commit hooks with:
```bash
pre-commit install
```
This ensures commits follow conventional commits format before pushing to GitHub.

### Language
- **Use British English spelling and terminology** in all code comments, documentation, commit messages, and variable names (e.g., `colour` not `color`, `localise` not `localize`, `organise` not `organize`)

## Release Process

### Release Tags
Tags trigger the `deploy-new-package.yml` workflow automatically:
```bash
git tag -a glad-bom-X.Y.Z -m "Release glad-bom X.Y.Z"
git push origin glad-bom-X.Y.Z

git tag -a glad-parent-X.Y.Z -m "Release glad-parent X.Y.Z"
git push origin glad-parent-X.Y.Z
```

### Manual Release
Trigger `deploy-new-package.yml` workflow manually via GitHub:
1. Go to Actions → Publish Maven Package
2. Click "Run workflow"
3. Select module (`glad-bom` or `glad-parent`)

### Distribution
Packages are published to the GitHub Packages Maven repository:
```
https://maven.pkg.github.com/ministryofjustice/glad-platform
```

## Key Properties to Update

### Dependency Versions (glad-bom/pom.xml)
Update these properties when managing dependency versions:
- `<aws.sdk.version>` - AWS SDK
- `<cucumber.version>` - Cucumber BDD testing
- `<spring-cloud-azure.version>` - Azure integration
- `<poi.version>` - Apache POI (Excel)
- `<gatling.version>` - Load testing
- `<immutables.version>` - Value object generation
- `<springdoc.version>` - OpenAPI/Swagger documentation
- `<playwright.version>` - Browser testing
- `<axe.version>` - Accessibility testing

### Build Configuration (glad-parent/pom.xml)
- `<java.version>` - Java language level (currently 25)
- `<glad-bom.version>` - Which BOM version this parent uses
- `<logback.version>`, `<tomcat.version>` - CVE override candidates for emergency patching

## Maven Settings

### GitHub Packages Authentication
Maven is configured to publish to GitHub Packages. Ensure credentials are configured in `~/.m2/settings.xml`:
```xml
<server>
    <id>github</id>
    <username>YOUR_GITHUB_USERNAME</username>
    <password>YOUR_GITHUB_TOKEN</password>
</server>
```

### Batch Mode
CI/CD workflows use `mvn --batch-mode` to suppress interactive prompts and ensure reproducible builds across all environments.

## Security and Scanning

### Snyk Scanning
The build process includes Snyk vulnerability scanning via the `.github/snyk-scan` custom action. Enabled for:
- Pull request validation (`pr-snyk-scan.yml`)
- Release builds (`deploy-new-package.yml`)

### CVE Management
For critical CVE fixes:
1. Update the affected version property in glad-bom/pom.xml or glad-parent/pom.xml
2. Release a new version immediately (see Release Process below)
3. Document the CVE fix in CHANGELOG.md
4. Use PR title: `fix(CVE-XXXX): patch vulnerable dependency`

## Useful Maven Commands

### Check dependency tree
```bash
mvn dependency:tree -f glad-bom/pom.xml
```

### Inspect effective POM
```bash
mvn help:effective-pom -f glad-parent/pom.xml
```

### Find available dependency updates
```bash
mvn versions:display-dependency-updates -f glad-bom/pom.xml
```

### Validate POM offline
```bash
mvn clean install -o
```

## Important Notes

- **Do not modify the root pom.xml** to include service dependencies—it's an aggregator only
- glad-bom should remain a simple configuration POM with no runtime dependencies
- glad-parent inherits from Spring Boot starter parent and applies organisational standards
- Always run `mvn verify` locally before pushing changes that affect build behaviour
- Breaking changes (e.g. Java version bumps) require careful coordination with consuming services
- Changelog updates are automated by `release-please-config.json`, but major changes should be documented in CHANGELOG.md

## Key Files to Edit

- **glad-bom/pom.xml** - Dependency versions (most common change)
- **glad-parent/pom.xml** - Build configuration, Java version, CVE overrides
- **pom.xml** (root) - Rarely modified; module aggregator only

