# Instruction

## Info

[INSTALLATION PAGE](https://brew.sh)

## Hot commands

- install 
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- uninstall
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```
- add to PATH
```bash
echo >> /Users/$(whoami)/.profile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/$(whoami)/.profile
eval "$(/opt/homebrew/bin/brew shellenv)"
```
- list installed packages
```bash
brew list installed
```
- brew switch beatween packages versions
```bash
brew unlink mytool@1.1
brew link mytool@2.0
```
