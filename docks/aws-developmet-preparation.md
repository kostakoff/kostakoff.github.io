# AWS preparation for development

## Prepare aws account
- Create AWS account
- Add payment method
- Create Route53 develpment route, can be in zone .link, it's cheap and can hide WHOIS info
- Request in AWS Certificate Manager wildcard certificate for your development domain
- Create deploy-sa iam user for develpmet deploy
- Add deploy-sa to all necessary groups, for example: CertMgrViewer, DNSAdmin, EC2Admin
- Install aws cli, for mac `brew install awscli`
- Install eksctl, for mac `brew install eksctl`
> The AWS Provider can source credentials and other settings from the shared configuration and credentials files. By default, these files are located at $HOME/.aws/config and $HOME/.aws/credentials on Linux and macOS, and "%USERPROFILE%\.aws\config" and "%USERPROFILE%\.aws\credentials" on Windows.
- Setup aws cli user
```
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Terraform preparation
- Install terraform, for mac `brew install terrafrom` 
- Setup terraform cache
```bash
echo 'plugin_cache_dir   = "$HOME/.terraform.d/plugin-cache"' > ~/.terraformrc
mkdir -p ~/.terraform.d/plugin-cache
```

## Prepare development certificates
- Generate CA, Server and Clien dev certificates
> Save it to `newcerts.sh`
```bash
#!/bin/bash

# Устанавливаем стандартные параметры
COUNTRY="CA"
STATE="BC"
LOCALITY="VAN"
ORGANIZATION="myltd"
ORGANIZATIONAL_UNIT="IT"
COMMON_NAME_CA="CA"
EMAIL="admin@myltd.lab"

# Создаем директории для хранения сертификатов и ключей
rm -rf ssl
mkdir -p ssl/ca/{certs,newcerts,private}
mkdir -p ssl/certs
touch ssl/ca/index.txt
echo 1000 > ssl/ca/serial

# Создаем файл конфигурации OpenSSL
cat > csr.conf <<EOF
[ ca ]
default_ca = CA_default

[ CA_default ]
certs             = ./ssl/ca/certs
crl_dir           = ./ssl/ca/crl
new_certs_dir     = ./ssl/ca/newcerts
database          = ./ssl/ca/index.txt
serial            = ./ssl/ca/serial
RANDFILE          = ./ssl/ca/private/.rand
private_key       = ./ssl/ca/private/ca.key.pem
certificate       = ./ssl/ca/certs/ca.cert.pem
crlnumber         = ./ssl/ca/crlnumber
crl               = ./ssl/ca/crl/ca.crl.pem
default_crl_days  = 30
default_md        = default
preserve          = no
policy            = policy_loose

[ policy_loose ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only
default_md          = sha256
prompt              = no

[ req_distinguished_name ]

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_server ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @server_alt_names

[ v3_client ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
subjectAltName = @client_alt_names

[ server_alt_names ]
DNS.1 = server.local

[ client_alt_names ]
DNS.1 = client.local
EOF

# Генерируем ключ и сертификат CA
openssl genrsa -out ssl/ca/private/ca.key.pem 4096
openssl req -config csr.conf -key ssl/ca/private/ca.key.pem -new -x509 -days 3650 -sha256 -extensions v3_ca -out ssl/ca/certs/ca.cert.pem -subj "/C=$COUNTRY/ST=$STATE/L=$LOCALITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME_CA/emailAddress=$EMAIL"
echo "CA сертификат и ключ созданы."

# Создаем серверный ключ и CSR
COMMON_NAME_SERVER="server.local"
openssl genrsa -out ssl/certs/server.key.pem 2048
openssl req -config csr.conf -key ssl/certs/server.key.pem -new -out ssl/certs/server.csr.pem -subj "/C=$COUNTRY/ST=$STATE/L=$LOCALITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME_SERVER/emailAddress=$EMAIL"
# Подписываем серверный сертификат с помощью CA
openssl ca -batch -config csr.conf -extensions v3_server -days 825 -notext -md sha256 -in ssl/certs/server.csr.pem -out ssl/certs/server.cert.pem
echo "Серверный сертификат и ключ созданы и подписаны."

# Создаем клиентский ключ и CSR
COMMON_NAME_CLIENT="client"
openssl genrsa -out ssl/certs/client.key.pem 2048
openssl req -config csr.conf -key ssl/certs/client.key.pem -new -out ssl/certs/client.csr.pem -subj "/C=$COUNTRY/ST=$STATE/L=$LOCALITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME_CLIENT/emailAddress=$EMAIL"
# Подписываем клиентский сертификат с помощью CA
openssl ca -batch -config csr.conf -extensions v3_client -days 825 -notext -md sha256 -in ssl/certs/client.csr.pem -out ssl/certs/client.cert.pem
echo "Клиентский сертификат и ключ созданы и подписаны."

echo "Все сертификаты и ключи созданы успешно."
```
- run certtficate generation
```bash
bash ./newcerts.sh
```
- Upload new certificates to AWS Certificate Manager
```bash
# upload ca
aws acm import-certificate --certificate file://ssl/ca/certs/ca.cert.pem --region ca-central-1
# combine cert chain
cat ssl/certs/server.cert.pem ssl/ca/certs/ca.cert.pem > server_full_chain.pem
# upload server cert
aws acm import-certificate --certificate file://server_full_chain.pem  --private-key file://ssl/certs/server.key.pem \
    --certificate-chain file://ssl/ca/certs/ca.cert.pem --region ca-central-1
# check acm certs
aws acm list-certificates --region ca-central-1
```
