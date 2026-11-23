# Project templates

Files to copy into a new (or migrated) .NET repository to plug it into the shared workflows. Rules are in [CONVENTIONS.md](../CONVENTIONS.md).

| File | Purpose |
|---|---|
| `.github/workflows/ci.yml` | Build and test on every push / PR |
| `.github/workflows/release.yml` | On push to `develop`: open the `release/x.y.z → main` PR |
| `.github/workflows/build.yml` | On push to `main`: tag, changelog, GitHub Release with binaries |
| `.config/dotnet-tools.json` | Local `nbgv` tool |
| `version.json` | Nerdbank.GitVersioning configuration |
| `Directory.Build.props` | Nerdbank.GitVersioning package + modern defaults |
| `global.json` | .NET SDK 10 pin |
| `cliff.toml` | git-cliff changelog configuration |

## Placeholders

| Placeholder | Replace with | Example |
|---|---|---|
| `__REPO__` | Repository name (also the release artifact name) | `csv-to-yealink-cfg` |
| `__PROJECT__` | Folder and name of the project to publish | `CsvToYealink.Cli` |

In `build.yml`, keep the block matching the project type (CLI, WPF/WinForms, .NET Framework) and delete the others. In `ci.yml` and `release.yml`, use `windows-latest` for Windows-only projects.

## Setup

From the root of the target repository (PowerShell), on a branch created from `develop`:

```powershell
$templates = "D:\Sviluppo\reusable-workflows\templates"   # local clone of this repo
$repo      = "csv-to-yealink-cfg"
$project   = "CsvToYealink.Cli"

# Copies .github, .config and the root files (overwrites existing ones)
Copy-Item "$templates\*" . -Recurse -Force -Exclude README.md

Get-ChildItem .github\workflows\*.yml, cliff.toml | ForEach-Object {
    (Get-Content $_ -Raw).Replace("__REPO__", $repo).Replace("__PROJECT__", $project) |
        Set-Content $_ -NoNewline
}

dotnet tool restore
dotnet build -c Release
git status
```

Then review `build.yml`, remove old workflow files (e.g. `changelog.yml`) and old versioning code (MinVer, custom MSBuild targets), commit with `ci: integrate reusable workflows` and open a PR to `develop`.

## Notes

- `version.json` starts at `0.1.0`; the *Prepare Release* workflow computes the next version from the latest `v*` tag, so existing repos keep their numbering.
- The `develop` branch must exist before the first release.
- For .NET Framework projects remove `Nullable`/`ImplicitUsings` from `Directory.Build.props`.
