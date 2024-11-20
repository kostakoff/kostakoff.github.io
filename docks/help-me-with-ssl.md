# Instruction

## Requirments
- [X] openssl
- [X] openjdk11 or higher

## SSL hot commands

- get certchain from ssl host:port
```bash
openssl s_client -showcerts -verify 5 -connect pypi.org:443 < /dev/null > ./chain.txt
```
- crete p12 from pem with CA in certchain
```bash
cat ./client.crt ./ca.crt > ./client.pem
openssl pkcs12 -export -out store.p12 -in ./client.pem -inkey ./client.key
```
- java keytool list cert/cert ket pair in p12
```bash
keytool -list -keystore ./store.p12 (-v optional)
```
- openssl list cert/key pair in p12
```bash
openssl pkcs12 -info -in ./store.pfx
```
- keytool import cert to store
```bash
keytool -importcert -file ./epa.crt -keystore truststore.jks -alias "epa"
```
- get cert from ssl host:port
```bash
echo -n | openssl s_client -connect pypi.org:443 -servername pypi.org | openssl x509 > ./pypi.org:443.crt
```
- keystore change certificate alias
```bash
keytool -changealias -alias fpch -destalias localhost -keystore ./keystore.jks
```
- verify client cert
```bash
openssl verify -CAfile ./ca.crt -untrusted ./subca.crt ./client.crt
```
- keytool change keystore paswword
```bash
keytool -storepasswd -keystore ./store.p12
```
- convert p12 to jks
```bash
keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12 -destkeystore keystore.jks -deststoretype jks
```
- export all certs from p12
```bash
keytool -list -rfc -keystore ./truststore.jks >> store.pem
```
- delete certificate with alias from jks or p12
```bash
keytool -delete -alias "caroot" -keystore ./kafka.keystore.jks
```
- encrypt file with public key
```bash
openssl rsautl -encrypt -in ./password.txt -out ./password.txt.enc -pubin -inkey ./key.pem
```
- decrypt file with private key
```bash
openssl rsautl -decrypt -in ./password.txt.enc -out ./password2.txt -inkey ./key.pem
```

## Certificate request temolate
- create certificate request configuration ssl_req.conf
```ini
[req]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[dn]
C = CA
ST = BC
L = VAN
O = Local
OU = Home
CN = my.domain.local

[req_ext]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = my.domain.local
DNS.2 = my2.dmain.local
```
- create certificate request file
```bash
openssl req -newkey rsa:2048  -nodes -sha256 -keyout key.pem -out req.csr -config ssl_req.conf
```
- verify csr
```bash
openssl req -text -noout -verify -in req.csr
```
