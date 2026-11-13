# HighTec Content Manager – Examples Catalog Generator

This repository contains an automated catalog generator for the **HighTec Content Manager (HCM)**.

The generator scans public GitHub repositories within a configured organization, extracts example project metadata and release information, and generates an `examples.json` catalog that can be consumed by the HighTec Content Manager.

## Overview

The catalog generator automatically:

1. Retrieves all public repositories from the configured GitHub organization.
2. Searches each repository for an `hcm-meta.json` file.
3. Reads the example project's metadata.
4. Retrieves published GitHub releases and their associated source ZIP archives.
5. Calculates the ZIP archive sizes.
6. Generates an `examples.json` catalog.
7. Commits and pushes catalog changes using GitHub Actions.

Only public repositories containing valid metadata and at least one release are included in the generated catalog.

## Repository Structure

```text
.
├── .github/
│   └── workflows/
│       └── main.yml
├── generate_index.py
├── examples.json
└── README.md
```

| File                         | Description                                                           |
| ---------------------------- | --------------------------------------------------------------------- |
| `generate_index.py`          | Python script responsible for generating the catalog                  |
| `examples.json`              | Automatically generated catalog containing available example projects |
| `.github/workflows/main.yml` | GitHub Actions workflow for automated catalog updates                 |
| `README.md`                  | Project documentation                                                 |

## Example Repository Requirements

To be included in the catalog, an example repository must meet the following requirements:

- The repository must be **public**.
- It must contain an `hcm-meta.json` file in its root directory on the default branch.
- It must have at least one published GitHub release.

### Metadata File

Each example repository should contain an `hcm-meta.json` file with the following structure:

```json
{
  "displayName": "AURIX TC3xx Example",
  "abstract": "Example project for Infineon AURIX TC3xx.",
  "device": "TC375",
  "tags": [
    "AURIX",
    "TriCore",
    "Getting Started"
  ],
  "contentType": "ECLIPSE_GENPRJ_STD"
}
```

| Field         | Description                                    |
| ------------- | ---------------------------------------------- |
| `displayName` | Display name shown in the Content Manager      |
| `abstract`    | Short description of the example project       |
| `device`      | Target device                                  |
| `tags`        | Keywords used for categorization and filtering |
| `contentType` | Content type used by the Content Manager       |

If an optional metadata field is missing, the generator uses its predefined default value.

### GitHub Releases

Each example repository must contain at least one GitHub release.

The generator retrieves the release tag, release notes, publication date, and source ZIP archive URL.

Repositories without releases are skipped.

## Generated Catalog

The resulting `examples.json` contains an `examples` array with metadata and version information for each example project.

Example structure:

```json
{
  "examples": [
    {
      "id": "aurix-example",
      "displayName": "AURIX TC3xx Example",
      "abstract": "Example project for Infineon AURIX TC3xx.",
      "device": "TC375",
      "tags": [
        "AURIX",
        "TriCore"
      ],
      "contentType": "ECLIPSE_GENPRJ_STD",
      "githubInfo": {
        "repoUrl": "https://github.com/ORG_NAME/aurix-example"
      },
      "versions": [
        {
          "version": "v1.0.0",
          "isLatest": true,
          "downloadUrl": "https://api.github.com/repos/ORG_NAME/aurix-example/zipball/v1.0.0",
          "releaseNotes": "Initial release",
          "sizeBytes": 102400,
          "lastModifiedEpoch": 1793491200000
        }
      ]
    }
  ]
}
```

The values above are illustrative.

The `isLatest` flag is assigned to the first release returned by the GitHub API. The `sizeBytes` value represents the downloaded source archive size, and `lastModifiedEpoch` contains the release publication timestamp in milliseconds.

## GitHub Actions Automation

The catalog is generated automatically using GitHub Actions.

Workflow location:

```text
.github/workflows/main.yml
```

### Schedule

The workflow runs every day at **02:00 UTC**.

```yaml
on:
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
```

It can also be triggered manually:

1. Open the repository on GitHub.
2. Navigate to the **Actions** tab.
3. Select **Generate Examples Catalog**.
4. Click **Run workflow**.

### Authentication

The workflow uses the automatically generated GitHub Actions token:

```yaml
GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

No additional Personal Access Token or GitHub App is required for accessing public repositories.

The workflow requires `contents: write` permission to commit the generated `examples.json` file back to the repository.

Repository or organization rules must also allow the workflow to push to the target branch.

### Organization Configuration

Set the target GitHub organization in `main.yml`:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  GITHUB_ORG: "YOUR_ORGANIZATION"
```

The generator only scans public repositories from the configured organization.

## Running Locally

### Requirements

- Python 3.11 or newer
- `requests` Python package
- A GitHub access token

Install dependencies:

```bash
pip install requests
```

Configure environment variables.

**Linux / macOS:**

```bash
export GITHUB_ORG="YOUR_ORGANIZATION"
export GITHUB_TOKEN="YOUR_GITHUB_TOKEN"
```

**Windows PowerShell:**

```powershell
$env:GITHUB_ORG = "YOUR_ORGANIZATION"
$env:GITHUB_TOKEN = "YOUR_GITHUB_TOKEN"
```

Run the generator:

```bash
python generate_index.py
```

The generated `examples.json` will be written to the current working directory.

The current script requires `GITHUB_TOKEN` to be set, including when accessing public repositories.

Never commit access tokens or private credentials to the repository.

## Catalog Update Behavior

Each workflow execution regenerates the entire catalog from the current public repositories and their releases.

If the generated `examples.json` differs from the committed version, GitHub Actions automatically creates a commit and pushes the updated file.

If no changes are detected, no commit is created.

Repositories that become private, are removed, lose their metadata file, or have no releases will no longer appear in the newly generated catalog.

## Important Notes

- Only public repositories are processed.
- The `hcm-meta.json` file must be accessible through the GitHub Contents API.
- GitHub releases must exist for examples to appear in the catalog.
- ZIP archive sizes are calculated by requesting the actual archives, which may increase execution time for large repositories.
- The catalog uses GitHub-generated source ZIP archives, not separately uploaded release assets.
- GitHub API rate limits and network errors can affect generation.
- The generated catalog is intended for use by the HighTec Content Manager.

## License

The licenses of individual example projects are defined in their respective GitHub repositories.
