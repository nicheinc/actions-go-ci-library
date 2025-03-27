
# actions-go-ci-library

`actions-go-ci-library` is a reusable
[GitHub Actions](https://docs.github.com/en/actions) workflow providing CI/CD
and other automation for our Go service repos. It currently supports the
following features:

- Running unit tests and basic lints on all open PRs
- Running more comprehensive tests and lints on non-draft PRs, including:
  - Integration testing with
    [API Proctor](https://github.com/nicheinc/api-proctor) (see [Usage](#usage)
    below for the various inputs that control how API Proctor runs in CI)
  - Checking for stale OpenAPI documents and READMEs
  - Looking for incompatibilities between the module version in `go.mod`, the
    app version in `version.env`, and source code changes
- Building and shipping service images on merge to `dev`, when `version.env`
  changes (either manually or via the
  [Merge and Release browser extension](https://github.com/nicheinc/merge-and-release-extension))
- Automatically testing in stage and marking releases production-ready, for PRs
  that have opted into CD
- Promoting releases from stage to production when they're marked
  production-ready (by unchecking "Set as a pre-release" on the release, either
  manually or through CD)
- Creating automatic dependency update PRs on a cron schedule

## Usage

See the `entity` service's
[`ci.yaml`](https://github.com/nicheinc/entity/blob/dev/.github/workflows/ci.yaml)
for a typical example of a service repo workflow that calls `actions-go-ci-library`.

The workflow accepts the following inputs:

- `services` (required) - A JSON array containing the names of all services in
  the repo: the eponymous microservice itself, any associated Kafka consumers,
  etc.

  - Example: `'["service", "service-updater"]'`

- `dependency-update-code-owners` (default `''`) - Comma-separated strings. Each
  string should be a GitHub username or team within the `nicheinc` org. Whenever
  an automated dependency update PR is posted within a repo, the usernames or
  teams set with this input will be assigned as reviewers. If not set, no user
  will be assigned to any automated dependency update PRs.

  - Example: `'nicheinc/delta-be,angelowilliams'`

- `api-proctor-branch` (default `dev`) - The branch of `api-proctor` to use for
  tests. If not set to the default value (`dev`), this branch must be associated
  with an open PR. This input should be left as `dev` unless the service PR
  contains changes that break existing API Proctor tests, requiring an
  associated `api-proctor` PR. This input value will be changed back to `dev`
  automatically when the service PR is merged to dev.

  For now, someone will still need to merge the `api-proctor` PR (ideally around
  the same time the service PR is deployed). In the future, we may automatically
  merge the `api-proctor` branch with the service branch.

- `externally-triggered-api-proctor-branch` - The branch of `api-proctor` to use
  for tests. If this input is set, then this will be used instead of the
  `api-proctor-branch`. This input should only be set when the parent workflow
  (e.g. a service's `ci.yaml` workflow) is triggered by a `workflow_dispatch` or
  `workflow_call` event. If not set to `dev`, this branch must be associated
  with an open PR.
- `dependency-branch-map` (default `'[]'`) - A JSON array mapping dependent
  services to respective branches. If this input is not set, then every
  dependency will just use the latest image in ECR. However, if a dependent
  service needs to use a specific feature branch in order for `api-proctor`
  tests to pass, this input must be used. This is primarily relevant for BFF
  services.

  - Example: `'["fact@feature/your-branch", "entity@bugfix/your-other-branch"]'`

- `development-branch` (default `master`) - The branch of the `development` repo
  to use for tests. If not set to the default value (`master`), this value must
  be associated with an active branch. This input should be left as `master`
  unless the service PR contains changes that break existing API Proctor tests,
  requiring an associated `development` repo PR. This input value will be
  changed back to `master` automatically when the service PR is merged to dev.
- `api-proctor-delay-seconds` (default `0`) - The number of seconds to sleep
  after starting services in Docker before running the API Proctor tests, to
  give services enough startup time. Set this to the smallest value such that
  the tests pass consistently, and include a comment explaining why the delay is
  necessary. This input will hopefully become unnecessary in the future, if we
  can await service readiness automatically.
- `api-proctor-timeout-minutes` (default `30`) - The timeout in minutes for
  running API Proctor tests.
- `check-error-logs` (default `true`) - After running API Proctor tests, check
  the Docker logs for errors. If you disable error log checking, please also
  include a comment explaining why doing so was necessary.
- `run-api-proctor-tests` (default `true`) - Run the API Proctor tests
  associated with this repo. If you disable API Proctor tests, please also
  include a comment explaining why doing so was necessary.
- `check-for-breaking-changes` (default `true`) - Use
  [gorelease](https://pkg.go.dev/golang.org/x/exp/cmd/gorelease) to check for
  breaking changes that violate semantic versioning rules.
- `debug-mode` (default `false`) - Enables debugging mode. When set to true, the
  `debug-logs` job, which normally only runs if another job failed, will always
  run.

Besides these GHA workflow inputs, the workflow also reads PR descriptions to
determine whether to enable CD, based on the presence of a checkbox that reads:

```
[x] Deploy to production automatically upon successful API Proctor tests
```
