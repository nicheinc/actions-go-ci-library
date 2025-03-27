
# actions-go-ci-library

`actions-go-ci-library` is a reusable
[GitHub Actions](https://docs.github.com/en/actions) workflow providing CI
and other automation for our Go library repos. It currently supports the
following features:

- Ensure the library builds and passes unit tests on all Go versions between the
  minimum supported major version in the `go.mod` file and the latest released
  Go version.
- Validating that open PRs meet our formatting standards.
- Looking for incompatibilities between the module version in `go.mod` and
  source code changes.
- Offers a single GHA job that can be used to gate PR merges on the success of
  all other required jobs in the workflow.

## Usage

See the `service`
[`ci.yaml`](https://github.com/nicheinc/service/blob/main/.github/workflows/ci.yaml)
for a typical example of a library repo workflow that calls
  `actions-go-ci-library`.

Any GHA workflow may make use of this re-usable action with a job like the
following:

```yaml
jobs:
  ci:
    name: Build, Test, and Lint
    uses: nicheinc/actions-go-ci-library/.github/workflows/action.yaml@v1
    secrets: inherit
```

It's important to use `secrets: inherit` so that the action will have permission
to read the same private repos as the calling repo does.

We recommend running the workflow containing this job on all events relevant to
pull requests to ensure that library developers get useful feedback on their
changes.

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
```

And finally, it's a good idea to give your workflow read permissions on the
repository so that the CI job can read the source code in order to build, test,
and lint it.

```yaml
permissions:
  contents: read
```

## Status checks

`actions-go-ci-library` offers the following status check that can be used to
build a branch protection ruleset to gate PR merges based on whether CI passes
for the library:

"Verify that all required jobs passed"

We recommend setting up a branch protection ruleset in your library that:
1. Requires a pull request before merging to the default branch.
2. Requires the "Verify that all required jobs passed" status check to pass
   before pushing to the default branch.
3. Blocks force pushes to the default branch.

By setting up this branch protection rule, you can ensure that all PRs pass the
common CI checks dictated in `actions-go-ci-library` before they can be merged.

## Development

Please see the [CONTRIBUTING.md](CONTRIBUTING.md) file for instructions on how
to contribute to this project.

