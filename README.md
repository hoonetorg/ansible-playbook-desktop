# ansible-playbook-desktop

Sets up desktop/notebook hosts (group `desktop`) with these roles, in this order:

| role | purpose |
|------|---------|
| ansible-role-repo | additional package repositories |
| ansible-role-disk | partitions, LUKS, btrfs filesystems/subvolumes, crypttab/fstab |
| ansible-role-user | groups, users, home directories |
| ansible-role-btrbk | btrbk snapshots/backups with own systemd units |

(ansible-role-software is prepared but not enabled yet.)

Each role reads one dict named after it (`repo`, `disk`, `user`, `btrbk`) from the inventory;
see the role READMEs.

## Usage

```bash
ansible-playbook -i <inventory> -k -K --ask-vault-password site.yml -l <host>
```

## License

Apache-2.0

Created with the help of AI
