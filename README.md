# test-hello-reusable-workflow

This is a repository to reproduce the issue of GitHub Actions Reusable Worklows on Self-hosted Runners.

## Issue

Reusable Workflows can't be executed in other GitHub Organizations' Self-hosted Runner.
For example, the Reusable Workflow [suzuki-shunsuke/hello-reusable-workflow](https://github.com/suzuki-shunsuke/hello-reusable-workflow/blob/4d45c8dba568207710036908955f0608be508320/.github/workflows/hello.yaml) has an input `runs-on` to change the worfklow job's `runs-on`.

```yaml
on:
  workflow_call:
    inputs:
      runs-on:
        required: false
        type: string
        default: '"ubuntu-latest"'
        description: |
          JSON string for runs-on.
          e.g.
          runs-on: '"macos-latest"'
          runs-on: '["foo"]'
jobs:
  hello:
    runs-on: ${{fromJSON(inputs.runs-on)}}
```

So callers can change the runs-on:

```yaml
test-github-hosted-runner:
  uses: suzuki-shunsuke/hello-reusable-workflow/.github/workflows/hello.yaml@4d45c8dba568207710036908955f0608be508320
  with:
    runs-on: >-
      ["macos-latest"]
```

This works well in case of GitHub hosted runner such as `ubuntu-latest`, `windows-lates`, and `macos-latest`.
But if you specify your Self-hosted Runner labels, the workflow causes the following error.

```
Called workflows cannot be queued onto self-hosted runners across organizations/enterprises. Failed to queue this job. Labels: '<YOUR SELF HOSTED RUNNER LABELS>'.
```

### What we check

We check our GitHub Organization setting and Self-hosted Runner setting, but we can't see any issues.

1. The organization allows all reusable workflows:

> Allow all actions and reusable workflows
> Any action or reusable workflow can be used, regardless of who authored it or where it is defined.

2. The runner allows `All workflows`:

> Workflow access
> Control how these runners are used by restricting them to specific workflows. [Learn more about managing runner groups.](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/managing-access-to-self-hosted-runners-using-groups#changing-what-workflows-can-access-a-runner-group)

3. I can't see any restriction in the side of Reusable Workflow

## How to reproduce

A Self-hosted Runner is required.

1. Copy [the workflow](.github/workflows/test.yaml) to your repository
1. Fix `runs-on` to your Self-hosted Runner labels
1. Run the workflow

Then the workflow job `test-github-hosted-runner` would pass but `test-self-hosted-runner` would fail.

![image](https://github.com/user-attachments/assets/2a8a549a-3364-4b86-875f-7e3b336454e8)

```
Called workflows cannot be queued onto self-hosted runners across organizations/enterprises. Failed to queue this job. Labels: '<YOUR SELF HOSTED RUNNER LABELS>'.
```

## Related issues and discussions

We can find some related issues and discussions, but they are inactive.

- https://github.com/actions/runner/issues/1749
- https://github.com/orgs/community/discussions/25214
- https://github.com/orgs/community/discussions/121692

Especially, a GitHub supporter said the issue was solved in [the comment](https://github.com/orgs/community/discussions/25214#discussioncomment-3246886), but as some people and me said this issue hasn't been solved yet.
Unfortunately, [the discussion](https://github.com/orgs/community/discussions/25214) was treated as `answered`, we can't expect the discussion will be handled again.

So we will create a new discussion.

## Is the restriction appropriate?

```
Called workflows cannot be queued onto self-hosted runners across organizations/enterprises. Failed to queue this job. Labels: '<YOUR SELF HOSTED RUNNER LABELS>'.
```

Apparently, the restriction is intentional.
But I'm not sure if the restriction is appropriate in terms of security.
I'm not sure why we can run third party actions in Self-hosted Runners but can't run Reusable workflows.

Due to this restriction, we can't fully utilize reusable workflows as OSS.
A workaround is to fork Reusable workflows to your GitHub Organizations, but it isn't ideal.

## LICENSE

[MIT](LICENSE)
