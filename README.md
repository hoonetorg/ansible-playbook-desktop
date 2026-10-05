# ansible-playbook-desktop

Sets up desktop/notebook hosts (group `desktop`) with these roles, in this order:

| role | purpose |
|------|---------|
| ansible-role-repo | additional package repositories, distribution non-free repos |
| ansible-role-hardware | firmware, drivers, power management, hardware quirks, VM guest agents |
| ansible-role-disk | partitions, LUKS, btrfs filesystems/subvolumes, crypttab/fstab |
| ansible-role-user | groups, users, home directories |
| ansible-role-syncthing | syncthing per user, devices/folders via REST API |
| ansible-role-citrix | Citrix Workspace app (ICAClient), RPM downloaded on the controller |
| ansible-role-btrbk | btrbk snapshots/backups with own systemd units |

(ansible-role-software is prepared but not enabled yet.)

Each role reads one dict named after it (`repo`, `hardware`, `disk`, `user`, `syncthing`, `citrix`, `btrbk`) from the inventory;
see the role READMEs.

## Usage

```bash
ansible-playbook --diff -i <inventory> -k -K --ask-vault-password site.yml -l <host>
```

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
