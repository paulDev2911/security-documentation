#Salt Structure

Salt-based automation for Qubes OS infrastructure. All states are organized by project (qube/template) with consistent patterns.

##Directory Layout

```
salt/
├── browser/              #Browser qube (Brave, Mullvad, Chromium)
├── debian/               #Debian base template
├── debian-minimal/       #Debian minimal template
├── debian-xfce/          #Debian XFCE template
├── dev/                  #Development qube
├── fedora/               #Fedora base template
├── fedora-minimal/       #Fedora minimal template
├── fedora-xfce/          #Fedora XFCE template
├── learning/             #Learning qube (OnlyOffice)
├── mgmt/                 #Management DispVM
├── sys-firewall/         #Firewall qube
├── sys-git/              #Git server (Qrexec)
├── sys-mullvad/          #Mullvad VPN qube
├── sys-net/              #Network qube
├── sys-pgp/              #GPG server (Split-GPG2)
├── sys-ssh-agent/        #SSH agent server (Qrexec)
├── sys-ssh/              #SSH server (Qrexec)
├── sys-usb/              #USB qube
├── vault/                #Vault qube (KeePassXC)
└── utils/                #Shared macros and tools
```

##State File Patterns

Each project follows this structure:

```
project/
├── clone.sls             #Clone template from base
├── clone.top             #Top file for clone state
├── create.sls            #Create qube(s) in dom0
├── create.top            #Top file for create state
├── install.sls           #Install packages in template
├── install.top           #Top file for install state
├── configure.sls         #Configure qube (AppVM)
├── configure.top         #Top file for configure state
├── init.top              #Combined top file (all states)
├── files/                #Static files
│   ├── admin/            #Dom0 files (policies, scripts)
│   │   ├── policy/       #Qrexec policies
│   │   └── bin/          #Dom0 scripts
│   ├── server/           #Server-side files (AppVM)
│   │   ├── rpc/          #Qrexec RPC services
│   │   ├── systemd/      #Systemd units/overrides
│   │   └── qubes-firewall.d/  #Firewall scripts
│   └── client/           #Client-side files
│       ├── systemd/      #Client systemd units
│       └── git-core/     #Git helpers (sys-git)
└── template.jinja        #Template variables
```

##State Types

**clone.sls:**
- Runs in: dom0
- Purpose: Clone template from base (debian-minimal, fedora-minimal, etc.)
- Macro: `clone_template('debian-minimal', sls_path)`
- Example: `sys-git/clone.sls` clones debian-minimal → tpl-sys-git

**create.sls:**
- Runs in: dom0
- Purpose: Create qubes (AppVM, DispVM, templates)
- Uses: `qvm/template.jinja` load macro
- Sets: prefs, features, tags, policies
- Example: `sys-git/create.sls` creates sys-git qube + policy

**install.sls:**
- Runs in: Template VM
- Purpose: Install packages, files, systemd units
- Condition: `{% if grains['nodename'] != 'dom0' -%}`
- Example: `sys-git/install.sls` installs git, qrexec RPC services

**configure.sls:**
- Runs in: AppVM
- Purpose: User-level configuration, files in /home/user
- Condition: `{% if grains['nodename'] != 'dom0' -%}`
- Example: `sys-git/configure.sls` creates ~/.ssh/identities.d

**init.top:**
- Combined top file including all states
- Simplifies deployment: `qubesctl top.enable project`

##Top Files

Top files map states to targets using matchers:

**Matchers:**
- `'dom0'` + `match: nodegroup` → Run in dom0
- `'tpl-sys-git'` + `match: list` → Specific template
- `'sys-git'` + `match: nodegroup` → Specific qube
- `'I@qubes:type:template and E@^fedora-[0-9][0-9]-minimal$'` + `match: compound` → Regex + grains

**Example (sys-git/init.top):**
```yaml
base:
  'dom0':
    - match: nodegroup
    - sys-git.create
  'tpl-sys-git':
    - sys-git.install
  'sys-git':
    - sys-git.configure
```

##Template Variables (template.jinja)

Templates use Jinja to define naming:

```jinja
{% set base = 'fedora' -%}
{% set version = salt['pillar.get']('qvm:fedora:version', '42') -%}
{% set flavor = 'minimal' -%}
{% set repo = salt['pillar.get']('qvm:fedora:repo', 'qubes-templates-itl') -%}
{% set template = base ~ '-' ~ version ~ '-' ~ flavor -%}
{% set template_clean = base ~ '-' ~ flavor -%}
```

