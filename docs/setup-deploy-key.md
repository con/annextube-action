# SSH Deploy Key Setup

Repository-local SSH keys stored in `.git/ssh/` (not tracked, not in global ~/.ssh).

## annextubetesting

```bash
cd /home/yoh/proj/annextubes/annextubetesting

# Generate key in repository
mkdir -p .git/ssh
ssh-keygen -t ed25519 -C "annextubetesting-deploy" -f .git/ssh/deploy -N ""

# Configure git to use this key
git config core.sshCommand "ssh -i $(pwd)/.git/ssh/deploy -F /dev/null"

# Add public key as deploy key (with write access)
cat .git/ssh/deploy.pub
# → https://github.com/con/annextubetesting/settings/keys
# Title: GitHub Actions Deploy
# ✅ Allow write access

# Add private key as secret
cat .git/ssh/deploy
# → https://github.com/con/annextubetesting/settings/secrets/actions
# Name: DEPLOY_KEY
```

## annextube-action

```bash
cd /home/yoh/proj/annextube-action

# Generate key in repository
mkdir -p .git/ssh
ssh-keygen -t ed25519 -C "annextube-action-deploy" -f .git/ssh/deploy -N ""

# Configure git to use this key
git config core.sshCommand "ssh -i $(pwd)/.git/ssh/deploy -F /dev/null"

# Add public key as deploy key (with write access)
cat .git/ssh/deploy.pub
# → https://github.com/con/annextube-action/settings/keys
# Title: GitHub Actions Deploy
# ✅ Allow write access

# Add private key as secret
cat .git/ssh/deploy
# → https://github.com/con/annextube-action/settings/secrets/actions
# Name: DEPLOY_KEY
```

## Verification

```bash
# Test SSH connection (should show repo name)
cd /home/yoh/proj/annextubes/annextubetesting
ssh -i .git/ssh/deploy -T git@github.com
# Expected: "Hi con/annextubetesting! ..."

cd /home/yoh/proj/annextube-action
ssh -i .git/ssh/deploy -T git@github.com
# Expected: "Hi con/annextube-action! ..."
```

## Notes

- Keys stored in `.git/ssh/` (not tracked by git)
- `git config core.sshCommand` is repository-local (stored in `.git/config`)
- `-F /dev/null` ignores global SSH config
- No changes to `~/.ssh/`
