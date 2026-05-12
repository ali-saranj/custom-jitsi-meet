# Custom Jitsi Meet SDK Publishing

This document outlines the architecture, release workflow, and integration details for the custom Jitsi Android SDK published to GitHub Packages and Azure DevOps Artifacts.

## Architecture

The project has been modified to publish not only the primary SDK module (`io.github.ali-saranj:custom-jitsi-sdk`) but also the required transient React Native dependencies (originally `com.facebook.react:*`) directly to multiple Maven repositories. This "fat-publishing" approach ensures that consumers simply define the target Maven repository and dependency coordinate without needing a local `node_modules` structure.

## Dual Publishing Support

The SDK is automatically published to:
1.  **GitHub Packages:** `https://maven.pkg.github.com/ali-saranj/custom-jitsi-meet`
2.  **Azure DevOps Feed:** `https://tfs.kasraco.net/tfs/Kasra/AllTeams/_packaging/kasra-maven-feed/maven/v1`

## Integration / Consumer Setup

### GitHub Packages
1. Generate a GitHub Personal Access Token (PAT) with `read:packages` permissions.
2. Add the repository to your `settings.gradle` or `build.gradle`:

```gradle
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/ali-saranj/custom-jitsi-meet")
        credentials {
            username = "YOUR_GITHUB_USERNAME"
            password = "YOUR_GITHUB_PAT"
        }
    }
}
```

### Azure DevOps (Kasra Feed)
1. Use your Azure DevOps credentials (Username/PAT).
2. Add the repository to your `settings.gradle` or `build.gradle`:

```gradle
repositories {
    maven {
        url = uri("https://tfs.kasraco.net/tfs/Kasra/AllTeams/_packaging/kasra-maven-feed/maven/v1")
        authentication {
            basic(BasicAuthentication)
        }
        credentials {
            username = "YOUR_AZURE_USER"
            password = "YOUR_AZURE_PAT"
        }
    }
}
```

## Dependency Usage
Add the dependency line in your `app/build.gradle`:

```gradle
dependencies {
    implementation("io.github.ali-saranj:custom-jitsi-sdk:v1.0.0") // Replace with the latest tag
}
```

## Publishing Workflow

### Local Publishing
Publish to your local Maven cache:
```bash
cd android
./gradlew publishToMavenLocal
```

### Remote Publishing
Ensure `local.properties` contains:
```properties
gpr.user=...
gpr.key=...
azure.user=...
azure.token=...
```
Then run:
```bash
cd android
./gradlew clean assembleRelease
./gradlew publish
```

## CI/CD Workflow
The GitHub Actions workflow at `.github/workflows/publish.yml` triggers on `v*.*.*` tags.
Ensure the following secrets are configured in your GitHub repository:
*   `AZURE_ARTIFACTS_USER`
*   `AZURE_ARTIFACTS_TOKEN`
*   `GITHUB_TOKEN` (automatically provided)

## Troubleshooting
- **Read timed out (Azure):** The Azure feed might be slow. Gradle timeouts have been increased in `gradle.properties`.
- **409 Conflict:** The version already exists in the repository. Increment the version tag.
- **SSL_ERROR_SYSCALL:** Network restriction or proxy issue. Check if the feed URL is reachable from your environment.
