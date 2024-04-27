# ansible-pilote

Ansible repository to manage `pilote`, a Raspberry Pi host running monitoring
software at home. This host was described in my [Journey of a Home-based
Personal Cloud Storage
Project](https://julien.riou.xyz/socallinuxexpo2024.handout.html) talk. This is
a personal repository that you can use as an example.

# Requirements

1. Configure network
1. Configure password for root user
1. Allow password for root user on SSH
1. Enable and start ssh.service
1. Write IP address in `inventory/hosts` file
1. Update variables in `group_vars/pilote.yml` file

# First run

```
ansible-playbook --ask-pass main.yml
```

# Subsequent runs

```
ansible-playbook main.yml
```

# Variables

See [documentation](group_vars/README.md).
