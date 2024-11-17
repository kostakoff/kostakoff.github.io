# AWS preparation for development
- Create AWS account
- Add payment method
- Create Route53 develpment route, can be in zone .link, it's cheap and can hide WHOIS info
- Request in AWS Certificate Manager wildcard certificate for your development domain
- Create deploy-sa iam user for develpmet deploy
- Add deploy-sa to all necessary groups, for example: CertMgrViewer, DNSAdmin, EC2Admin

## Terraform preparation
- install terraform
- setup terraform cache
```bash
echo 'plugin_cache_dir   = "$HOME/.terraform.d/plugin-cache"' > ~/.terraformrc
mkdir -p ~/.terraform.d/plugin-cache
```
