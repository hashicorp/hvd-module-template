# Contributing

The `-hvd` modules are public and open to community contributors. You can create an issue asking us to review possible issues or requests for updates, or we accept pull-requests.

## Issues

There is a basic issue template to post issues.

## Pull-request

The repo includes a pull-request template with basic requirments for testing and validation which we provide config for basic smoke test such as `terarform fmt, validate` and `terraform-docs`


## Tooling

This repository uses [Task](https://taskfile.dev/) to manage development tasks. Make sure you have Task installed before proceeding.

## Available Tasks

You can list all available tasks by running:

```bash
task
task default
task --list-all
```

## Testing

To run all tests:

```bash
task test
```

This will run both Go and Terraform tests:
- `task test-go`: Runs formatting checks and tests for Go code
- `task test-terraform`: Runs formatting checks, initialization, and validation for all Terraform directories

## Cleaning

To clean the environment:

```bash
task clean
```

This includes:
- `task clean-go`: Cleans the Go environment (runs `go mod tidy`)
- `task clean-terraform`: Removes Terraform directories and lock files

## Documentation

To update Terraform documentation:

```bash
task terraform-docs
```

Note: This requires a `.terraform-docs.yml` configuration file. If you don't have one, you can generate it with:

```bash
task generate-terraform-docs-config
```

## Creating a Release

To create a new release, once a feature branch as been merged from pull request.

```bash
MOD_RELEASE=X.Y.Z task release
```

Requirements:
- GitHub CLI (`gh`) must be installed
- `MOD_RELEASE` environment variable must be set to a valid semver tag (e.g., `1.2.3`)
- The new version must be greater than the current tag

The release process will:
1. Create a release branch
2. Update documentation URLs
3. Create a tagged release
4. Generate release notes
5. Create a GitHub release
6. Clean up the temporary release branch

```
MOD_RELEASE=0.1.2 task release -n
task: [release] export MOD_REPO="terraform-<provider>-<product>-hvd"
export MOD_RELEASE="0.1.2"
export REPO_OWNER="hashicorp"
echo "Releasing hashicorp/terraform-<provider>-<product>-hvd version 0.1.2"

task: [release] git checkout -b rel-${MOD_RELEASE}
task: [release] python .github/scripts/markdown-url-converter/markdown-url-converter.py --repo="hashicorp/terraform-<provider>-<product>-hvd" --release="0.1.2" --overwrite .

task: [release] git commit  --allow-empty -am "Release version ${MOD_RELEASE}"
task: [release] git tag -a ${MOD_RELEASE} -m "${MOD_RELEASE} release"
task: [release] git revert HEAD --no-edit
task: [release] git push origin --tags
task: [release] git push origin rel-${MOD_RELEASE}
task: [release] gh release create 0.1.2 \
--repo hashicorp/terraform-<provider>-<product>-hvd \
--title "0.1.2" \
--generate-notes --verify-tag

task: [release] git push origin --delete rel-${MOD_RELEASE}
```
You can also set the `MOD_RELEASE` variable in a `.env.local` file for convenience.
