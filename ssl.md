## Update 18-04-2021 (mostly https://gist.github.com/r0lodex/0fe03fc8d22241d79cba65107b30868b)

```bash
NAME=$1

mkdir $NAME
cd $NAME

# Generate private key
openssl genrsa -des3 -out myCA.key 2048

# Generate root certificate
openssl req -x509 -new -nodes -key myCA.key -sha256 -days 825 -out myCA.pem

# Generate a private key
openssl genrsa -out $NAME.key 2048

# Create a certificate-signing request
openssl req -new -key $NAME.key -out $NAME.csr

# Create a config file for the extensions
>$NAME.ext cat <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $NAME # Be sure to include the domain name here because Common Name is not so commonly honoured by itself
DNS.2 = *.$NAME 
EOF

# Create the signed certificate
openssl x509 -req -in $NAME.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out $NAME.crt -days 825 -sha256 -extfile $NAME.ext
```
- Save the file somewhere e.g `ssl.sh` and run `bash ssl.sh domain.com`
- Using Chrome, hit a page on your server via HTTPS and continue past the red warning page (assuming you haven't done this already).
- Open up `Chrome Settings > Show advanced settings > HTTPS/SSL > Manage Certificates`
- Click the Authorities tab and scroll down to find your certificate under the Organization Name that you gave to the certificate.

---

# Setting Up Local SSL
The guide below is about setting up SSL for local development, 8 steps in 5 minutes.
We'll be using `openssl` to configure this, as we would on a production server.

---

1. Generate `rootCA.key` using `openssl`

```
openssl genrsa -des3 -out rootCA.key 2048
```


2. Generate `rootCA.pem`, you can specify any number of days at `-days` before the key expires

```
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem
```

3. Locally trust the certificate by importing `rootCA.pem` into Keychain Access and enable **Always Trust** on that certificate.

￼
![image](https://user-images.githubusercontent.com/6908558/66054287-470a8a80-e523-11e9-9b70-7ddc3d523a80.png)


4. Create new file with these settings, name it `server.csr.cnf`. This is to use this for importing in the later command. Fill the information as you filled previously

```
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn

[dn]
C=MY
ST=RandomState
L=RandomCity
O=RandomOrganization
OU=RandomOrganizationUnit
emailAddress=hello@example.com
CN = domain.com ## replace me
```

5. Create a new file `v3.ext` (X509 v3 certificate). Note the `@alt_names`, it's the domain we register to trust.

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = domain.com
DNS.2 = *.domain.com
```

6. Use the command below to generate the file `server.key`

```
openssl req -new -sha256 -nodes -out server.csr -newkey rsa:2048 -keyout server.key -config <( cat server.csr.cnf )
```

7. Run the command below to generate the file `server.crt`

```
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 500 -sha256 -extfile v3.ext
```

8. Bring `server.key` and `server.crt` to your nginx configuration.

```
listen 443 ssl;
listen [::]:443 ssl ipv6only=on;
ssl_certificate /etc/nginx/ssl/server.crt;
ssl_certificate_key /etc/nginx/ssl/server.key;
```