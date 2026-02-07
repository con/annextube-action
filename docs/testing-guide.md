# AnnexTube Action Setup Guide

## What We've Built

✅ **`con/annextube-action`** - Reusable GitHub Action
✅ **Workflow in annextubetesting** - `.github/workflows/deploy-ghpages.yml`
✅ **Comprehensive documentation** - README, setup guides

## Next Steps: Testing

### 1. Generate SSH Deploy Keys (Repository-Local)

```bash
# annextubetesting
cd /home/yoh/proj/annextubes/annextubetesting
mkdir -p .git/ssh
ssh-keygen -t ed25519 -C "annextubetesting-deploy" -f .git/ssh/deploy -N ""
git config core.sshCommand "ssh -i $(pwd)/.git/ssh/deploy -F /dev/null"

# annextube-action
cd /home/yoh/proj/annextube-action
mkdir -p .git/ssh
ssh-keygen -t ed25519 -C "annextube-action-deploy" -f .git/ssh/deploy -N ""
git config core.sshCommand "ssh -i $(pwd)/.git/ssh/deploy -F /dev/null"
```

### 2. Add Public Keys as Deploy Keys

**annextubetesting**: https://github.com/con/annextubetesting/settings/keys
```bash
cat /home/yoh/proj/annextubes/annextubetesting/.git/ssh/deploy.pub
```
- Title: `GitHub Actions Deploy`
- ✅ Check "Allow write access"

**annextube-action**: https://github.com/con/annextube-action/settings/keys
```bash
cat /home/yoh/proj/annextube-action/.git/ssh/deploy.pub
```
- Title: `GitHub Actions Deploy`
- ✅ Check "Allow write access"

### 3. Add Private Keys as Secrets

**annextubetesting**: https://github.com/con/annextubetesting/settings/secrets/actions
```bash
cat /home/yoh/proj/annextubes/annextubetesting/.git/ssh/deploy
```
- Name: `DEPLOY_KEY`

**annextube-action**: https://github.com/con/annextube-action/settings/secrets/actions
```bash
cat /home/yoh/proj/annextube-action/.git/ssh/deploy
```
- Name: `DEPLOY_KEY`

### 4. Push annextube-action to GitHub

```bash
cd /home/yoh/proj/annextube-action
git push origin main
```

### 5. Push annextubetesting workflow to GitHub

```bash
cd /home/yoh/proj/annextubes/annextubetesting
git push origin master
```

This will trigger the workflow automatically!

### 6. Monitor Deployment

**URL**: https://github.com/con/annextubetesting/actions

- Watch the "Deploy to GitHub Pages" workflow run
- Should take ~3-5 minutes
- Check for any errors

### 7. Verify Deployment

**URL**: https://con.github.io/annextubetesting/

- Site should load with thumbnails visible
- Check videos list appears
- Verify navigation works

## Expected Behavior

When you push changes to:
- `videos/**`
- `playlists/**`
- `.annextube/**`
- `.gitattributes`
- `authors.tsv`

The workflow will:
1. Checkout repository
2. Install dependencies (Python, Node.js, git-annex, annextube)
3. Build frontend with base path `/annextubetesting/`
4. Create/update gh-pages branch
5. Push to GitHub
6. GitHub Pages deploys automatically

## Repository Status

### annextube-action (con/annextube-action)
```
✅ action.yml - Action definition
✅ README.md - Main documentation
✅ CHANGELOG.md - Version history
✅ docs/setup-deploy-key.md - SSH key setup guide

Status: Ready to push to GitHub
Commit: 69e4463 "Initial release: GitHub Actions for annextube deployment"
```

### annextubetesting (con/annextubetesting)
```
✅ .github/workflows/deploy-ghpages.yml - Deployment workflow
✅ gh-pages branch exists (from manual testing)
✅ Thumbnails unannexed (ready for deployment)

Status: Ready to test automated deployment
Commit: 5facc8a "Add GitHub Actions workflow for automatic Pages deployment"
```

## Troubleshooting

If deployment fails:

1. **Check deploy key has write access**:
   - Go to https://github.com/con/annextubetesting/settings/keys
   - Should show "Read/Write" not "Read-only"

2. **Check secret is named correctly**:
   - Go to https://github.com/con/annextubetesting/settings/secrets/actions
   - Should be exactly `DEPLOY_KEY` (case-sensitive)

3. **Check workflow logs**:
   - Go to https://github.com/con/annextubetesting/actions
   - Click on failed run
   - Check each step for errors

4. **Test SSH key locally**:
   ```bash
   cd /home/yoh/proj/annextubes/annextubetesting
   ssh -i .git/ssh/deploy -T git@github.com
   # Should show: "Hi con/annextubetesting! ..."
   ```

## What Happens Next

After successful deployment:

1. **Automatic updates**: Any push to master with content changes triggers deployment
2. **Manual deployment**: Can trigger via "Run workflow" button in Actions tab
3. **Live site**: https://con.github.io/annextubetesting/ updates automatically
4. **Monitoring**: Check Actions tab for deployment history

## Future Archives

To set up another archive repository:

1. Generate separate SSH key for that repo
2. Add deploy key with write access
3. Add private key as `DEPLOY_KEY` secret
4. Copy workflow file from annextubetesting
5. Enable GitHub Pages in settings

The action is reusable - same workflow file works for all repositories!

---

**Ready to proceed?**

1. Set up SSH keys (steps 1-3)
2. Push both repositories (steps 4-5)
3. Watch the magic happen! ✨
