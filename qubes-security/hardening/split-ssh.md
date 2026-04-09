#Split-SSH

SSH keys isolated in sys-ssh-agent, accessed via Qrexec. Multiple agents supported.

##Setup

**sys-ssh-agent:**
- Template: tpl-sys-ssh-agent (debian-13-minimal + openssh-server)
- Network: None (airgapped)
- Storage: 40GB (SSH keys)
- Autostart: No (on-demand)

**Client qubes:**
- Install: `ssh-agent-forwarder` package
- Systemd: `ssh-agent-forwarder@<AGENT>.service`

##Architecture

**Multi-agent support:**
- One SSH agent per identity/domain (e.g., `work`, `personal`, `github`)
- Each agent runs independently in sys-ssh-agent
- Clients connect to specific agent via Qrexec

**Directory structure (sys-ssh-agent):**
```
~/.ssh/identities.d/
├── work/
│   ├── id_ed25519
│   └── id_ed25519.pub
├── personal/
│   ├── id_ed25519
│   └── id_ed25519.pub
└── github/
    ├── id_ed25519
    └── id_ed25519.pub
```

##Usage

**Generate SSH key in sys-ssh-agent:**
```bash
#In sys-ssh-agent
mkdir -p ~/.ssh/identities.d/work
ssh-keygen -t ed25519 -f ~/.ssh/identities.d/work/id_ed25519 -C "work@example.com"
```

**Start agent forwarder in client:**
```bash
#In client qube (e.g., dev)
systemctl --user start ssh-agent-forwarder@work.service
```

**Use SSH agent:**
```bash
#In client qube
export SSH_AUTH_SOCK=/tmp/ssh-agent-forwarder/work.sock
ssh-add -l  #List keys from sys-ssh-agent
ssh user@example.com  #Uses key from sys-ssh-agent
```

**Auto-start agent on login:**
```bash
#In client qube
systemctl --user enable ssh-agent-forwarder@work.service

#Add to ~/.bashrc or ~/.zshrc
export SSH_AUTH_SOCK=/tmp/ssh-agent-forwarder/work.sock
```

##Qrexec Policy

**Policy:** `/etc/qubes/policy.d/80-sys-ssh-agent.policy`
```
SshAgent+work * @anyvm sys-ssh-agent ask default_target=sys-ssh-agent
SshAgent+personal * @anyvm sys-ssh-agent ask default_target=sys-ssh-agent
SshAgent+github * @anyvm sys-ssh-agent ask default_target=sys-ssh-agent
SshAgent+* * @anyvm @anyvm deny
```

**Ask prompt:** User confirms SSH agent access in sys-ssh-agent.

##How It Works

1. Client starts `ssh-agent-forwarder@work.service`
2. Service connects to sys-ssh-agent via Qrexec (`SshAgent+work`)
3. sys-ssh-agent starts `ssh-agent` for `work` identity
4. sys-ssh-agent loads keys from `~/.ssh/identities.d/work/`
5. Client socket: `/tmp/ssh-agent-forwarder/work.sock`
6. SSH client uses socket to sign with keys in sys-ssh-agent

**Keys never leave sys-ssh-agent.**

##Key Management

**List keys in agent:**
```bash
#In client qube
export SSH_AUTH_SOCK=/tmp/ssh-agent-forwarder/work.sock
ssh-add -l
```

**Export public key:**
```bash
#In sys-ssh-agent
cat ~/.ssh/identities.d/work/id_ed25519.pub

#Copy to client qube or external service (GitHub, etc.)
```

**Backup keys:**
```bash
#In sys-ssh-agent
tar czf ssh-keys-backup.tar.gz ~/.ssh/identities.d/

#Copy to vault or external storage
qvm-copy ssh-keys-backup.tar.gz
```

##Multiple Agents Example

**Scenario:** Work SSH keys vs. Personal SSH keys

**sys-ssh-agent setup:**
```bash
#In sys-ssh-agent
mkdir -p ~/.ssh/identities.d/{work,personal}
ssh-keygen -t ed25519 -f ~/.ssh/identities.d/work/id_ed25519
ssh-keygen -t ed25519 -f ~/.ssh/identities.d/personal/id_ed25519
```

**Client usage:**
```bash
#In dev qube (work SSH)
systemctl --user start ssh-agent-forwarder@work.service
export SSH_AUTH_SOCK=/tmp/ssh-agent-forwarder/work.sock
ssh git@work-git-server.com

#In dev qube (personal SSH)
systemctl --user start ssh-agent-forwarder@personal.service
export SSH_AUTH_SOCK=/tmp/ssh-agent-forwarder/personal.sock
ssh git@github.com
```

**Benefit:** Work/personal keys never mixed, separate confirmation prompts.

##Integration with sys-ssh

**sys-ssh uses sys-ssh-agent:**

Client config (`~/.ssh/config` in dev):
```
Host example
    HostName localhost
    Port 1840
    ProxyCommand qrexec-client-vm -- sys-ssh ConnectTCP+example.com+22
    IdentityAgent /tmp/ssh-agent-forwarder/work.sock
```

**Flow:**
1. Client runs `ssh example`
2. SSH connects via sys-ssh (Qrexec proxy)
3. SSH uses agent from sys-ssh-agent (via `IdentityAgent`)
4. sys-ssh-agent signs authentication challenge
5. Connection established via sys-ssh → VPN

##Security

**Isolation:**
- SSH keys stored only in sys-ssh-agent
- sys-ssh-agent airgapped (no network)
- Client qubes never see private keys

**User confirmation:**
- Every agent access prompts in sys-ssh-agent
- User approves/denies each connection

**Attack surface:**
- Client compromise: Cannot steal keys
- sys-ssh-agent compromise: Keys exposed (critical)

##Troubleshooting

**"Could not open a connection to your authentication agent":**
- Check service running: `systemctl --user status ssh-agent-forwarder@work.service`
- Check socket exists: `ls /tmp/ssh-agent-forwarder/work.sock`
- Start sys-ssh-agent if not running

**"Permission denied (publickey)":**
- Verify public key added to remote server
- Check key loaded: `ssh-add -l`
- Verify `SSH_AUTH_SOCK` points to correct socket

**Agent not starting:**
- Check Qrexec policy allows client → sys-ssh-agent
- Check key exists in `~/.ssh/identities.d/<AGENT>/`
- Check sys-ssh-agent running