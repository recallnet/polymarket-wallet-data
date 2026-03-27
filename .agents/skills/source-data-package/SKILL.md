---
name: source-data-package
description: Set up and manage Frictionless Data Package repos created from the recallnet source-data-package-template. Use this skill whenever the user is working in a repo created from the source-data-package-template, mentions adding source data, creating a data package, setting up a new data ingestion repo, registering data resources, or working with datapackage.json. Also use it when the user asks about LFS setup for data repos, date-folder conventions, or Frictionless Data Package structure in the context of recallnet repos.
---

# Source Data Package

This skill guides you through setting up and working with repos created from the `recallnet/source-data-package-template`. These repos follow the [Frictionless Data Package](https://specs.frictionlessdata.io/data-package/) spec and use Git LFS for data files.

## Repo structure

```
datapackage.json          # package descriptor (metadata + resource list)
.gitattributes            # LFS tracking rules for data/**
data/                     # date-stamped folders for raw data (LFS-tracked)
  YYYY-MM-DD/
    source-file.csv
scripts/                  # preparation, transformation, or validation scripts
README.md
```

## First-time machine setup

Git LFS must be installed and configured before cloning or pushing data. Without it, data files won't be stored correctly and clones will get pointer files instead of real data.

1. **Install git-lfs** — on nix-darwin managed machines, add `git-lfs` to `home.packages` in `home.nix` and rebuild. On other systems: `brew install git-lfs` or `apt install git-lfs`.

2. **Configure LFS globally** — either run `git lfs install` or (on nix-darwin) add the filter config to `programs.git.settings`:
   ```nix
   filter.lfs.clean = "git-lfs clean -- %f";
   filter.lfs.smudge = "git-lfs smudge -- %f";
   filter.lfs.process = "git-lfs filter-process";
   filter.lfs.required = true;
   ```

3. **Verify** — `git lfs version` should print a version string and `git config filter.lfs.required` should return `true`.

## Setting up a new data package repo

After creating a repo from the template:

### 1. Update the descriptor

Edit `datapackage.json` and replace the placeholder values:

- `name` — lowercase, url-safe identifier (alphanumeric, `.`, `_`, `-` only). Use the repo name.
- `title` — human-readable title for the dataset.
- `description` — what this data package contains and why it exists.
- `id` — a unique identifier (e.g., a UUID or a doi). Can be left blank initially.
- `sources` — where the data comes from. Fill in `title` (source name) and `path` (URL to the source).
- `licenses` — defaults to CC-BY-4.0. Change if the data has different licensing.
- `version` — starts at `0.1.0`. Bump as the package evolves.

**Example:**
```json
{
  "name": "coingecko-market-data",
  "id": "",
  "title": "CoinGecko Market Data",
  "description": "Daily market snapshots from the CoinGecko API",
  "version": "0.1.0",
  "licenses": [
    {
      "name": "CC-BY-4.0",
      "path": "https://creativecommons.org/licenses/by/4.0/",
      "title": "Creative Commons Attribution 4.0 International"
    }
  ],
  "sources": [
    {
      "title": "CoinGecko API",
      "path": "https://api.coingecko.com/api/v3"
    }
  ],
  "resources": []
}
```

### 2. Add data files

Place data files in date-stamped folders under `data/`:

```
data/2026-03-19/market-snapshot.csv
data/2026-03-19/token-metadata.json
```

The date represents when the data was captured or ingested, not when it was published at the source. Use ISO 8601 format (`YYYY-MM-DD`).

All common data formats placed under `data/` are automatically tracked by Git LFS via `.gitattributes`. The tracked extensions are: csv, tsv, json, jsonl, parquet, arrow, avro, xlsx, xls, sqlite, db, gz, zip, tar, bz2.

If you need to add a format not in this list, append a line to `.gitattributes`:
```
data/**/*.newext filter=lfs diff=lfs merge=lfs -text
```

### 3. Register resources

Every data file should be registered as a resource in `datapackage.json`. Add an entry to the `resources` array for each file:

```json
{
  "name": "market-snapshot",
  "path": "data/2026-03-19/market-snapshot.csv",
  "format": "csv",
  "mediatype": "text/csv",
  "description": "End-of-day market snapshot with price, volume, and market cap"
}
```

Resource fields:
- `name` — lowercase identifier for this resource (same naming rules as the package name)
- `path` — relative path to the file from the repo root
- `format` — file extension (csv, json, parquet, etc.)
- `mediatype` — MIME type (text/csv, application/json, application/vnd.apache.parquet, etc.)
- `description` — what this specific resource contains

When multiple date folders contain the same logical resource (e.g., daily snapshots), register each file as a separate resource or use the most recent one and update the path on each ingestion — whichever convention the team agrees on.

### 4. Add scripts (optional)

Place any data preparation, transformation, or validation scripts in `scripts/`. These might include:

- Fetch scripts that pull data from an API
- Transformation scripts that clean or reshape raw data
- Validation scripts that check data quality

## Common operations

### Adding a new day's data

```bash
mkdir -p data/$(date +%Y-%m-%d)
# copy or fetch data into the new folder
# update resources in datapackage.json
git add data/$(date +%Y-%m-%d)/ datapackage.json
git commit -m "data: add $(date +%Y-%m-%d) snapshot"
```

### Verifying LFS is tracking correctly

```bash
git lfs ls-files          # list LFS-tracked files in the repo
git lfs status            # show pending LFS uploads
```

If a data file shows up in `git diff` as full content instead of a pointer, LFS isn't tracking it — check `.gitattributes` and ensure `git lfs install` has been run.
