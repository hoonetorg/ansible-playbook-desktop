# ansible-playbook-desktop

Sets up desktop/notebook hosts (group `desktop`) with these roles, in this order:

| role | purpose |
|------|---------|
| ansible-role-repo | additional package repositories |
| ansible-role-disk | partitions, LUKS, btrfs filesystems/subvolumes, crypttab/fstab |
| ansible-role-user | groups, users, home directories |
| ansible-role-syncthing | syncthing per user, devices/folders via REST API |
| ansible-role-btrbk | btrbk snapshots/backups with own systemd units |

(ansible-role-software is prepared but not enabled yet.)

Each role reads one dict named after it (`repo`, `disk`, `user`, `syncthing`, `btrbk`) from the inventory;
see the role READMEs.

## Usage

```bash
ansible-playbook --diff -i <inventory> -k -K --ask-vault-password site.yml -l <host>
```

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
