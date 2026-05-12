# Custom Jitsi Meet SDK Publishing

This document outlines the architecture, release workflow, and integration details for the custom Jitsi Android SDK published to GitHub Packages.

## Architecture

The project has been modified to publish not only the primary SDK module (`io.github.ali-saranj:custom-jitsi-sdk`) but also the required transient React Native dependencies (originally `com.facebook.react:*`) directly to the GitHub Packages Maven repository. This fat-publishing approach ensures that consumers simply define the target Maven repository and dependency coordinate without needing a local `node_modules` structure.

## Integration / Consumer Setup

1. Generate a GitHub Personal Access Token (PAT) with `read:packages` permissions.
2. In your consumer project's `settings.gradle` or root `build.gradle`, add the GitHub Packages repository:

```gradle
repositories {
    google()
    mavenCentral()
    maven {
        url = uri("https://maven.pkg.github.com/ali-saranj/custom-jitsi-meet")
        credentials {
            username = "YOUR_GITHUB_USERNAME" // Or via GITHUB_ACTOR / local.properties
            password = "YOUR_GITHUB_PAT"      // Or via GITHUB_TOKEN / local.properties
        }
    }
}
```

3. Add the dependency line in your `app/build.gradle`:

```gradle
dependencies {
    implementation("io.github.ali-saranj:custom-jitsi-sdk:2.0.2-dirty") // Replace with the latest tag
}
```

## Local Publishing

You can publish the SDK and its dependencies locally to your Maven Local cache:

```bash
cd android
./gradlew publishToMavenLocal
```

For publishing to GitHub Packages directly:

```bash
cd android
# Ensure GITHUB_ACTOR and GITHUB_TOKEN environment variables or local.properties gpr.user and gpr.key are set.
./gradlew assembleRelease
./gradlew publish -x :sdk:publishReleasePublicationToGitHubPackagesRepository --continue
./gradlew :sdk:publish
```

## CI/CD Workflow

A GitHub Actions workflow is present at `.github/workflows/publish.yml`. It triggers upon tagging a commit matching `v*.*.*` (e.g., `v1.0.0`). The pipeline automatically sets up JDK 17, caches dependencies, and executes the Gradle publish tasks utilizing the default `GITHUB_TOKEN`.

## Troubleshooting

- **401 Unauthorized:** Your credentials are wrong or missing. Check `local.properties` or environment variables `GITHUB_ACTOR` and `GITHUB_TOKEN`.
- **403 Forbidden / 409 Conflict:** The version you are trying to publish already exists or you lack `write:packages` permission. Version numbers dynamically resolve from Git Tags!
- **Missing transient packages:** If compilation fails for `com.facebook.react` packages, it means the dependencies were not pushed. Ensure you ran the full `./gradlew publish` command for the submodules.
