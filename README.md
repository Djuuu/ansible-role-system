Ansible Role: System
====================

Miscellaneous system init stuff (base configuration, packages, users, ...).

Requirements
------------

Base Linux distribution.

Some tasks or configuration are specific to Debian and variants.

Role Variables
--------------

Base settings:

```yaml
#system_hostname: myhost

system_timezone: UTC
```

Users:

```yaml
system_users: []
#  - name: alice
#  - name: bob
#    group: users
#  - name: carl
#    groups: [ sudo, docker ]
#    home: /home/coral
#    shell: /bin/zsh
#    generate_ssh_key: true

system_users_deleted: []
#  - name: daniel
#  - elmo
```

Groups:

```yaml
system_groups: []
#  - name: users
#  - name: admins
#    gid: 987
#    system: true
```

System packages (ex: apt):

```yaml
system_packages: []
#  - fail2ban
#  - firewalld
#  - dnsutils
#  - net-tools
```

OPKG packages:

```yaml
system_packages_opkg: []
#  - git
#  - git-http
#  - grep
#  - htop
```

Python pip packages:

```yaml
system_packages_pip: []
#  - python-dateutil
#  - requests

# Use pip --break-system-packages option
system_packages_pip_break_system_packages: false
```

Go / Go packages:  
(Go will be installed if a version is provided, default is empty)

```yaml
system_install_go_version:
system_packages_go: []
#  - github.com/jesseduffield/lazygit@latest
#  - github.com/jesseduffield/lazydocker@latest
```

Additional roles can be included when installing packages:
```yaml
system_required_roles:
#  - geerlingguy.docker
```

Available configuration tasks (see related variables below):

```yaml
system_configure: []
#  - ssh       # tasks/configure/ssh.yml
#  - firewalld # tasks/configure/firewalld.yml
#  - fail2ban  # tasks/configure/fail2ban.yml
#  - journald  # tasks/configure/journald.yml
```

SSH configuration (used when `system_configure` contains `ssh`):

```yaml
# Optional additional custom port
system_ssh_custom_port:
# Allow password authentication
system_ssh_password_authentication: false
# Custom sshd configuration
system_ssh_custom_config:
#  - file: 99-custom.conf
#    content: |
#      # Fix SSH freezing on Wifi
#      IPQoS cs0 cs0
#      #IPQoS throughput
```

Fail2ban configuration (used when `system_configure` contains `fail2ban`):

```yaml
# Default fail2ban bantime
system_fail2ban_jail_bantime: 1h
# Default fail2ban maxretry
system_fail2ban_jail_maxretry: 10
# Default fail2ban local configuration
system_fail2ban_jail_local: |
  [DEFAULT]

  # "bantime" is the number of seconds that a host is banned.
  bantime  = {{ system_fail2ban_jail_bantime }}

  # "maxretry" is the number of failures before a host get banned.
  maxretry = {{ system_fail2ban_jail_maxretry }}

  # https://github.com/fail2ban/fail2ban/issues/3292
  # Debian 12 has no log files, just journalctl
  backend = systemd

  # Destination email address used solely for the interpolations in
  # jail.{conf,local} configuration files.
  destemail = root@localhost

  # Sender email address used solely for some actions
  sender = root@<fq-hostname>

  [sshd]
  enabled = true
  mode    = ddos
  port    = ssh{{
  system_ssh_custom_port | default('') | ternary(
  ',' ~ system_ssh_custom_port, ''
  ) }}

  [selinux-ssh]
  enabled = true
  port    = ssh{{
  system_ssh_custom_port | default('') | ternary(
  ',' ~ system_ssh_custom_port, ''
  ) }}
```

Journald configuration (used when `system_configure` contains `journald`):

```yaml
system_journald: {}
#  # Defaults:
#  Storage:              auto
#  Compress:             yes
#  Seal:                 yes
#  SplitMode:            uid
#  SyncIntervalSec:      5m
#  RateLimitIntervalSec: 30s
#  RateLimitBurst:       10000
#  SystemMaxUse:
#  SystemKeepFree:
#  SystemMaxFileSize:
#  SystemMaxFiles:       100
#  RuntimeMaxUse:
#  RuntimeKeepFree:
#  RuntimeMaxFileSize:
#  RuntimeMaxFiles:      100
#  MaxRetentionSec:
#  MaxFileSec:           1month
#  ForwardToSyslog:      yes
#  ForwardToKMsg:        no
#  ForwardToConsole:     no
#  ForwardToWall:        yes
#  TTYPath:              /dev/console
#  MaxLevelStore:        debug
#  MaxLevelSyslog:       debug
#  MaxLevelKMsg:         notice
#  MaxLevelConsole:      info
#  MaxLevelWall:         emerg
#  LineMax:              48K
#  ReadKMsg:             yes
#  Audit:                no
```

Mount configuration

```yaml
# Mount options
system_mounts: []
#  - src: "//my-server/example"
#    path: "/mount/example"
#    fstype: cifs
#    state: mounted
#    opts: "uid=1234,gid=123,file_mode=0664,dir_mode=0775,rw,nodev,nosuid,noexec,credentials=/home/admin/.credentials,iocharset=utf8"
#    opts_no_log: true

# Mount credentials files
system_mount_credential_files: []
#  - dest: /home/admin/.credentials
#    content: |
#      username=me
#      password=s3cr3t
```

Additional config files
-----------------------

Firewalld can be configured through files put in the following locations (relative to your playbook):

- `config/system/{{ inventory_hostname }}/firewalld/ipsets` 
- `config/system/{{ inventory_hostname }}/firewalld/services` 
- `config/system/{{ inventory_hostname }}/firewalld/zones`

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: all
  roles:
    - djuuu.system
```

Or run individual tasks:

```yaml
- hosts: all
  tasks:

    - name: Set hostname
      import_role: { name: djuuu.system, tasks_from: set-hostname }

    - name: Set timezone
      import_role: { name: djuuu.system, tasks_from: set-timezone }
        
    - name: Init system packages
      import_role: { name: djuuu.system, tasks_from: init-packages }

    - name: Init system users
      import_role: { name: djuuu.system, tasks_from: init-users }
```

License
-------

Beerware License
