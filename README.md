# Tinfoil Nitro Enclave Build Action

This action builds, attests, and publishes AWS Nitro Enclave images, automating the entire process from Docker container to signed EIF file.

## Features

- Builds Docker containers optimized for AWS Nitro Enclaves
- Automatically generates EIF (Enclave Image Format) files
- Creates attestation measurements for security verification
- Publishes images to GitHub Container Registry (ghcr.io)
- Automatically generates release notes with measurements and image information

## Prerequisites

- Repository must have these permissions enabled:
  - `contents: write` - For creating releases
  - `packages: write` - For publishing to GitHub Container Registry
  - `id-token: write` - For OIDC attestation
  - `attestations: write` - For signing attestations

- Access to GitHub Container Registry (ghcr.io)


## Usage

Create a workflow file (e.g., `.github/workflows/release.yml`):

```yaml
name: Build and Attest

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: large
    permissions:
      contents: write
      packages: write
      id-token: write
      attestations: write

    steps:
      - uses: actions/checkout@v4
      - uses: tinfoilanalytics/nitro-enclave-build-action@v1
        with:
          docker-context: .
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

This workflow will trigger on any tag matching the pattern `v*`, which is common for semantic versioning. When a tag is pushed, the action will build the Docker image, create an EIF file, and publish the image to GitHub Container Registry.

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `docker-context` | Location of your Docker build context. This directory should contain your Dockerfile and any files needed for the build. | No | `.` |
| `github-token` | GitHub token for authentication. Automatically provided by GitHub Actions for both public and private repos. | Yes | N/A |

## Outputs

The action generates several outputs and artifacts:

### Published Files

- `enclave.eif` - The Nitro Enclave Image File that can be deployed to AWS
- `enclave-info.json` - Contains enclave metadata, measurements, and configuration information
- `predicate.json` - Contains the PCR measurements for attestation

### Container Images

The action publishes an OCI image to GitHub Container Registry:

```
ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

### GitHub Release

A GitHub release is created containing:
- The `enclave-info.json` file
- Release notes with:
  - PCR measurements from `predicate.json`
  - Link to the OCI image
  - SHA256 hash of the EIF file

### Example Release Output

```
Measurements:
{
  "HashAlgorithm": "Sha384 { ... }",
  "PCR0": "4a6fe966f6cadb1...",
  "PCR1": "4b4d5b3661b3efc...",
  "PCR2": "554e3d254f33f81..."
}

OCI image: ghcr.io/myorg/myrepo:v1.0.0
EIF hash: 123456789abcdef...
```

## Security

This action generates attestation measurements that can be used to verify the integrity of your enclave. Always verify these measurements match your expected values before deploying enclaves in production.

## Contributing

Contributions are welcome! Please feel free to submit a PR.

## License

This project is open-sourced under the MIT License - see the [LICENSE](LICENSE) file for details.
