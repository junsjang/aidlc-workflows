# Developer's Guide

## Running CodeBuild Locally

You can run AWS CodeBuild builds locally using the [CodeBuild local agent](https://docs.aws.amazon.com/codebuild/latest/userguide/use-codebuild-agent.html). This is useful for testing buildspec changes without pushing to the remote.

### Prerequisites

- Docker installed and running
- The `codebuild_build.sh` script (already included in the repository root)

If you need to re-download the script:

```bash
curl -O https://raw.githubusercontent.com/aws/aws-codebuild-docker-images/master/local_builds/codebuild_build.sh
chmod +x codebuild_build.sh
```

### Basic Usage

```bash
./codebuild_build.sh -i aws/codebuild/standard:5.0 -a ./.codebuild/ \
  -l public.ecr.aws/codebuild/local-builds:$([ "$(arch)" = "arm64" ] && echo "aarch64" || echo "latest")
```

### Overriding the Buildspec

By default the local agent uses `buildspec.yml` in the source directory. Use the `-b` flag to specify a different buildspec file:

```bash
./codebuild_build.sh -i aws/codebuild/standard:5.0 -a ./.codebuild/ \
  -b ./path/to/my-buildspec.yml \
  -l public.ecr.aws/codebuild/local-builds:$([ "$(arch)" = "arm64" ] && echo "aarch64" || echo "latest")
```

### All Script Options

| Flag | Required | Description |
|------|----------|-------------|
| `-i IMAGE` | Yes | Customer build container image (e.g. `aws/codebuild/standard:5.0`) |
| `-a DIR` | Yes | Artifact output directory |
| `-b FILE` | No | Buildspec override file. Defaults to `buildspec.yml` in the source directory |
| `-s DIR` | No | Source directory. First `-s` is the primary source; additional `-s` flags use `<sourceIdentifier>:<sourceLocation>` format for secondary sources. Defaults to the current working directory |
| `-l IMAGE` | No | Override the default local agent image |
| `-r DIR` | No | Report output directory |
| `-c` | No | Use AWS configuration and credentials from your local host (`~/.aws` and `AWS_*` environment variables) |
| `-p PROFILE` | No | AWS CLI profile to use (requires `-c`) |
| `-e FILE` | No | File containing environment variables (`VAR=VAL` format, one per line) |
| `-m` | No | Mount the source directory into the build container directly |
| `-d` | No | Run the build container in Docker privileged mode |

### Using AWS Credentials

To pass your local AWS credentials into the build container:

```bash
./codebuild_build.sh -i aws/codebuild/standard:5.0 -a ./.codebuild/ -c \
  -l public.ecr.aws/codebuild/local-builds:$([ "$(arch)" = "arm64" ] && echo "aarch64" || echo "latest")
```

To use a specific AWS profile:

```bash
./codebuild_build.sh -i aws/codebuild/standard:5.0 -a ./.codebuild/ -c -p my-profile \
  -l public.ecr.aws/codebuild/local-builds:$([ "$(arch)" = "arm64" ] && echo "aarch64" || echo "latest")
```

### Passing Environment Variables

Create a file (e.g. `env.txt`) with one `VAR=VAL` per line:

```
MY_VAR=hello
ANOTHER_VAR=world
```

Then pass it with `-e`:

```bash
./codebuild_build.sh -i aws/codebuild/standard:5.0 -a ./.codebuild/ \
  -e ./env.txt \
  -l public.ecr.aws/codebuild/local-builds:$([ "$(arch)" = "arm64" ] && echo "aarch64" || echo "latest")
```