Used in states:
```yaml
"{{ template.template }}-template-installed":
  qvm.template_installed:
    - name: {{ template.template }}
    - fromrepo: {{ template.repo }}
```

##Macros

Located in `utils/macros/`, reusable across projects:

**clone-template.sls:**
```jinja
{% from 'utils/macros/clone-template.sls' import clone_template -%}
{{ clone_template('debian-minimal', sls_path) }}
```
- Clones base template → tpl-<project>
- Handles template updates before clone
- Optional: include create state

**install-repo.sls:**
```jinja
{% from 'utils/macros/install-repo.sls' import install_repo -%}
{{ install_repo('browser', 'brave') }}
```
- Installs repository keyring + sources
- Debian: `/etc/apt/sources.list.d/brave.sources` + `/usr/share/keyrings/brave.asc`
- Fedora: `/etc/yum.repos.d/brave.repo` + `/etc/pki/rpm-gpg/RPM-GPG-KEY-brave`

**policy.sls:**
```jinja
{% from 'utils/macros/policy.sls' import policy_set with context -%}
{{ policy_set(sls_path, '80') }}
```
- Deploys policy: `/etc/qubes/policy.d/80-<project>.policy`
- Source: `salt://<project>/files/admin/policy/default.policy`
- Context: `sls_path` variable available in policy template

**sync-appmenus.sls:**
```jinja
{% from 'utils/macros/sync-appmenus.sls' import sync_appmenus -%}
{{ sync_appmenus('tpl-browser') }}
```
- Starts qube, syncs appmenus, optionally shuts down
- Runs as GUI user (from `dom0/gui-user.jinja`)

**update-admin.sls:**
```jinja
{% from 'utils/macros/update-admin.sls' import update_admin -%}
{{ update_admin('fedora-minimal', 'tpl-sys-git') }}
```
- Updates template via `qubes-vm-update`
- Forces shutdown after update (important for clones)

##Qube Creation (qvm/template.jinja)

Standardized qube creation using `load` macro:

```jinja
{%- from "qvm/template.jinja" import load -%}

{% load_yaml as defaults -%}
name: sys-git
force: True
present:
- template: tpl-sys-git
- label: gray
prefs:
- netvm: ""
- audiovm: ""
- memory: 200
- maxmem: 300
features:
- enable:
  - servicevm
- disable:
  - service.cups
{%- endload %}
{{ load(defaults) }}
```

**Common settings:**
- `force: True` → Recreate if exists
- `present:` → Qube properties (template, label, class)
- `prefs:` → Qube preferences
- `features:` → Qubes features (enable/disable/set)
- `tags:` → Qube tags (add/del)

##Project-Specific States

**sys-git:**
- `install-client.sls` → Install Git Qrexec client in any template
- Files: `files/client/git-core/` (git-remote-qrexec, git-init-qrexec)
- Policy argument registration for repository names

**sys-ssh-agent:**
- `install-client.sls` → Install SSH agent forwarder client
- Files: `files/client/systemd/ssh-agent-forwarder@.service`
- Socket: `/tmp/ssh-agent-forwarder/<AGENT>.sock`

**sys-mullvad:**
- `configure.sls` → Install WireGuard config template
- `files/server/rc.local.j2` → Auto-start WireGuard on boot
- `files/server/qubes-firewall.d/` → DNS redirection, NAT rules
- `files/admin/bin/qvm-mullvad` → Helper script for config deployment

**dev:**
- `install-common.sls` → Base development tools
- `install-vscodium.sls` → VSCodium from external repo
- `install-dotnet-tools.sls` → .NET tooling
- `install-rider.sls` → JetBrains Toolbox
- `home-cleanup.sls` → Remove unused ~/Desktop, ~/Documents, etc.

**browser:**
- `install-brave.sls`, `install-mullvad.sls`, `install-chromium.sls` → Separate install states
- `appmenus.sls` → Sync appmenus after installation
- Creates: tpl-browser, dvm-browser, disp-browser, browser (AppVM)

##Utils Tools

**utils/tools/common/update.sls:**
- Shared by all install states
- Runs `pkg.uptodate` with refresh

**utils/tools/zsh/:**
- `install.sls` → Install zsh, zsh-autosuggestions, zsh-syntax-highlighting
- `change-shell.sls` → Change user shell to zsh
- `touch-zshrc.sls` → Prevent zsh first-run prompt

**utils/tools/builder/:**
- `core.sls` → Build tools (make, rpm, dpkg-dev, etc.)
- `doc.sls` → Documentation tools (pandoc, ronn, groff)

