tags:
  - setup (all)
  - core
  - ssh
  - base16
  - nvim
  - tmux
  - zsh
  - dotfiles

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
