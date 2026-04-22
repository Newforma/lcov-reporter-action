# LCOV Reporter Action

Newforma's fork of the [romeovs/lcov-reporter-action](https://github.com/romeovs/lcov-reporter-action) GitHub Action. Parses LCOV coverage reports and generates an HTML coverage summary.

## Fork history

This repository is a fork of a fork:

```
romeovs/lcov-reporter-action        (original, now unmaintained)
  └─ 06393993/lcov-reporter-action   (added job-summary support, @actions/core bump)
       └─ Newforma/lcov-reporter-action  (this repo)
```

The upstream (`romeovs`) has been marked as unmaintained. There are no meaningful upstream changes to merge — the only addition since our fork point is `pull_request_target` event support, which we don't use.

## How Newforma uses this action

This action is consumed by `platform-github-actions` in the `lint-unit-test-check-ui` workflow. It runs **after** Jest produces a coverage report, and posts the results to the **GitHub Actions job summary** (not as a PR comment).

```yaml
- name: Coverage Report
  uses: Newforma/lcov-reporter-action@master
  with:
    lcov-file: assets/ui/${{ inputs.uiFolderName }}/coverage/lcov.info
    post-to: job-summary
```

We only use two inputs: `lcov-file` and `post-to`. All other inputs fall through to their defaults.

## Note on dependencies

The runtime dependencies are:

- **`@actions/core` ^1.10.0** — Two major versions behind (latest is v3). The v1→v3 breaking changes are only about dropping older Node.js support — the APIs we use (`getInput`, `setFailed`, `summary.addRaw().write()`) are stable across all versions. Works fine as-is on Node 24.
- **`@actions/github` ^1.1.0** — Outdated (v1 from 2019). Uses the legacy `new GitHub(token)` API. However, since we only use `post-to: job-summary`, the GitHub API client is never exercised — the posting goes through `@actions/core`'s summary API exclusively. The `@actions/github` dependency only becomes relevant if the PR comment or commit comment features are used, which we do not.
- **`lcov-parse` ^1.0.0** — Stable. Parses LCOV format into structured data.

The dev toolchain (Rollup 1.x, Jest 25, Babel, Prettier 1.x) is also dated but functional. The pre-built `dist/main.js` bundle has been tested and runs correctly on Node 24. There is no need to rebuild unless the source code changes.

## Inputs

| Input                  | Required | Default                | Description                                             |
| ---------------------- | -------- | ---------------------- | ------------------------------------------------------- |
| `github-token`         | No       | `${{ github.token }}`  | GitHub token for posting PR comments                    |
| `lcov-file`            | No       | `./coverage/lcov.info` | Path to the LCOV file (relative to `working-directory`) |
| `lcov-base`            | No       |                        | Path to a base branch LCOV file for diff comparison     |
| `filter-changed-files` | No       | `false`                | Only report coverage for files changed in the PR        |
| `delete-old-comments`  | No       | `false`                | Delete previous coverage comments before posting        |
| `working-directory`    | No       | `./`                   | Working directory for resolving file paths              |
| `post-to`              | No       | `comment`              | Where to post the report: `comment` or `job-summary`    |
| `title`                | No       |                        | Title to include in the coverage report                 |

## Acknowledgements

The initial code is based on [ziishaned/jest-reporter-action](https://github.com/ziishaned/jest-reporter-action).

Changed file retrieval based on [jitterbit/get-changed-files](https://github.com/jitterbit/get-changed-files).