**utils/tools/helpers/:**
- Helper scripts: `run-terminal`, `run-browser`, `run-file-manager`, `run-mail`
- Fallback wrapper for missing terminal emulators

**utils/tools/xfce/:**
- XFCE preferred applications config
- Points to helper scripts (run-terminal, etc.)

##Deployment Workflow

**1. Initial setup (dom0):**
```sh
cd ~/salt-qubes-sec
sudo ./scripts/setup.sh
```
- Copies salt/ → /srv/user_salt/salt-qubes-sec
- Copies minion.d/*.conf → /etc/salt/minion.d/

**2. Development (dev qube):**
```sh
#Edit states in dev qube
git add -A
git commit -m "Update sys-git install"
git push qrexec://@default/salt-qubes-sec
```

**3. Sync to dom0:**
```sh
#In dom0
./scripts/sync-from-dev.sh dev
sudo ./scripts/setup.sh
```

**4. Apply states:**
```sh
#Via top file
sudo qubesctl top.enable sys-git
sudo qubesctl --targets=tpl-sys-git,sys-git state.apply
sudo qubesctl top.disable sys-git

#Direct state application
sudo qubesctl state.apply sys-git.create
sudo qubesctl --skip-dom0 --targets=tpl-sys-git state.apply sys-git.install
sudo qubesctl --skip-dom0 --targets=sys-git state.apply sys-git.configure
```

##Common Patterns

**Conditional execution (templates only):**
```jinja
{% if grains['nodename'] != 'dom0' -%}
#Install packages
{% endif -%}
```

**OS-specific packages:**
```jinja
{% set pkg = {
    'Debian': {
      'pkg': ['openssh-client'],
    },
    'RedHat': {
      'pkg': ['openssh-clients'],
    },
}.get(grains.os_family) -%}

"project-installed-os-specific":
  pkg.installed:
    - pkgs: {{ pkg.pkg|sequence|yaml }}
```

**Include dependencies:**
```jinja
include:
  - utils.tools.common.update
  - sys-pgp.install-client

"project-installed":
  pkg.installed:
    - require:
      - sls: utils.tools.common.update
```

**File deployment:**
```jinja
"project-rpc-service":
  file.managed:
    - name: /etc/qubes-rpc/Service
    - source: salt://project/files/server/rpc/Service
    - mode: '0755'
    - user: root
    - group: root
```

**Systemd service management:**
```jinja
"project-enable-service":
  service.enabled:
    - name: wg-quick@wireguard

"project-restart-service":
  cmd.run:
    - name: systemctl restart NetworkManager
    - onchanges:
      - file: project-nm-config
```

##Policy Files

Policies in `files/admin/policy/default.policy`:

```
##Do not modify this file, create a new policy with with a lower number in the
##file name instead. For example `30-user.policy`.
GitFetch * @anyvm @default ask target=sys-git default_target=sys-git
GitPush  * @anyvm @default ask target=sys-git default_target=sys-git
GitFetch * @anyvm @anyvm deny
GitPush  * @anyvm @anyvm deny
##vim:ft=qrexecpolicy
```

Variables available in templates:
- `{{ sls_path }}` → Project name (sys-git, sys-ssh, etc.)

##Debugging

**Check which states apply:**
```sh
sudo qubesctl state.show_top
```

**Test single state:**
```sh
sudo qubesctl state.apply sys-git.create test=True
```

**Check grains:**
```sh
sudo qubesctl --skip-dom0 --targets=tpl-sys-git grains.items
```

**Verbose output:**
```sh
sudo qubesctl --skip-dom0 --targets=tpl-sys-git state.apply sys-git.install -l debug
```

##Best Practices

**Package installation:**
- Always include `utils.tools.common.update` dependency
- Use `install_recommends: False` and `skip_suggestions: True`
- Fedora: Add `setopt: "install_weak_deps=False"`

**File permissions:**
- Dom0 policies: mode '0664', user root, group qubes
- RPC services: mode '0755', user root, group root
- User files: mode '0644' or '0600', user user, group user

**Template management:**
- Always shutdown template after `update_admin` macro
- Use `sync_appmenus` after package installation with .desktop files
- Set `include_in_backups: False` for templates

**Qube features:**
- Infrastructure: `servicevm` enabled
- Templates: Disable cups, cups-browsed, meminfo-writer
- Network qubes: Enable service.qubes-updates-proxy

**Naming conventions:**
- Template: `tpl-<project>`
- DispVM template: `dvm-<project>`
- DispVM: `disp-<project>`
- AppVM: `<project>`