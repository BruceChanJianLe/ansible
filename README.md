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
  - brave
  - nix
  - ghostty
  - additional (not meant for containers)

Install dependencies (ansible, git)
```bash
sudo apt install ansible git -y
```

Setting up new laptop
```bash
# Full Setup without decryption (For anyone who wants to try)
ansible-pull -U https://github.com/brucechanjianle/ansible --ask-become-pass -C experimental

# Full Setup with decryption
ANSIBLE_ASK_VAULT_PASS=True ansible-pull -U https://github.com/brucechanjianle/ansible --ask-vault-pass -e "enable_decryption=true" --ask-become-pass -C experimental

# Partial Setup, works for docker as well
ansible-pull -U https://github.com/brucechanjianle/ansible --skip-tags additional --ask-become-pass
```

Setting up for a specific group of users defined in inventory file
```bash
ansible-playbook local.yml --ask-become-pass -l others
```

## Reference
- [A good read](https://wearenotch.com/speed-up-ansible-playbook-execution/#:~:text=The%20first%20time%20a%20playbook,due%20to%20Ansible's%20idempotence%20checking.)
