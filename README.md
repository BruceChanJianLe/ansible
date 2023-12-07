tags:
  - setup
  - ssh
  - dotfiles

tasks:
  - ssh.yml
    - setup
    - ssh
    - dotfiles
  - dotfiles.yml

Setting up new laptop
```bash
ANSIBLE_ASK_VAULT_PASS=True ansible-pull -U https://github.com/brucechanjianle/ansible --ask-vault-pass -e "enable_decryption=true dotfile_branch=master" --ask-become-pass
```

Setting up docker
```bash
ansible-pull -U https://github.com/brucechanjianle/ansible
```
