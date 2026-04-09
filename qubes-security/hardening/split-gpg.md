GPG keys isolated in sys-pgp, accessed via Qrexec.

##Setup

**sys-pgp:**
- Template: tpl-sys-pgp (debian-13-minimal + gnupg2)
- Network: None (airgapped)
- Storage: Default
- Autostart: No (on-demand)

**Client qubes:**
- Install: `split-gpg2` package
- Config: `~/.config/qubes-split-gpg2/qubes-split-gpg2.conf`

##Usage

**Import key to sys-pgp:**
```bash
#In sys-pgp
gpg --import private-key.asc
gpg --list-secret-keys
```

**Client configuration:**
```bash
#In client qube (e.g., dev)
mkdir -p ~/.config/qubes-split-gpg2
cat > ~/.config/qubes-split-gpg2/qubes-split-gpg2.conf <<EOF
[main]
server = sys-pgp
EOF
```

**Sign commit (Git):**
```bash
#In dev qube
git config --global gpg.program qubes-gpg-client-wrapper
git config --global user.signingkey <KEY_ID>
git commit -S -m "Signed commit"
```

**Encrypt/decrypt file:**
```bash
#Encrypt
gpg -e -r <KEY_ID> file.txt

#Decrypt
gpg -d file.txt.gpg > file.txt
```

**Sign file:**
```bash
gpg --detach-sign file.txt
```

##Qrexec Policy

**Policy:** `/etc/qubes/policy.d/80-sys-pgp.policy`
```
qubes.Gpg2 * @anyvm sys-pgp ask default_target=sys-pgp
qubes.Gpg2 * @anyvm @anyvm deny
```

**Ask prompt:** User confirms each GPG operation in sys-pgp.

##How It Works

1. Client qube runs `qubes-gpg-client-wrapper`
2. Wrapper sends GPG request via Qrexec (`qubes.Gpg2`)
3. sys-pgp receives request, prompts user for confirmation
4. sys-pgp executes GPG operation with local keys
5. Result returned to client qube via Qrexec

**Keys never leave sys-pgp.**

##Key Management

**Generate key in sys-pgp:**
```bash
#In sys-pgp
gpg --full-generate-key
```

**Export public key:**
```bash
#In sys-pgp
gpg --armor --export <KEY_ID> > public-key.asc

#Copy to client qube
qvm-copy public-key.asc
```

**Import public key in client:**
```bash
#In client qube
gpg --import public-key.asc
```

**Backup keys:**
```bash
#In sys-pgp
gpg --armor --export-secret-keys <KEY_ID> > private-key.asc
gpg --armor --export <KEY_ID> > public-key.asc

#Copy to vault or external storage
qvm-copy private-key.asc public-key.asc
```

##Security

**Isolation:**
- GPG keys stored only in sys-pgp
- sys-pgp airgapped (no network)
- Client qubes never see private keys

**User confirmation:**
- Every GPG operation prompts in sys-pgp
- User approves/denies each operation

**Attack surface:**
- Client compromise: Cannot steal keys
- sys-pgp compromise: Keys exposed (critical)

##Troubleshooting

**"No secret key" error:**
- Check `~/.config/qubes-split-gpg2/qubes-split-gpg2.conf` points to sys-pgp
- Verify key exists in sys-pgp: `gpg --list-secret-keys`
- Check Qrexec policy allows client → sys-pgp

**Prompt not appearing:**
- Start sys-pgp manually
- Check sys-pgp running: `qvm-ls | grep sys-pgp`

**Git signing fails:**
- Verify `git config --global gpg.program` is `qubes-gpg-client-wrapper`
- Check `git config --global user.signingkey` matches key ID in sys-pgp