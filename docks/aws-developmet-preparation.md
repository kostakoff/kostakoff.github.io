# AWS preparation for development
- Create AWS account
- Add payment method
- Create Route53 develpment route, can be in zone .link, it's cheap and can hide WHOIS info
- Request in AWS Certificate Manager wildcard certificate for your development domain
- Create deploy-sa iam user for develpmet deploy
- Add deploy-sa to all necessary groups, for example: CertMgrViewer, DNSAdmin, EC2Admin
- install aws cli, for mac `brew install awscli`
> The AWS Provider can source credentials and other settings from the shared configuration and credentials files. By default, these files are located at $HOME/.aws/config and $HOME/.aws/credentials on Linux and macOS, and "%USERPROFILE%\.aws\config" and "%USERPROFILE%\.aws\credentials" on Windows.
- setup aws cli user
```
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Terraform preparation
- install terraform
- setup terraform cache
```bash
echo 'plugin_cache_dir   = "$HOME/.terraform.d/plugin-cache"' > ~/.terraformrc
mkdir -p ~/.terraform.d/plugin-cache
```
