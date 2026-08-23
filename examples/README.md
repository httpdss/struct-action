# StructKit Action Examples

This directory contains example workflows demonstrating various use cases for the StructKit Action.

## Examples

1. **validate-pr.yml** - Validate structure definitions on pull requests
2. **drift-detection.yml** - Detect structure drift in CI
3. **generate-and-commit.yml** - Generate structures and create a PR
4. **scheduled-check.yml** - Scheduled drift detection
5. **custom-structures.yml** - Using external structure repositories

## Usage

Copy any example to your `.github/workflows/` directory and customize as needed.

```bash
cp examples/validate-pr.yml .github/workflows/
```

## Customization Tips

- Replace version pins (`@v1`) with specific commit SHAs for production use
- Adjust `struct_file` paths to match your repository structure
- Configure branch names and PR settings according to your workflow
- Add environment-specific secrets if using private structure repositories
