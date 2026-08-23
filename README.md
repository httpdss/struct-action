# StructKit Action

[![GitHub](https://img.shields.io/github/license/httpdss/structkit-action)](LICENSE)

A GitHub Action for running [StructKit](https://github.com/httpdss/structkit) commands in your CI/CD pipeline. Use this action to validate project structure definitions or generate files and folders automatically.

## Features

- 🔍 **Validate** structure definitions on every PR
- 🚀 **Generate** project structures in CI/CD workflows
- 🔄 **Drift Detection** - detect when generated files don't match definitions
- 🛡️ **Safe Defaults** - runs with `--no-hooks` and `--non-interactive` by default
- 📦 **Custom Structures** - supports external structure repositories
- 🎯 **Flexible Versioning** - install from PyPI, git, or specific versions

## Quick Start

### Validate on Pull Request

```yaml
name: Validate Structure
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: httpdss/structkit-action@v1
        with:
          command: validate
```

### Detect Structure Drift

Check if generated files match their definitions (fail if they don't):

```yaml
name: Check Structure Drift
on: [pull_request]

jobs:
  drift-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: httpdss/structkit-action@v1
        with:
          command: generate
          dry_run: true
          diff: true
          fail_on_diff: true
```

### Generate Files

Generate structure without creating a PR (useful for pre-commit hooks or local automation):

```yaml
name: Generate Structure
on:
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: httpdss/structkit-action@v1
        with:
          command: generate
          struct_file: .struct.yaml
          output_dir: .

      - name: Commit changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git commit -m "chore: regenerate structure" || echo "No changes to commit"
          git push
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `command` | Command to run: `validate` or `generate` | No | `validate` |
| `struct_file` | Path to the StructKit configuration file | No | `.struct.yaml` |
| `output_dir` | Output directory for generated files | No | `.` |
| `dry_run` | Run in dry-run mode (preview changes without writing) | No | `false` |
| `diff` | Show diff of changes (use with dry_run for drift detection) | No | `false` |
| `no_hooks` | Disable hooks during execution (check if your StructKit version supports this flag) | No | `false` |
| `non_interactive` | Run in non-interactive mode (check if your StructKit version supports this flag) | No | `false` |
| `structkit_version` | StructKit version to install (version number, `latest`, or git URL) | No | `latest` |
| `structures_path` | Path to custom structures directory | No | `''` |
| `structures_repository` | Custom structures repository to checkout (format: `owner/repo`) | No | `''` |
| `structures_repository_path` | Path within structures_repository where structures are located | No | `structures` |
| `structures_repository_ref` | Git ref (branch/tag/commit) to checkout from structures_repository | No | `main` |
| `extra_args` | Additional arguments to pass to StructKit command | No | `''` |
| `python_version` | Python version to use | No | `3.x` |
| `fail_on_diff` | Fail the action if changes would be made (drift detection) | No | `false` |

## Outputs

| Output | Description |
|--------|-------------|
| `success` | Whether the command completed successfully (`true`/`false`) |
| `has_changes` | Whether the command would make changes (dry-run) or made changes (`true`/`false`) |
| `diff_file` | Path to the diff output file (if generated) |
| `exit_code` | Exit code from the StructKit command |

## Advanced Examples

### Use Specific StructKit Version

```yaml
- uses: httpdss/structkit-action@v1
  with:
    command: validate
    structkit_version: "1.2.3"
```

### Install from Git

```yaml
- uses: httpdss/structkit-action@v1
  with:
    command: generate
    structkit_version: "https://github.com/httpdss/structkit.git@main"
```

### Use Custom Structures Repository

```yaml
- uses: httpdss/structkit-action@v1
  with:
    command: generate
    structures_repository: myorg/my-structures
    structures_repository_path: templates
    structures_repository_ref: v2.0
```

### Generate with Extra Arguments

```yaml
- uses: httpdss/structkit-action@v1
  with:
    command: generate
    extra_args: "--verbose --force"
```

### Use Action Outputs

```yaml
- uses: httpdss/structkit-action@v1
  id: structkit
  with:
    command: generate
    dry_run: true
    diff: true

- name: Check for changes
  if: steps.structkit.outputs.has_changes == 'true'
  run: |
    echo "Structure would be modified!"
    cat ${{ steps.structkit.outputs.diff_file }}
```

### Validate Multiple Structure Files

```yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        struct_file:
          - .struct.yaml
          - config/api.struct.yaml
          - config/frontend.struct.yaml
    steps:
      - uses: actions/checkout@v4
      - uses: httpdss/structkit-action@v1
        with:
          command: validate
          struct_file: ${{ matrix.struct_file }}
```

## Comparison with Reusable Workflow

This action is designed to be a **step** in your workflow, not a complete workflow. Key differences:

| Feature | This Action | Reusable Workflow |
|---------|-------------|-------------------|
| Type | Composite action (step) | Complete workflow (job) |
| PR Creation | No (you control it) | Yes (built-in) |
| Flexibility | High (mix with other steps) | Lower (standalone job) |
| Use Case | Custom workflows | Quick automation |

If you need automatic PR creation, use the [reusable workflow](https://github.com/httpdss/structkit/blob/main/.github/workflows/struct-generate.yaml). If you need fine-grained control over your pipeline, use this action.

## Requirements

- Python 3.x (automatically installed by the action)
- GitHub Actions runner with bash support

## Compatibility Note

This action is designed to work with different versions of StructKit. Some command-line flags (`--no-hooks`, `--non-interactive`, `--diff`, `--dry-run`) may not be available in all versions. The action defaults to not using these flags unless explicitly enabled. Check your [StructKit version's documentation](https://github.com/httpdss/structkit) to confirm which flags are supported.

## How It Works

1. **Setup** - Installs Python and StructKit
2. **Checkout** - Optionally checks out custom structures repository
3. **Execute** - Runs the specified StructKit command
4. **Outputs** - Provides execution results for downstream steps
5. **Cleanup** - Removes temporary files

## Common Patterns

### Pre-merge Validation

```yaml
name: Validate
on:
  pull_request:
    branches: [main]

jobs:
  validate-structure:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: httpdss/structkit-action@v1
        with:
          command: validate
```

### Scheduled Drift Detection

```yaml
name: Daily Drift Check
on:
  schedule:
    - cron: '0 0 * * *'

jobs:
  drift-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: httpdss/structkit-action@v1
        with:
          command: generate
          dry_run: true
          diff: true
          fail_on_diff: true
```

### Manual Generation with Approval

```yaml
name: Generate Structure
on:
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - uses: httpdss/structkit-action@v1
        with:
          command: generate

      - uses: peter-evans/create-pull-request@v7
        with:
          commit-message: "chore: regenerate structure"
          title: "Update generated structure"
          body: "Automated structure regeneration"
          branch: "structkit/update-${{ github.run_id }}"
```

## Troubleshooting

### Command Not Found

If `structkit` command is not found, ensure Python is properly set up:

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.x'
- uses: httpdss/structkit-action@v1
```

### Permission Denied

When using `structures_repository`, ensure your workflow has access:

```yaml
permissions:
  contents: read
```

### Dry Run Shows No Changes

The action detects changes by parsing StructKit output. If the output format changes, the detection might not work. Check the action logs for the actual command output.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Related Projects

- [StructKit](https://github.com/httpdss/structkit) - The main StructKit project
- [StructKit Reusable Workflow](https://github.com/httpdss/structkit/blob/main/.github/workflows/struct-generate.yaml) - Full workflow with PR creation

## Support

For issues and questions:
- [Open an issue](https://github.com/httpdss/structkit-action/issues)
- [StructKit Documentation](https://github.com/httpdss/structkit)
