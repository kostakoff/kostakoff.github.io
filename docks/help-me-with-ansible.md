# Instruction

## ansible instllation
- install through homebrew
```bash
brew install ansible
```
- install through PIP
```bash
pip3 install ansible
```

## ansible hot commands

- ansible ecrypt
```bash
ansible-vault encrypt ./test.txt
```
- ansible decrypt
```bash
ansible-vault decrypt ./test.txt
```
- ansible run playbook
```bash
ansible-playbook -vvv playbook.yaml -i hosts.yaml --tags sometag
```
- ansible install galaxy collection from git repo [more](https://docs.ansible.com/ansible/latest/collections_guide/collections_installing.html) 
```bash
ansible-galaxy collection install git+git@github.com:kostakoff/macos.softwareupdate.git,main --upgrade
```
