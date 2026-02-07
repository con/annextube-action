# Setting Up SSH Deploy Keys

This guide walks through creating SSH deploy keys for automatic GitHub Pages deployment.

## Why SSH Deploy Keys?

- **Repository-specific**: Each archive uses its own key
- **No personal tokens**: Doesn't require your personal GitHub access token
- **Secure**: Private key never leaves GitHub Secrets
- **Scoped permissions**: Only has write access to one repository

## Step-by-Step Setup

### 1. Generate SSH Key Pair

Generate a new ED25519 SSH key (recommended):

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/annextube_deploy -N ""
```

This creates two files:
- `~/.ssh/annextube_deploy` (private key) - **Keep this secret!**
- `~/.ssh/annextube_deploy.pub` (public key) - Safe to share

**For multiple repositories**, generate separate keys:

```bash
ssh-keygen -t ed25519 -C "annextubetesting-deploy" -f ~/.ssh/annextubetesting_deploy -N ""
ssh-keygen -t ed25519 -C "myarchive-deploy" -f ~/.ssh/myarchive_deploy -N ""
```

### 2. Add Public Key as Deploy Key

**Go to your repository settings:**

```
https://github.com/YOUR_ORG/YOUR_REPO/settings/keys
```

**Click "Add deploy key"**

**Fill in the form:**
- **Title**: `GitHub Actions Deploy` (or descriptive name)
- **Key**: Paste contents of `~/.ssh/annextube_deploy.pub`
  ```bash
  cat ~/.ssh/annextube_deploy.pub
  ```
- ✅ **Check "Allow write access"** (REQUIRED for deployment)
- Click "Add key"

**Verify it's added:**
- You should see the key listed under "Deploy keys"
- It should show "Read/Write" access

### 3. Add Private Key as Repository Secret

**Go to your repository secrets:**

```
https://github.com/YOUR_ORG/YOUR_REPO/settings/secrets/actions
```

**Click "New repository secret"**

**Fill in the form:**
- **Name**: `DEPLOY_KEY`
- **Value**: Paste entire contents of `~/.ssh/annextube_deploy` (the private key)
  ```bash
  cat ~/.ssh/annextube_deploy
  ```
- Click "Add secret"

**Important notes:**
- Include everything from `-----BEGIN OPENSSH PRIVATE KEY-----` to `-----END OPENSSH PRIVATE KEY-----`
- Do not add quotes or modify the key
- The secret name must match what's in your workflow file

### 4. Clean Up Local Keys (Optional)

After adding to GitHub, you can remove the local keys:

```bash
rm ~/.ssh/annextube_deploy
rm ~/.ssh/annextube_deploy.pub
```

Or keep them in a secure location if you need them later.

### 5. Verify Setup

**Check deploy key:**
```
https://github.com/YOUR_ORG/YOUR_REPO/settings/keys
```
- Should show "Read/Write" access
- Should show recent usage after first deployment

**Check secret:**
```
https://github.com/YOUR_ORG/YOUR_REPO/settings/secrets/actions
```
- Should show `DEPLOY_KEY` listed
- Cannot view value (encrypted by GitHub)

## Testing the Setup

### Test with Workflow Dispatch

1. Go to Actions tab: `https://github.com/YOUR_ORG/YOUR_REPO/actions`
2. Select "Deploy to GitHub Pages" workflow
3. Click "Run workflow" → "Run workflow"
4. Watch the workflow run
5. Check for successful deployment

### Test with Commit

1. Make a change to your archive:
   ```bash
   cd ~/my-archive
   echo "test" > test.txt
   git add test.txt
   git commit -m "Test deployment"
   git push origin master
   ```

2. Go to Actions tab and watch deployment
3. Verify site updates at `https://YOUR_ORG.github.io/YOUR_REPO/`

## Troubleshooting

### Error: "Permission denied (publickey)"

**Cause**: Deploy key not added or doesn't have write access

**Fix:**
1. Verify deploy key is added: `https://github.com/YOUR_ORG/YOUR_REPO/settings/keys`
2. Ensure "Allow write access" is checked
3. Try regenerating and re-adding both keys

### Error: "Host key verification failed"

**Cause**: GitHub's host key not in known_hosts

**Fix**: The action handles this automatically. If you see this error, it's likely a different SSH issue.

### Error: Secret not found

**Cause**: Secret name mismatch

**Fix:**
1. Check workflow file uses: `${{ secrets.DEPLOY_KEY }}`
2. Check secret is named exactly `DEPLOY_KEY` (case-sensitive)
3. Re-add secret with correct name

### Deploy key shows "Read-only"

**Cause**: "Allow write access" checkbox not checked

**Fix:**
1. Delete the existing deploy key
2. Re-add with "Allow write access" checked

## Security Best Practices

### Do:
- ✅ Generate separate keys for each repository
- ✅ Use descriptive names for keys
- ✅ Delete local key files after adding to GitHub
- ✅ Rotate keys periodically (every 6-12 months)
- ✅ Remove unused deploy keys from inactive repositories

### Don't:
- ❌ Share private keys via email, chat, or other channels
- ❌ Commit private keys to repositories
- ❌ Reuse the same key across multiple repositories
- ❌ Use personal SSH keys as deploy keys
- ❌ Store private keys in unencrypted files

## Key Rotation

To rotate a key:

1. Generate new key pair (step 1)
2. Add new public key as deploy key (step 2)
3. Update secret with new private key (step 3)
4. Test deployment with new key
5. Remove old deploy key from repository settings

## Multiple Repositories

For multiple archived repositories:

```bash
# Generate keys for each repo
ssh-keygen -t ed25519 -f ~/.ssh/annextubetesting_deploy -N ""
ssh-keygen -t ed25519 -f ~/.ssh/my-music-archive_deploy -N ""
ssh-keygen -t ed25519 -f ~/.ssh/cooking-videos_deploy -N ""

# Add public key to each repo's deploy keys
# Add private key to each repo's secrets

# Clean up
rm ~/.ssh/*_deploy*
```

## References

- [GitHub Deploy Keys Documentation](https://docs.github.com/en/developers/overview/managing-deploy-keys#deploy-keys)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [SSH Key Generation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
