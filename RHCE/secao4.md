# File examples for the EXAM

## inventory

```ini
[dev]
servera

[test]
serverb

[prod]
serverc
serverd

[balancers]
servere

[webservers:children]
prod
```

## ansible.cfg

```ini
[defaults]
remote_user=student
inventory=/home/student/ansible/inventory
roles_path=/home/student/ansible/roles
collections_path=/home/student/ansible/collections
ask_pass=false

[privilege_escalation]
become=true
become_method=sudo
become_user=root
become_ask_pass=false
```

## .vimrc

```vim
autocmd FileType yaml setlocal ts=2 sw=2 ai et cuc nu
```
