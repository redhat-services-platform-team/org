# org

Red Hat Services Platform Team GitHub Organization Metadata

This repository manages the GitHub organization configuration for **redhat-services-platform-team** using [Peribolos](https://docs.prow.k8s.io/docs/components/cli-tools/peribolos/). All changes to org membership, teams, repositories, and permissions are made through the `config.yaml` file and applied automatically via a GitHub Actions pipeline.

## How It Works

1. You submit a change to `config.yaml` via a pull request.
2. The pipeline runs a **dry-run validation** against the live GitHub org to verify the change is safe.
3. An org admin reviews and approves the pull request.
4. Once merged to `main`, the pipeline runs again with `--confirm` to **apply the changes** to the GitHub org.

## What You Can Request

| Request Type | Where to Edit in `config.yaml` |
|---|---|
| New repository | Add an entry under `repos:` at the org level |
| New team | Add an entry under `teams:` |
| New team member | Add your GitHub username to `members:` (org level) **and** to the relevant team |
| Team repo permissions | Add the repo under the team's `repos:` with a permission level |
| Change repo settings | Modify fields under the repo's entry in `repos:` |

### Permission Levels

| Level | Description |
|---|---|
| `read` | View and clone the repository |
| `triage` | Read plus manage issues and pull requests |
| `write` | Push to the repository and manage issues/PRs |
| `maintain` | Write plus manage repo settings (no destructive actions) |
| `admin` | Full access including settings, secrets, and team management |

## Step-by-Step: Submitting a Change

### 1. Clone the repository

```bash
git clone git@github.com:redhat-services-platform-team/org.git
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

Go to https://github.com/redhat-services-platform-team/org/pulls and create a pull request from your branch. The pipeline will automatically run a dry-run validation. If the dry run passes, request a review from an org admin.

### 6. Merge and apply

Once approved and merged to `main`, the pipeline applies the changes to the GitHub org. No manual intervention is needed.

## Examples

### Request a new repository

Add an entry under the `repos:` section at the org level:

```yaml
    repos:
      my-new-repo:
        description: "Description of the repository"
        default_branch: main
        has_projects: true
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

## Important Notes

- **Do not create repositories manually.** `members_can_create_repositories` is set to `false`. All repos must be defined in `config.yaml`.
- **Users cannot appear in both `admins` and `members`.** Admins are implicitly members.
- **Keep lists alphabetically sorted** for consistency and easier review.
- **Use nested teams** for tiered access (e.g., `my-team` for write, `my-team-admins` for admin).

## Questions?

Contact an org admin: tech2734 or thaas3.
