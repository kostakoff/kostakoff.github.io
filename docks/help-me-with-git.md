# Instruction

## GIT preparation in new OS

- set user name and developer email (it not should be real name and real email)
```bash
git config --global user.name "developer"
git config --global user.email "developer@local"
git config --global core.editor nano
```
- generate ssh key for Git SCM
```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
```

## GIT hot commands
- change remote url to $new_url
```bash
git remote set-url origin $new_url
```
- combine few commits in to one (where 2 is count of commits)
```bash
git rebase -i HEAD~2
```
and set `squash` to each combined commit
- push git push tags (optional force)
```bash
git push --force --tags
```
