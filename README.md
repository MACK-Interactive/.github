# .github

## Reusable workflow: add subtasks

This repository exposes a reusable workflow at `.github/workflows/add-subtasks.yml`.

It creates and links default sub-issues when:

- an issue is opened with the `with-subtasks` label
- or the `with-subtasks` label is added later

Each run also prepends a managed status badge block to the parent issue body and updates it on reruns.
The badge remains visible and reflects the latest workflow result:

- `https://img.shields.io/badge/sub--issues-added-brightgreen`
- `https://img.shields.io/badge/sub--issues-not%20added-red`

### Default subtasks

By default, the workflow creates:

- `Design`
- `Dev`
- `API Update`
- `QA`

### Consumer repository setup

In each consuming repository, add a caller workflow such as:

```yaml
name: Add default sub-issues

on:
  issues:
    types: [opened, labeled]

permissions:
  issues: write
  contents: read

jobs:
  add-subtasks:
    uses: MACK-Interactive/.github/.github/workflows/add-subtasks.yml@main
    with:
      owner: ${{ github.repository_owner }}
      repo: ${{ github.event.repository.name }}
      issue_number: ${{ github.event.issue.number }}
      issue_title: ${{ github.event.issue.title }}
      action: ${{ github.event.action }}
      has_required_label: ${{ contains(github.event.issue.labels.*.name, 'with-subtasks') }}
      label_name: ${{ github.event.label.name || '' }}
```

### Optional overrides

You can override the required label and subtask list from the caller workflow:

```yaml
with:
  required_label: with-subtasks
  subtasks_json: '["Design","Backend","Frontend","QA","Docs"]'
  link_mode: auto
  subtask_body_template: |
    ## Context
    Sub-task for #{{PARENT_ISSUE_NUMBER}}
    ## Parent issue
    #{{PARENT_ISSUE_NUMBER}}
    ## Scope
    {{SUBTASK_NAME}}
    ## Checklist
    - [ ] Implement
    - [ ] Test
    - [ ] Document
```

`subtasks_json` must be a non-empty JSON array of strings.
The workflow enforces a maximum of 20 subtasks and 120 characters per subtask.
`link_mode` is optional and is forwarded to the sub-issue link API when set.
`subtask_body_template` is optional. Supported placeholders are `{{PARENT_ISSUE_NUMBER}}`, `{{PARENT_ISSUE_TITLE}}`, and `{{SUBTASK_NAME}}`.
By default, created sub-issue titles use only the parent issue ID: `#<parent_issue_number> - <subtask_name>`.

### Security scope

This reusable workflow only allows targets under `MACK-Interactive` (owner check in workflow script).

### Version pinning recommendation

For production stability, pin to a tag or commit SHA instead of `@main`.
