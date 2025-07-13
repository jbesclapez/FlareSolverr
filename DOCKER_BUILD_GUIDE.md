# Building Your Own FlareSolverr Docker Images

This guide explains how to build and use your own FlareSolverr Docker images using GitHub Actions.

## Overview

FlareSolverr includes pre-configured GitHub Actions workflows that automatically build and publish Docker images to GitHub Container Registry (GHCR). This allows you to:

- Build your own custom versions
- Avoid dependency on upstream image repositories
- Control your own image versions and updates
- Support multiple architectures (AMD64, ARM64)

## GitHub Actions Workflows

### 1. Development Builds (`build-docker.yml`)
- **Triggers**: Every push to `master`/`develop` branches and pull requests
- **Purpose**: Continuous integration and development image builds
- **Platforms**: Linux AMD64, ARM64
- **Registry**: GitHub Container Registry (ghcr.io)
- **Tags**: Branch names, PR numbers, commit SHAs

### 2. Release Builds (`release-docker.yml`)
- **Triggers**: Git tags (e.g., `v3.3.25`)
- **Purpose**: Official release images
- **Platforms**: Linux 386, AMD64, ARM v7, ARM64 v8
- **Registry**: Both DockerHub and GitHub Container Registry
- **Tags**: Semantic version tags

## Setup Instructions

### Step 1: Fork the Repository
1. Fork the FlareSolverr repository to your GitHub account
2. Clone your fork locally

### Step 2: Configure Repository Settings

#### Enable GitHub Container Registry
1. Go to your repository → **Settings** → **Actions** → **General**
2. Under "Workflow permissions", select **Read and write permissions**
3. Check **Allow GitHub Actions to create and approve pull requests**

#### Repository Secrets (Optional for GHCR)
For GitHub Container Registry, no additional secrets are needed as it uses `GITHUB_TOKEN`.

For DockerHub publishing (release builds only), add these secrets in **Settings** → **Secrets and variables** → **Actions**:
- `DOCKERHUB_USERNAME`: Your DockerHub username
- `DOCKERHUB_TOKEN`: DockerHub access token
- `GH_PAT`: GitHub Personal Access Token (for releases)

### Step 3: Enable Package Visibility
1. Go to your GitHub profile → **Packages**
2. Find your `flaresolverr` package after first build
3. Click **Package settings** → **Change visibility** → **Public** (if desired)

## Building Images

### Automatic Builds
Images are built automatically when you:
- Push to `master` or `develop` branches
- Create pull requests
- Create git tags (for releases)

### Manual Builds
You can trigger builds manually:
1. Go to **Actions** tab in your repository
2. Select **build-docker** workflow
3. Click **Run workflow**
4. Choose whether to push the image

## Using Your Built Images

### Method 1: Environment Variables (Recommended)
Set these environment variables before running docker-compose:

```bash
export GITHUB_REPOSITORY="yourusername/flaresolverr"
export IMAGE_TAG="latest"  # or specific tag like "master-abc1234"
docker-compose up -d
```

### Method 2: Direct Image Override
Update the docker-compose.yml image line:

```yaml
services:
  flaresolverr:
    image: ghcr.io/yourusername/flaresolverr:latest
```

### Method 3: Local Environment File
Create a `.env` file in your project directory:

```env
GITHUB_REPOSITORY=yourusername/flaresolverr
IMAGE_TAG=latest
```

## Available Image Tags

### Development Tags
- `latest` - Latest master branch build
- `master` - Master branch builds  
- `develop` - Develop branch builds
- `master-<commit>` - Specific commit builds
- `pr-<number>` - Pull request builds

### Release Tags
- `v3.3.25` - Specific version releases
- Semantic version patterns

## Image Information

### Platforms Supported
- **Development builds**: `linux/amd64`, `linux/arm64`
- **Release builds**: `linux/386`, `linux/amd64`, `linux/arm/v7`, `linux/arm64/v8`

### Image Registry URLs
- **GitHub Container Registry**: `ghcr.io/yourusername/flaresolverr:tag`
- **DockerHub** (releases only): `yourusername/flaresolverr:tag`

## Troubleshooting

### Build Failures
1. Check the **Actions** tab for detailed logs
2. Ensure your Dockerfile and requirements are valid
3. Verify repository permissions are set correctly

### Image Pull Errors
1. Ensure the image exists: `docker manifest inspect ghcr.io/yourusername/flaresolverr:latest`
2. Check if package is public or authenticate: `docker login ghcr.io`
3. Verify the correct image name and tag

### Authentication Issues
```bash
# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

## Security Best Practices

1. **Use specific tags** instead of `latest` in production
2. **Enable Dependabot** for dependency updates
3. **Review Dockerfile** for security vulnerabilities
4. **Monitor image sizes** and optimize layers
5. **Scan images** for vulnerabilities regularly

## Advanced Configuration

### Custom Build Arguments
Modify the Dockerfile to accept build arguments:

```dockerfile
ARG BUILD_VERSION=dev
ARG BUILD_DATE
LABEL version=${BUILD_VERSION}
LABEL build-date=${BUILD_DATE}
```

### Multi-Stage Optimization
The Dockerfile already uses multi-stage builds for size optimization.

### Cache Optimization
The workflow uses GitHub Actions cache to speed up builds.

## Monitoring and Maintenance

### Build Status
Monitor your builds in the **Actions** tab. Failed builds will show red indicators.

### Image Updates
- Development images update automatically on code changes
- Release images are created when you tag releases
- Consider setting up Dependabot for dependency updates

### Cleanup
Old packages can be deleted from your **Packages** page to save storage.

## Support

If you encounter issues:
1. Check the [FlareSolverr documentation](README.md)
2. Review GitHub Actions logs
3. Check Docker build logs
4. Open an issue in your repository

## Examples

### Complete Workflow Example
```bash
# 1. Set your repository
export GITHUB_REPOSITORY="yourusername/flaresolverr"

# 2. Use latest build
export IMAGE_TAG="latest"

# 3. Start container
docker-compose up -d

# 4. Check logs
docker-compose logs -f flaresolverr

# 5. Test the service
curl http://localhost:8191/v1/
```

### Development Workflow
```bash
# 1. Make changes to code
git add .
git commit -m "feat: add new feature"

# 2. Push to trigger build
git push origin master

# 3. Wait for build to complete (check Actions tab)

# 4. Use the new image
export GITHUB_REPOSITORY="yourusername/flaresolverr"
export IMAGE_TAG="master-$(git rev-parse --short HEAD)"
docker-compose up -d
```

This setup provides you with complete control over your Docker images while maintaining the same functionality as the original FlareSolverr. 