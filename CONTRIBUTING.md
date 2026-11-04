# Contributing

## Issues and security reports

Use [GitHub Issues](https://github.com/gregor-j/dxs500/issues) for bug reports
and feature requests. Describe the expected behavior, actual behavior, and steps
to reproduce a bug.

Do not disclose vulnerabilities in public issues or pull requests. Use
[private vulnerability reporting](https://github.com/gregor-j/dxs500/security/advisories/new)
instead. Never include credentials or other secrets in contributions. Secret
scanning and push protection are enabled.

## Pull requests

1. Create a branch in this repository if you have write access, or use a fork.
2. Keep the change focused and explain its purpose in the pull request.
3. Open the pull request against `main`. Use a clear, descriptive title: it
   becomes the squash commit title.
4. Describe how you checked the change and any limitations. There are currently
   no CI workflows or required status checks.
5. Resolve all review conversations before merging.

All changes to `main` must go through a pull request, including changes made by
administrators. There are no bypass exceptions. An approving review is not
required, but maintainers can still request changes.

## Merging

Only squash merges are enabled. Each pull request becomes one commit on `main`,
which must keep a linear history. Merge commits and rebase merges are disabled.
Force pushes to `main` and deletion of `main` are blocked.

Auto-merge is disabled; a maintainer with the required permissions merges the
pull request manually. Pull request branches in this repository are automatically
deleted after merging.

Signed commits and DCO sign-offs are not required.

## GitHub Actions

When adding workflows, pin every referenced action to a full commit SHA rather
than a branch or version tag. The repository enforces SHA pinning.

The `GITHUB_TOKEN` has read-only permissions by default. Keep permissions minimal
and declare any additional permissions explicitly in the workflow or job.
Workflows cannot approve pull requests.

## Dependency security

Dependabot security alerts and automated security updates are enabled. When
supported dependency manifests are present, Dependabot can open security update
pull requests. Review and merge those changes through the same pull request
process.
