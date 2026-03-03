# JDK 25 Linux x64 Build Workflow

This repository contains a GitHub Actions workflow for building OpenJDK 25 for Linux x64 (64-bit) architecture.

The build runs inside a **CentOS 7 Docker container** to produce binaries compatible with **glibc 2.17**, avoiding the `GLIBC_2.34 not found` error on older Linux systems.

## Workflow File

The workflow is defined in `.github/workflows/build-jdk25-linux-x64.yaml`

## Docker Files

- `docker/jdk25-linux-x64/Dockerfile` — CentOS 7–based build environment
- `docker-compose.yml` — Orchestrates the Docker build for local use

## Features

- **Automated Build**: Compiles JDK 25 from the OpenJDK source repository
- **Platform**: Linux x64 (64-bit)
- **glibc 2.17 Compatibility**: Built on CentOS 7 to work on older Linux distributions
- **Boot JDK**: Uses Java 25 EA (Temurin distribution)
- **Artifact Upload**: Automatically packages and uploads the built JDK as a workflow artifact

## Triggers

The workflow can only be triggered manually via the GitHub Actions UI (workflow_dispatch).

## Manual Trigger Options

When manually triggering the workflow, you can specify:

- **jdk_repo**: JDK Git repository URL to clone from (default: `https://github.com/openjdk/jdk.git`). Use this to build from a fork or custom JDK repository.
- **jdk_ref**: JDK commit, branch, or tag to build (default: `jdk-25+11`). Examples:
  - `jdk-25+11` - Specific JDK 25 build tag
  - `master` - Latest development version
  - Any valid commit SHA
- **build_variant**: Choose between `server`, `client`, or `minimal` JVM variants (default: `server`)

## Build Steps (via Docker)

1. **Checkout repository**: Clones this repository
2. **Docker build**: Builds the JDK inside a CentOS 7 container with devtoolset-11 (modern GCC on glibc 2.17)
3. **Extract artifact**: Copies the tarball out of the Docker image
4. **Upload artifact**: Uploads the built JDK as a workflow artifact (retained for 7 days)

## Build Environment (inside Docker)

The CentOS 7 container installs:
- devtoolset-11 (GCC, G++, make)
- autoconf, automake
- zip/unzip
- X11 development libraries
- CUPS development libraries
- Font configuration libraries
- ALSA sound libraries
- FreeType development libraries

## Output

After a successful build, you can download:
- **jdk25-linux-x64**: A tarball containing the complete JDK 25 installation (e.g., `jdk25-linux-x64-0131c1b.tar.gz`)

## Local Build with Docker Compose

You can also build locally using Docker Compose:

```bash
# Build with default settings (jdk-25+11, server variant)
docker compose build
docker compose up

# Build from a custom JDK repository
JDK_REPO=https://github.com/example/jdk.git docker compose build
docker compose up

# Build a specific tag
JDK_REF=jdk-25+12 docker compose build
docker compose up

# Build with minimal variant
BUILD_VARIANT=minimal docker compose build
docker compose up
```

The built tarball will be available at `./output/jdk25-linux-x64-<commit>.tar.gz`.

## Usage

To use the built JDK:

1. Download the artifact from the workflow run
2. Extract the tarball: `tar -xzf jdk25-linux-x64-<commit>.tar.gz`
3. Set JAVA_HOME to the extracted `jdk` directory
4. Add `$JAVA_HOME/bin` to your PATH

## Notes

- The build runs on CentOS 7 (glibc 2.17), ensuring compatibility with older Linux distributions
- The build process can take 1-3 hours or more depending on GitHub Actions runner availability and system load. Full JDK compilation is resource-intensive.
- The workflow uses `--disable-warnings-as-errors` to allow the build to complete even with warnings
- Artifacts are retained for 7 days by default
