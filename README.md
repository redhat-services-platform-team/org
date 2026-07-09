# org

Red Hat Services Platform Team GitHub Organization Metadata

This repository manages the GitHub organization configuration for **redhat-services-platform-team** using [Peribolos](https://docs.prow.k8s.io/docs/components/cli-tools/peribolos/). All changes to org membership, teams, repositories, and permissions are made through the `config.yaml` file and applied automatically via a GitHub Actions pipeline.

## How It Works

1. You submit a change to `config.yaml` via a pull request.
2. The pipeline runs a **dry-run validation** against the live GitHub org to verify the change is safe.
3. An org admin reviews and approves the pull request.
4. Once merged to `main`, the pipeline runs again with `--confirm` to **apply the changes** to the GitHub org.

### When the pipeline runs

The **Sync Org Configuration** workflow runs only when `config.yaml` is changed:

| Event | Trigger |
|---|---|
| Pull request to `main` | `config.yaml` changed → dry-run validation |
| Merge to `main` | `config.yaml` changed → apply with `--confirm` |
| README or other files only | Workflow does **not** run |

Changes to `README.md` alone do not need Peribolos validation, but branch protection still requires the `peribolos` status check to merge. For documentation-only pull requests, ask an org admin to merge with **bypass rules** after review.

## What You Can Request

| Request Type | Where to Edit in `config.yaml` |
|---|---|
| New repository | Add an entry under `repos:` at the org level |
| New team | Add an entry under `teams:` |
| New team member | Add your GitHub username to `members:` (org level) **and** to the relevant team |
| Remove org or team member | Remove the username from `members:` (org level and/or team) |
| Change team maintainers | Edit `maintainers:` on the team |
| Team repo permissions | Add or update the repo under the team's `repos:` with a permission level |
| Remove team repo access | Remove the repo from the team's `repos:` section |
| Change repo settings | Modify fields under the repo's entry in `repos:` |

### Permission Levels

| Level | Description |
|---|---|
| `read` | View and clone the repository |
| `triage` | Read plus manage issues and pull requests |
| `write` | Push to the repository and manage issues/PRs |
| `maintain` | Write plus manage repo settings (no destructive actions) |
| `admin` | Full access including settings, secrets, and team management |

## Existing Teams

Use this table to find the right team when requesting access. See `config.yaml` for the current membership lists.

| Team | Description | Repository Access |
|---|---|---|
| `maintainers` | Organization maintainers | `org`: write |
| `americas-spt` | Americas Ansible SPT | `ready-to-use-content`: write |
| `americas-spt-admins` | Americas Ansible SPT Admins | `ready-to-use-content`: admin |
| `naps-sec` | NAPS Security Team | `rhel-stig-content`: write |
| `naps-sec-admins` | NAPS Security Team Admins (nested under `naps-sec`) | `rhel-stig-content`: admin |

Team maintainers can manage team membership. Contact a team maintainer or an org admin if you are unsure which team to join.

## Step-by-Step: Submitting a Change

### 1. Clone the repository

Using SSH:

```bash
git clone git@github.com:redhat-services-platform-team/org.git
cd org
```

Using HTTPS:

```bash
git clone https://github.com/redhat-services-platform-team/org.git
cd org
```

### 2. Create a branch

```bash
git checkout -b request/my-change-description
```

Use a descriptive branch name, for example:
- `request/add-repo-my-new-project`
- `request/add-team-platform-ops`
- `request/add-user-jsmith`

### 3. Edit `config.yaml`

Open `config.yaml` and make your changes. See the examples below for common scenarios.

### 4. Commit and push

```bash
git add config.yaml
git commit -m "Request: add my-new-repo repository"
git push -u origin request/my-change-description
```

### 5. Open a pull request

Go to [the pull requests page](https://github.com/redhat-services-platform-team/org/pulls) and create a pull request from your branch. If you changed `config.yaml`, the pipeline automatically runs a dry-run validation. If the dry run passes, request a review from an org admin.

Use a clear title and include enough context for reviewers:

| Field | Guidance |
|---|---|
| **Title** | `Request: <what you need>` — e.g. `Request: add jsmith to americas-spt` |
| **Body** | GitHub username(s), team or repo affected, and brief justification |

Example body:

```markdown
## Summary
- Add `jsmith` to org members and `americas-spt` team

## Details
- GitHub username: jsmith
- Team: americas-spt
- Reason: New member of the Americas SPT group
```

### 6. Merge and apply

Once approved and merged to `main`, the pipeline applies the changes to the GitHub org. No manual intervention is needed.

## Examples

All examples below are nested under `orgs.redhat-services-platform-team` in `config.yaml`. Indentation in the snippets matches that level.

### Request a new repository

Add an entry under the org-level `repos:` section:

```yaml
    repos:
      my-new-repo:
        description: "Description of the repository"
        default_branch: main
        has_projects: true
        private: true   # set to false for a public repository
```

Then grant a team access under that team's `repos:` section:

```yaml
      my-team:
        repos:
          my-new-repo: write
```

### Request a new team

Add the team under the `teams:` section:

```yaml
      my-new-team:
        description: "What this team is for"
        privacy: closed
        maintainers:
          - team-lead-username
        members:
          - member1
          - member2
        repos:
          some-repo: write
```

If any team members are not yet in the org, also add them to the `members:` list at the org level (in alphabetical order).

### Request to be added to an existing team

Add your GitHub username to the org-level `members:` list (if not already present) and to the relevant team's `members:` list, both in alphabetical order:

```yaml
    members:
      - existinguser
      - your-username    # add here in alphabetical order
      - zuser
```

```yaml
      some-team:
        members:
          - existingmember
          - your-username  # add here in alphabetical order
```

### Request repo permissions for a team

Add the repo under the team's `repos:` section with the desired permission:

```yaml
      my-team:
        repos:
          target-repo: write
```

### Request a nested sub-team (tiered access)

Use nested teams when a group needs elevated permissions on the same repositories. The `naps-sec` team in `config.yaml` follows this pattern:

```yaml
      naps-sec:
        description: "NAPS Security Team"
        privacy: closed
        members:
          - team-member
        repos:
          rhel-stig-content: write
        teams:
          naps-sec-admins:
            description: "NAPS Security Team Admins"
            privacy: closed
            maintainers:
              - team-lead-username
            members: []
            repos:
              rhel-stig-content: admin
```

Members of the parent team get write access; members of the nested sub-team get admin access.

## Config Structure Reference

```yaml
orgs:
  redhat-services-platform-team:
    admins:          # Org administrators (GitHub usernames)
    members:         # Org members (GitHub usernames, alphabetical)
    repos:           # Repository definitions
      repo-name:
        description: "..."
        default_branch: main
        private: true
    teams:           # Team definitions
      team-name:
        description: "..."
        privacy: closed
        maintainers:   # Team maintainers (can manage team membership)
        members:       # Team members
        repos:         # Repos this team has access to
          repo-name: write
        teams:         # Nested sub-teams (optional)
          sub-team-name:
            ...
```

## Troubleshooting

### The pipeline did not run on my pull request

The workflow only triggers when `config.yaml` is in the diff. README-only changes will not start Peribolos. See [When the pipeline runs](#when-the-pipeline-runs) above.

### The pipeline failed (config changes)

Open the **Sync Org Configuration** check on your pull request and expand the failed step log. Common causes:

| Error / symptom | Likely cause | Fix |
|---|---|---|
| Unknown user or 404 for a member | GitHub username is wrong or the account does not exist | Verify the username on GitHub and fix spelling/casing |
| User listed in both `admins` and `members` | Duplicate entry | Remove the user from `members`; admins are implicit members |
| Fewer than 2 admins | `--min-admins=2` enforced by the pipeline | Ensure at least two usernames remain under `admins` |
| Repo not found for team | Repo missing from org-level `repos:` | Add the repo under `orgs.redhat-services-platform-team.repos` first |
| Dry run passes but merge is blocked | Required `peribolos` check missing | Confirm `config.yaml` is in the PR; re-push if needed |

Fix the issue, commit, and push to the same branch. The workflow re-runs automatically.

### Merge blocked despite approval

If reviews are complete but merge is still blocked, check **Required checks** on the pull request. Config changes need a passing `peribolos` check. Documentation-only changes may need an org admin to bypass the missing check after review.

## Important Notes

- **Do not create repositories manually.** `members_can_create_repositories` is set to `false`. All repos must be defined in `config.yaml`.
- **Users cannot appear in both `admins` and `members`.** Admins are implicitly members.
- **Keep lists alphabetically sorted** for consistency and easier review.
- **Use nested teams** for tiered access (e.g., `my-team` for write, `my-team-admins` for admin).
- **Define repos at the org level first.** A team cannot be granted access to a repo that is not listed under the org-level `repos:` section.

## Questions?

Contact an org admin: tech2734 or thaas3.
