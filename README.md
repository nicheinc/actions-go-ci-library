# actions-go-ci-library

`actions-go-ci-library` is a reusable
[GitHub Actions](https://docs.github.com/en/actions) workflow providing CI
and other automation for our Go library repos. This action is designed to allow
library maintainers to:

- Ensure the library builds and passes unit tests on all Go minor versions
  between the minimum supported Go minor version in the `go.mod` file and the
  latest released Go minor version.
- Validate that open PRs meet our formatting standards.
- Flag incompatibilities between the module version in `go.mod` and
  source code changes.
- Easily add a branch protection rulesets to gate PRs on whether all required CI
  jobs pass.

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
    secrets:
      NPM_READ_ACCESS: ${{ secrets.NPM_READ_ACCESS }}
```

Any repo using this action must have access to an Actions secret named
`NPM_READ_ACCESS` containing a GitHub PAT token with read access to any private
`nicheinc` repositories that this library depends on, then pass that token as a
secret to the `actions-go-ci-library` re-usable action as above. This token
allows the action to pull down the private dependencies to build and test the
library for library repositories that don't have vendored dependencies checked
into the repo.

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

### Dependabot

This action can also be run on security update PRs created by Dependabot.
However, keep in mind that GHA runs triggered by Dependabot do not use the same
Actions secrets as GHA runs triggered by humans, but instead use a separate set
of Dependabot secrets. If you want to use a workflow that calls this action on
Dependabot PRs, make sure that your repository's Dependabot secrets contains 
an `NPM_READ_ACCESS` secret with a GitHub PAT token that has read access to
any private `nicheinc` repositories that this library depends on.

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

## Deployment

To roll out a new version of `actions-go-ci-library`, you will need to tag a
release following [semver](https://semver.org/) principles and push the new tag
to the GitHub remote. Please use the GitHub Release UI to generate the new tag,
as that will ensure that the tag is accompanied by helpful release notes.

At that point, the [release.yaml](./.github/workflows/release.yaml) workflow
will run and automatically update the "v{major}" tag corresponding to your new
release.

For instance, if you create a new release `v1.2.3`, the `release.yaml` workflow
will automatically create or update the `v1` tag to point to the commit
corresponding to `v1.2.3`. This way, calling repos that are using
`actions-go-ci-library@v1` will automatically pick up the latest release in the
`v1` series without needing to update their workflow files.

Note: this deployment mechanism is only a good idea for repositories in the
`nicheinc` organization, where we can trust that the `v1` tag will only be
updated to point to stable, authorized releases. External callers of this
workflow ought to pin to a specific commit hash instead.
