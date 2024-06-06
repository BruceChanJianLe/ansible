tags:
  - setup (all)
  - core
  - ssh
  - base16
  - nvim
  - tmux
  - zsh
  - dotfiles
  - private_aliases

Install dependencies (ansible, git)
```bash
sudo apt install ansible git -y
```

Setting up new laptop
```bash
ANSIBLE_ASK_VAULT_PASS=True ansible-pull -U https://github.com/brucechanjianle/ansible --ask-vault-pass -e "enable_decryption=true dotfile_branch=master" --ask-become-pass
```

Setting up docker
```bash
ansible-pull -U https://github.com/brucechanjianle/ansible -e "dotfile_branch=docker"
```

Setting up for a specific group of users defined in inventory file
```bash
ansible-playbook local.yml --ask-become-pass -l others
```

## Reference
- [A good read](https://wearenotch.com/speed-up-ansible-playbook-execution/#:~:text=The%20first%20time%20a%20playbook,due%20to%20Ansible's%20idempotence%20checking.)
