#Split-Git

Git repositories isolated in sys-git, accessed via Qrexec.

##Setup

**sys-git:**
- Template: tpl-sys-git (debian-13-minimal + git)
- Network: None (airgapped)
- Storage: 20GB (Git repositories)
- Autostart: Yes

**Client qubes:**
- Install: `git`, `git-remote-qrexec` package
- Config: Git uses `qrexec://` URLs

##Usage

**Initialize repository in sys-git:**
```bash
#In client qube (e.g., dev)
git init-qrexec @default/my-repo
```

**Clone repository:**
```bash
#In client qube
git clone qrexec://@default/my-repo
cd my-repo
```

**Push changes:**
```bash
#In client qube
git add .
git commit -m "Update"
git push
```

**Pull changes:**
```bash
#In client qube
git pull
```

##Qrexec Policy

**Policy:** `/etc/qubes/policy.d/80-sys-git.policy`
```
GitFetch * @anyvm @default ask target=sys-git default_target=sys-git
GitPush  * @anyvm @default ask target=sys-git default_target=sys-git
GitInit  * @anyvm @default ask target=sys-git default_target=sys-git
GitFetch * @anyvm @anyvm deny
GitPush  * @anyvm @anyvm deny
GitInit  * @anyvm @anyvm deny
```

**Ask prompt:** User confirms each Git operation in sys-git.

##How It Works

**Clone:**
1. Client runs `git clone qrexec://@default/my-repo`
2. Git calls `git-remote-qrexec`
3. `git-remote-qrexec` sends Qrexec request (`GitFetch`)
4. sys-git receives request, prompts user
5. sys-git runs `git-upload-pack` on bare repo
6. Objects transferred to client via Qrexec

**Push:**
1. Client runs `git push`
2. Git calls `git-remote-qrexec`
3. `git-remote-qrexec` sends Qrexec request (`GitPush`)
4. sys-git receives request, prompts user
5. sys-git runs `git-receive-pack` on bare repo
6. Objects transferred to sys-git via Qrexec

**Init:**
1. Client runs `git init-qrexec @default/my-repo`
2. Sends Qrexec request (`GitInit`)
3. sys-git creates bare repo: `~/src/my-repo.git`

**Repositories never leave sys-git.**

##Repository Management

**List repositories in sys-git:**
```bash
#In sys-git
ls ~/src/*.git
```

**Manual repository creation:**
```bash
#In sys-git
mkdir ~/src/my-repo.git
cd ~/src/my-repo.git
git init --bare
```

**Backup repositories:**
```bash
#In sys-git
tar czf repos-backup.tar.gz ~/src/*.git

#Copy to vault or external storage
qvm-copy repos-backup.tar.gz
```

**Restore repository:**
```bash
#In sys-git
tar xzf repos-backup.tar.gz -C ~/
```

##Multiple Remotes

**Use sys-git + external remote:**
```bash
#In client qube
git clone qrexec://@default/my-repo
cd my-repo

#Add external remote (via sys-ssh)
git remote add github git@github.com:user/repo.git

#Push to sys-git
git push origin main

#Push to GitHub (via sys-ssh)
git push github main
```

**Workflow:**
1. All commits pushed to sys-git (airgapped backup)
2. Selective push to external remote (GitHub, etc.)
3. sys-git always has full history

##Security

**Isolation:**
- Git repositories stored only in sys-git
- sys-git airgapped (no network)
- Client qubes have working copy only

**User confirmation:**
- Every Git operation prompts in sys-git
- User approves/denies each fetch/push/init

**Attack surface:**
- Client compromise: Cannot access other repos
- sys-git compromise: All repos exposed (critical)

##Troubleshooting

**"fatal: unable to access 'qrexec://@default/my-repo'":**
- Check sys-git running: `qvm-ls | grep sys-git`
- Check repository exists in sys-git: `ls ~/src/my-repo.git`
- Check Qrexec policy allows client → sys-git

**"fatal: could not read from remote repository":**
- Verify repository initialized: `git init-qrexec @default/my-repo`
- Check sys-git prompt appeared and was approved
- Check repository permissions in sys-git

**Push rejected:**
- Check repository not bare in client (should be normal Git repo)
- Check sys-git repository is bare: `~/src/my-repo.git`