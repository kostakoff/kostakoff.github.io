# Instruction
## How to prepare ubuntu20 for application development

### Required resources
- [] ubuntu 24.10
- [] 24Gb memory
- [] 10 cores

### Steps
- install apt tools
```bash
sudo apt install git mc xca openjdk-11-jdk openjdk-17-jdk maven virtualenv ansible python3 nmap docker.io docker-compose skopeo terminator caffeine p7zip-full
```
- install snap tools
```bash
for package in gtkhash "chromium  --classic" "code --classic" dbeaver-ce drawio "go --classic" "groovy --classic" "helm --classic" "intellij-idea-ultimate --classic" "kubectl --classic" pinta postman vlc; do sudo snap install $package; done
```
