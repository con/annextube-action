# AnnexTube GitHub Pages Deploy Action

Automatically deploy [annextube](https://github.com/con/annextube) YouTube archives to GitHub Pages.

This action builds the web interface and deploys your archive to GitHub Pages whenever you push updates to your archive repository.

## Features

- ✅ **Automatic deployment** on push to main/master branch
- ✅ **Zero configuration** - uses automatic `GITHUB_TOKEN` (no secrets needed!)
- ✅ **git-annex support** - annexed files remain as symlinks (safe default)
- ✅ **External URL support** - uses annex remote URLs if available (Phase 2)
- ✅ **Configurable** - control what gets deployed
- ✅ **Pinned versions** - no surprise upgrades (opt-in auto-upgrade available)
- ✅ **Fast** - only rebuilds when needed
- ✅ **Secure** - uses built-in GitHub Actions authentication

## Quick Start

### 1. Create Workflow File

Create `.github/workflows/deploy-ghpages.yml` in your repository:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [master, main]
    paths:
      - 'videos/**'
      - 'playlists/**'
      - '.annextube/**'
  workflow_dispatch:

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Need full history for git-annex

      - name: Deploy to GitHub Pages
        uses: con/annextube-action@v1
        # Uses automatic GITHUB_TOKEN - no secrets needed!
```

### 2. Enable GitHub Pages

- Go to `https://github.com/YOUR_ORG/YOUR_REPO/settings/pages`
- **Source**: Deploy from a branch
- **Branch**: `gh-pages`
- **Folder**: `/ (root)`
- Click **Save**

### 3. Push and Deploy

Push a commit to your repository:

```bash
git add .
git commit -m "Update archive"
git push origin master
```

Your site will be deployed automatically and available at:
`https://YOUR_ORG.github.io/YOUR_REPO/`

**No secrets or SSH keys needed!** The action uses the automatic `GITHUB_TOKEN` provided by GitHub Actions.

## Configuration

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `repo-path` | Path to the annextube repository | No | `.` |
| `github-token` | GitHub token for deployment | No | `${{ github.token }}` (automatic) |
| `gh-pages-branch` | Target branch for GitHub Pages | No | `gh-pages` |
| `build-frontend` | Build frontend before deployment | No | `true` |
| `copy-data` | Copy data files to gh-pages | No | `true` |
| `annextube-version` | Version of annextube to install | No | Pinned version |
| `auto-upgrade-frontend` | Auto-upgrade to latest annextube | No | `false` |

### Outputs

| Output | Description |
|--------|-------------|
| `deployment-url` | The URL where the site will be deployed |

### Annexed Files Behavior

By default, **annexed files remain as symlinks** in the gh-pages branch. This is the safe default:

```yaml
- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    copy-data: true  # Copies files as-is (keeps annexing)
```

**To make files directly available (unannexed)**, run `annextube unannex` before deployment:

```yaml
- name: Unannex thumbnails for web viewing
  run: |
    annextube unannex --output-dir . --pattern "videos/*/*/*/thumbnail.jpg"
    git add -A
    git commit -m "Unannex thumbnails for GitHub Pages" || true

- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
```

**External URL support** (Phase 2): When videos are stored on git-annex special remotes (S3, WebDAV), the web interface will automatically use those URLs for playback.

### Advanced Configuration

**Skip frontend build** (if you pre-build elsewhere):

```yaml
- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    build-frontend: false
```

**Deploy metadata only** (for large archives):

```yaml
- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    copy-data: false
```

**Custom branch name**:

```yaml
- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    gh-pages-branch: 'docs'
```

**Use specific annextube version**:

```yaml
- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    annextube-version: 'annextube==0.2.0'
```

## Workflow Triggers

### Deploy on Content Changes

Only rebuild when videos or metadata change:

```yaml
on:
  push:
    branches: [master]
    paths:
      - 'videos/**'
      - 'playlists/**'
      - 'authors.tsv'
```

### Scheduled Deployments

Deploy daily (e.g., after automated archive updates):

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
  workflow_dispatch:      # Allow manual triggers
```

### Manual Deployment

Only deploy when manually triggered:

```yaml
on:
  workflow_dispatch:
```

## Version Management

### Pinned Versions (Default - Recommended)

By default, the action uses a **pinned version** of annextube to avoid unexpected breaking changes:

```yaml
- uses: con/annextube-action@v1  # Action version pinned
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    # annextube-version uses default pinned version
```

**Benefits:**
- ✅ Predictable deployments
- ✅ No surprise upgrades
- ✅ Reproducible builds

### Auto-Upgrade (Opt-in)

Enable automatic frontend upgrades if you want latest features:

```yaml
- uses: con/annextube-action@v1
  with:
    ssh-private-key: ${{ secrets.DEPLOY_KEY }}
    annextube-version: 'latest'
    auto-upgrade-frontend: true
```

**⚠️ Warning:** May introduce breaking changes. Test in a separate branch first.

### Dependabot for Version Upgrades

Recommended: Use Dependabot to automatically create PRs for action version upgrades.

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
    labels:
      - "dependencies"
      - "github-actions"
```

This creates PRs when new action versions are released, allowing you to review and test before merging.

## How It Works

1. **Checkout**: Fetches your repository with full history
2. **Setup**: Installs Python, Node.js, git-annex, and annextube (pinned version)
3. **Build**: Builds the frontend with correct base path
4. **Deploy**: Creates/updates gh-pages branch with built assets
   - **Annexed files remain as symlinks** (safe default)
   - External annex URLs used if available (Phase 2)
5. **Push**: Pushes to gh-pages using SSH deploy key
6. **Publish**: GitHub Pages automatically serves the new content

## Troubleshooting

### Deployment fails with "Permission denied"

- Ensure `permissions: contents: write` is set in workflow
- Check that GITHUB_TOKEN has not been disabled in repository settings

### Site shows 404 or broken assets

- Check that GitHub Pages is enabled in repository settings
- Verify branch is set to `gh-pages` and folder is `/ (root)`
- Wait a few minutes for GitHub Pages to deploy

### Build fails with "Frontend directory not found"

- Ensure your workflow checks out the repository with `fetch-depth: 0`
- The action needs access to the annextube source for frontend building

### git-annex errors in workflow

- The action installs git-annex automatically
- For private remotes, you may need to configure credentials

## Examples

See [annextubetesting](https://github.com/con/annextubetesting) for a working example.

## Security

- **No secrets required** - uses automatic `GITHUB_TOKEN`
- **Scoped permissions** - token only has access to the repository it runs in
- **Automatic expiration** - token expires after workflow completes
- **Read-only source access** - action only writes to gh-pages branch
- **No credential storage** - nothing to manage or rotate

## License

MIT License - see [annextube license](https://github.com/con/annextube/blob/master/LICENSE)

## Related

- [annextube](https://github.com/con/annextube) - YouTube archival system
- [GitHub Pages documentation](https://docs.github.com/en/pages)
- [GitHub Actions documentation](https://docs.github.com/en/actions)
