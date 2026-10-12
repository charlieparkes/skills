---
name: gitlab-cli
description: >-
  Interact with GitLab through glab, git, and gitlab-ci-local. Use when
  creating or managing merge requests, checking pipelines and jobs, working
  with issues, validating GitLab CI configuration, or performing Git
  branch, commit, and push workflows for a GitLab repository.
---

# GitLab CLI

Use `glab` for GitLab operations and `git` for local repository operations.

## Establish context

Run commands from the target repository because `glab` infers the GitLab
project from its remote:

```bash
git remote -v
git status --short --branch
glab auth status
```

Use `-R <namespace>/<project>` with `glab` commands when operating outside the
target repository. A full GitLab URL or Git URL is also accepted.

Before changing remote state:

1. Read repository instructions.
2. Confirm the current repository, branch, remote, and working tree.
3. Preserve unrelated local changes.
4. Commit, push, approve, merge, close, retry, cancel, or delete only when the
   request authorizes that operation.

Use `glab <command> --help` when the installed version differs from these
examples.

## Git workflow

Create or switch branches:

```bash
git switch -c <branch>
git switch <branch>
```

Update a branch:

```bash
git fetch origin
git rebase origin/<target-branch>
```

Inspect changes before committing or pushing:

```bash
git diff
git diff --cached
git log --oneline origin/<target-branch>..HEAD
```

Stage and commit only the intended files:

```bash
git add <path>...
git commit -m "<summary>"
```

Use a heredoc for a multiline commit message:

```bash
git commit -m "$(cat <<'EOF'
<summary>

<details>
EOF
)"
```

Push:

```bash
git push -u origin HEAD
git push
```

After a rebase, use `git push --force-with-lease`, never `git push --force`.
Do not use interactive Git commands in a noninteractive agent terminal.

Resolve a rebase conflict by editing the files, staging each resolution, and
running:

```bash
git rebase --continue
```

Abort when preserving the pre-rebase state is safer:

```bash
git rebase --abort
```

## Merge requests

Push the source branch before creating a merge request:

```bash
glab mr create \
  --title "<title>" \
  --description "$(cat <<'EOF'
## Summary
- <change>
EOF
)" \
  --target-branch "<target-branch>"
```

Use `glab mr create --fill` when commit messages provide an accurate title and
description.

Find and inspect merge requests:

```bash
glab mr list
glab mr list --author @me
glab mr list --reviewer @me
glab mr list --source-branch "<branch>"
glab mr view <iid>
glab mr diff <iid>
glab mr checkout <iid>
```

Update merge request metadata:

```bash
glab mr update <iid> --title "<title>"
glab mr update <iid> --description "<description>"
glab mr update <iid> --add-label "<label>"
glab mr update <iid> --assignee "<username>"
glab mr update <iid> --reviewer "<username>"
glab mr update <iid> --draft
glab mr update <iid> --ready
```

Manage the merge request lifecycle:

```bash
glab mr approve <iid>
glab mr rebase <iid>
glab mr merge <iid>
glab mr merge <iid> --squash --remove-source-branch
glab mr close <iid>
glab mr reopen <iid>
```

Add a note:

```bash
glab mr note <iid> --message "<message>"
```

Use the API for discussion threads:

```bash
glab api projects/:id/merge_requests/<iid>/discussions
glab api projects/:id/merge_requests/<iid>/discussions/<discussion-id>/notes \
  --method POST \
  --field "body=<reply>"
```

When run in a repository, `glab api` resolves `:id` to the current GitLab
project.

## Pipelines and jobs

Inspect pipelines:

```bash
glab ci list
glab ci status
glab ci status --branch "<branch>"
glab ci view "<branch-or-tag>"
```

Wait for the current branch pipeline without polling:

```bash
glab ci status --wait
```

Inspect, retry, or cancel jobs:

```bash
glab ci trace <job-id>
glab ci retry <job-id>
glab ci cancel job <job-id>
```

Cancel a pipeline:

```bash
glab ci cancel pipeline <pipeline-id>
```

Run a pipeline:

```bash
glab ci run --branch "<branch>"
```

Download artifacts from the most recent successful pipeline for a ref:

```bash
glab job artifact "<ref>" "<job-name>" --path "<destination>"
```

If a pipeline fails, identify the failed job from `glab ci status` or
`glab ci view`, then inspect it with `glab ci trace`.

## Issues

```bash
glab issue list
glab issue list --assignee @me
glab issue create --title "<title>" --description "<description>"
glab issue view <iid>
glab issue update <iid> --add-label "<label>"
glab issue note <iid> --message "<message>"
glab issue close <iid>
glab issue reopen <iid>
```

## CI configuration

Validate the default configuration:

```bash
glab ci lint
```

Validate a specific file or simulate pipeline creation:

```bash
glab ci lint path/to/.gitlab-ci.yml
glab ci lint --dry-run --ref "<ref>"
```

When `gitlab-ci-local` is installed, preview local expansion without running
jobs:

```bash
gitlab-ci-local --file .gitlab-ci.yml --preview
```

Treat local validation as supplemental because GitLab settings, protected
resources, runners, variables, and remote includes can affect server behavior.

## Repositories and releases

```bash
glab repo view
glab repo view --web
glab release list
```

Create and push an annotated tag only when release tagging is requested:

```bash
git tag -a <tag> -m "<message>"
git push origin <tag>
```

## API fallback

Use `glab api` when a dedicated subcommand does not support the operation:

```bash
glab api projects/:id/merge_requests
glab api projects/:id/pipelines
glab api "projects/:id/issues?state=opened"
```

For another project, pass its URL-encoded project path or numeric project ID
instead of `:id`. Use `--paginate` for endpoints that return multiple pages.

## Multi-repository work

For changes spanning repositories:

1. Inspect each repository independently.
2. Use a consistent branch name when appropriate.
3. Keep commits and merge requests scoped to one repository.
4. Run each repository's required checks.
5. Link related merge requests in their descriptions.
