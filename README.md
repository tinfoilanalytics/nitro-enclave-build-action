# Tinfoil Nitro Build Action

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
      - uses: tinfoilanalytics/nitro-enclaves-build-action@v1
        with:
          docker-context: .
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
