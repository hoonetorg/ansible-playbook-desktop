# ansible-playbook-desktop

Sets up desktop/notebook hosts (group `desktop`) with these roles, in this order:

| role | tag | purpose |
|------|-----|---------|
| ansible-role-repo | repo | additional package repositories, distribution non-free repos |
| ansible-role-system | system | system-wide settings: vim never writes viminfo (/etc/vimrc) |
| ansible-role-hardware | hardware | firmware, drivers, power management, hardware quirks, VM guest agents |
| ansible-role-disk | disk | partitions, LUKS, btrfs filesystems/subvolumes, crypttab/fstab |
| ansible-role-virt | virt | KVM/libvirt host with virt-manager, libvirt networks |
| ansible-role-network | network | NetworkManager LAN/Wi-Fi profiles, same MAC, firewalld default zone and zone services (the only firewalld writer) |
| ansible-role-ssh | ssh | SSH host keys from the vault (same keys after a reinstall), /etc/ssh/ssh_known_hosts from the inventory, temporary VMs in ~/.ssh/known_hosts_vms |
| ansible-role-user | user | groups, users, home directories |
| ansible-role-nfsserver | nfsserver | NFSv4 server, exports |
| ansible-role-syncthing | syncthing | syncthing per user, devices/folders via REST API |
| ansible-role-citrix | citrix | Citrix Workspace app (ICAClient), RPM downloaded on the controller |
| ansible-role-software | software | packages, browser policies and extensions, web apps (sites as app windows in own browser profiles) |
| ansible-role-ksuite | ksuite | Infomaniak desktop apps kDrive, kChat, kMeet (AppImages in /opt/ksuite) |
| ansible-role-gnome | gnome | GNOME settings for all users (dconf system defaults and locks) |
| ansible-role-btrbk | btrbk | btrbk snapshots/backups with own systemd units |

Each role reads one dict named after its tag (`repo`, `hardware`, `disk`, …) from the inventory; see the role
READMEs.

## Usage

```bash
ansible-playbook --diff -i <inventory> -k -K --ask-vault-password site.yml -l <host>
```

Only some roles (the tag is on the role in `site.yml`, so all its tasks run, also the one that builds
`<role>_final`; fact gathering always runs):

```bash
ansible-playbook ... site.yml -l <host> --tags gnome
ansible-playbook ... site.yml -l <host> --tags software,gnome
ansible-playbook ... site.yml -l <host> --skip-tags disk
ansible-playbook site.yml --list-tags                  # all tags
ansible-playbook site.yml --tags gnome --list-tasks    # what would run
```

Roles depend on earlier ones (see the order above, e.g. nfsserver needs user and virt): a new host always gets
a full run first.

zypper waits up to 5 minutes for the zypp lock (`ZYPP_LOCK_TIMEOUT`, play environment), e.g. while
packagekitd (GNOME Software) refreshes in the background, instead of failing immediately.

## Secrets in output and logs

Task results (diffs, module arguments, loop items) end up on screen and in log files. Rules for the roles:

- secrets never in loop items: loop over a copy of the list without the secret field and look the secret up by
  name (loop items are logged in plain text, even if the module masks its own parameter)
- secrets in module arguments the module does not mask itself (e.g. `uri` headers, `command` arguments, `stdin`,
  request bodies): `no_log: true`
- files with secret content (`template`, `copy`): `no_log: true` and `diff: false`

## License

Apache-2.0

Created with the help of AI
