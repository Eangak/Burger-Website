# Release Guide

## Versioning Convention

Burger-Website follows **Semantic Versioning (SemVer)** using the format `vMAJOR.MINOR.PATCH`.

- **MAJOR** (`v2.0.0`): Introduces incompatible changes.
- **MINOR** (`v1.1.0`): Adds new functionality while maintaining backward compatibility.
- **PATCH** (`v1.0.1`): Fixes bugs or makes backward-compatible improvements.

Each release uses a unique Git tag. Docker images are tagged with the same version and pushed to Amazon ECR. Because ECR tags are immutable, an existing release tag must never be overwritten.

## How to Create a Release

1. Ensure the changes are committed, tested, and pushed to the `main` branch.
2. Confirm that the version tag does not already exist:

   ```bash
   git fetch --tags
   git tag --list
   ```

3. Create an annotated tag for the release (example: `v1.0.0`):

   ```bash
   git checkout main
   git pull origin main
   git tag -a v1.0.0 -m "Release v1.0.0"
   ```

4. Push the tag to GitHub to trigger the configured Jenkins release pipeline:

   ```bash
   git push origin v1.0.0
   ```

5. Verify that Jenkins builds and pushes the versioned Docker image to Amazon ECR, deploys the application to Docker Swarm, and confirms the service is healthy.

6. Verify the deployed website and record the release evidence.

## Rollback

If a release fails, use the Jenkins rollback action to redeploy a previously published, known-good image tag. Do not overwrite or delete an existing release tag or immutable ECR image.

## Release Checklist

- [ ] Changes committed and pushed to `main`
- [ ] Version follows Semantic Versioning
- [ ] Git tag is unique and pushed to GitHub
- [ ] Docker image published to Amazon ECR
- [ ] Docker Swarm deployment is healthy
- [ ] Release and rollback evidence recorded