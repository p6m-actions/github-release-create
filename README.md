# GitHub Release Create

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/github-release-create?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action that creates a release with Docker image digest as an artifact. This action standardizes the release creation process across multiple projects and language types, making it ideal for use in project archetypes.

## Usage

```yaml
- name: Create Release
  uses: p6m-actions/github-release-create@v1
  with:
    tag: ${{ steps.cut-tag.outputs.tag }}
    digest: ${{ steps.docker-publish.outputs.digest }}
    body: |
      Application version: `${{ steps.cut-tag.outputs.version }}`
      Docker image digest: `${{ steps.docker-publish.outputs.digest }}`
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | - | Git tag for the release |
| `name` | No | `Version {tag}` | Release name/title |
| `digest` | Yes | - | Docker image digest to include as artifact |
| `make-latest` | No | `true` | Mark this release as the latest |
| `generate-notes` | No | `true` | Automatically generate release notes from commits |
| `body` | No | - | Additional release body content (appended after auto-generated notes) |
| `artifacts` | No | - | Additional artifact files to attach (comma-separated paths) |
| `remove-artifacts` | No | `true` | Remove old artifacts before uploading new ones |

## Outputs

| Output | Description |
|--------|-------------|
| `release-id` | The ID of the created release |
| `release-url` | The URL of the created release |
| `digest-artifact` | Path to the digest artifact file (always `digest.txt`) |

## Examples

### Basic Usage (Build Workflow)

```yaml
- name: Create Release
  uses: p6m-actions/github-release-create@v1
  with:
    tag: ${{ steps.cut-patch.outputs.tag }}
    digest: ${{ steps.docker-publish.outputs.digest }}
```

### With Custom Release Body

```yaml
- name: Create Release
  uses: p6m-actions/github-release-create@v1
  with:
    tag: ${{ steps.cut-tag.outputs.tag }}
    name: "Version ${{ steps.cut-tag.outputs.version }}"
    digest: ${{ steps.docker-build.outputs.digest }}
    body: |
      Application version: `${{ steps.cut-tag.outputs.version }}`
      Docker image digest: `${{ steps.docker-build.outputs.digest }}`

      ## Changes
      See generated release notes below for detailed changes.
```

### With Additional Artifacts

```yaml
- name: Create Release
  uses: p6m-actions/github-release-create@v1
  with:
    tag: ${{ steps.cut-tag.outputs.tag }}
    digest: ${{ steps.docker-publish.outputs.digest }}
    artifacts: "checksums.txt,CHANGELOG.md"
    body: |
      Application version: `${{ steps.cut-tag.outputs.version }}`
```

### Updating Existing Release (Promote Workflow)

```yaml
- name: Update Release Notes
  uses: p6m-actions/github-release-create@v1
  with:
    tag: ${{ inputs.tag }}
    digest: ${{ env.DIGEST_TO_PROMOTE }}
    make-latest: false
    generate-notes: false
    body: |
      ${{ steps.get-notes.outputs.original-body }}

      Released to ${{ inputs.environment }} by @${{ github.triggering_actor }} on ${{ steps.get-timestamp.outputs.timestamp }}
```

## Features

- **Language Agnostic**: Works with any project that builds Docker images (Python, JavaScript, Rust, .NET, Java, etc.)
- **Standardized Artifacts**: Always creates a `digest.txt` artifact for environment promotion workflows
- **Flexible**: Supports custom release names, bodies, and additional artifacts
- **Composable**: Designed to work seamlessly with other p6m-actions for a complete CI/CD pipeline
- **Auto-generated Notes**: Leverages GitHub's release notes generation by default