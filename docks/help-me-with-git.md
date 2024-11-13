# Instruction

## git preparation in new OS

- set user name and developer email (it not should be real name and real email)
```bash
git config --global user.name "developer"
git config --global user.name "developer@local"
```
- generate ssh key for Git SCM
```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
```

## quick help
- change remote url to $new_url
```bash
git remote set-url origin $new_url
```
