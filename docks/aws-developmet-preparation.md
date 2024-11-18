# AWS preparation for development

## Prepare aws account
- Create AWS account
- Add payment method
- Create Route53 develpment route, can be in zone .link, it's cheap and can hide WHOIS info
- Request in AWS Certificate Manager wildcard certificate for your development domain
- Create deploy-sa iam user for develpmet deploy
- Add deploy-sa to all necessary groups, for example: CertMgrViewer, DNSAdmin, EC2Admin
- Install aws cli, for mac `brew install awscli`
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
- Install terraform
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
mkdir -p ca/{certs,newcerts,private}
mkdir -p certs private
chmod 700 ca/private
chmod 700 private
touch ca/index.txt
echo 1000 > ca/serial

# Создаем файл конфигурации OpenSSL
cat > openssl.cnf <<EOF
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = ./ca
certs             = \$dir/certs
crl_dir           = \$dir/crl
new_certs_dir     = \$dir/newcerts
database          = \$dir/index.txt
serial            = \$dir/serial
RANDFILE          = \$dir/private/.rand
private_key       = \$dir/private/ca.key.pem
certificate       = \$dir/certs/ca.cert.pem
crlnumber         = \$dir/crlnumber
crl               = \$dir/crl/ca.crl.pem
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
C                     = $COUNTRY
ST                    = $STATE
L                     = $LOCALITY
O                     = $ORGANIZATION
OU                    = $ORGANIZATIONAL_UNIT
emailAddress          = $EMAIL

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_server ]
basicConstraints = CA:FALSE
nsCertType = server
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ v3_client ]
basicConstraints = CA:FALSE
nsCertType = client
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth

[ alt_names ]
DNS.1 = server.local
EOF

# Генерируем ключ и сертификат CA
openssl genrsa -out ca/private/ca.key.pem 4096
chmod 400 ca/private/ca.key.pem

openssl req -config openssl.cnf -key ca/private/ca.key.pem -new -x509 -days 3650 -sha256 -extensions v3_ca -out ca/certs/ca.cert.pem -subj "/C=$COUNTRY/ST=$STATE/L=$LOCALITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME_CA/emailAddress=$EMAIL"
chmod 444 ca/certs/ca.cert.pem

echo "CA сертификат и ключ созданы."

# Создаем серверный ключ и CSR
COMMON_NAME_SERVER="server.local"

openssl genrsa -out private/server.key.pem 2048
chmod 400 private/server.key.pem

openssl req -config openssl.cnf -key private/server.key.pem -new -out server.csr.pem -subj "/C=$COUNTRY/ST=$STATE/L=$LOCALITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME_SERVER/emailAddress=$EMAIL"

# Подписываем серверный сертификат с помощью CA
openssl ca -batch -config openssl.cnf -extensions v3_server -days 825 -notext -md sha256 -in server.csr.pem -out certs/server.cert.pem
chmod 444 certs/server.cert.pem

echo "Серверный сертификат и ключ созданы и подписаны."

# Создаем клиентский ключ и CSR
COMMON_NAME_CLIENT="client"

openssl genrsa -out private/client.key.pem 2048
chmod 400 private/client.key.pem

openssl req -config openssl.cnf -key private/client.key.pem -new -out client.csr.pem -subj "/C=$COUNTRY/ST=$STATE/L=$LOCALITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME_CLIENT/emailAddress=$EMAIL"

# Подписываем клиентский сертификат с помощью CA
openssl ca -batch -config openssl.cnf -extensions v3_client -days 825 -notext -md sha256 -in client.csr.pem -out certs/client.cert.pem
chmod 444 certs/client.cert.pem

echo "Клиентский сертификат и ключ созданы и подписаны."

# Убираем ненужные файлы
rm server.csr.pem client.csr.pem

echo "Все сертификаты и ключи созданы успешно."
```
- run certtficate generation
```bash
mkdir ./ssl && cd ./ssl
bash ./newcerts.sh
```
- Upload new certificates to AWS Certificate Manager
```bash
# upload ca
aws acm import-certificate --certificate file://ca/certs/ca.cert.pem --region ca-central-1
# combine cert chain
cat certs/server.cert.pem > server_full_chain.pem
cat ca/certs/ca.cert.pem >> server_full_chain.pem
# upload server cert
aws acm import-certificate --certificate file://server_full_chain.pem  --private-key file://private/server.key.pem \
    --certificate-chain file://ca/certs/ca.cert.pem --region ca-central-1
# check acm certs
aws acm list-certificates --region ca-central-1
```
