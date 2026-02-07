# Testing Guide - Initial Deployment

Quick guide for testing the action with annextubetesting.

## Prerequisites

- annextube-action repository: `/home/yoh/proj/annextube-action`
- annextubetesting repository: `/home/yoh/proj/annextubes/annextubetesting`
- Both have workflow files committed
- Local git configured with SSH keys (already done)

## Testing Steps

### 1. Push annextube-action

```bash
cd /home/yoh/proj/annextube-action
git push origin main
```

### 2. Enable GitHub Pages for annextubetesting

**URL**: https://github.com/con/annextubetesting/settings/pages

- **Source**: Deploy from a branch
- **Branch**: `gh-pages`
- **Folder**: `/ (root)`
- Click **Save**

### 3. Push annextubetesting (triggers workflow)

```bash
cd /home/yoh/proj/annextubes/annextubetesting
git push origin master
```

This automatically triggers the deployment workflow!

### 4. Monitor Deployment

**Actions**: https://github.com/con/annextubetesting/actions

- Watch "Deploy to GitHub Pages" workflow
- Should complete in ~3-5 minutes
- Check logs if any errors

### 5. Verify Deployment

**Live site**: https://con.github.io/annextubetesting/

- Thumbnails should be visible
- Videos list appears
- Navigation works
- No console errors

## What Happens

1. Workflow triggers on push to master
2. Checks out repository with full history
3. Action installs Python, Node.js, git-annex, annextube
4. Builds frontend with base path `/annextubetesting/`
5. Creates/updates gh-pages branch
6. Pushes to gh-pages using automatic GITHUB_TOKEN
7. GitHub Pages deploys automatically

## Troubleshooting

### Workflow doesn't trigger

- Check workflow file is in `.github/workflows/`
- Verify changes match path filters (videos/**, playlists/**)
- Try manual trigger via Actions tab → "Run workflow"

### Build fails

- Check Actions logs for specific error
- Ensure `permissions: contents: write` is in workflow
- Verify git-annex installation succeeded

### Page shows 404

- Confirm GitHub Pages is enabled
- Check branch is `gh-pages` not `master`
- Wait a few minutes for deployment

### Broken assets

- Check browser console for errors
- Verify base path in index.html: `/annextubetesting/`
- Confirm all assets exist in gh-pages branch

## Expected Result

After successful deployment:

- ✅ gh-pages branch exists with frontend + data
- ✅ Site accessible at https://con.github.io/annextubetesting/
- ✅ Thumbnails visible (unannexed from previous step)
- ✅ Videos list shows metadata
- ✅ Navigation functional

## No Secrets Required!

The action uses automatic `GITHUB_TOKEN` - no SSH keys or deploy keys needed. Just push and it works!
