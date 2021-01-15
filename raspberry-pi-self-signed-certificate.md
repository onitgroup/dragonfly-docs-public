# Create and deploy a self-signed certificate for the Raspberry PI

## Introduction

This gist describes the creation and deployment of a self-signed certificate for the [UV4L WebRTC server extension](https://www.linux-projects.org/uv4l/webrtc-extension/) on a **Raspberry PI**. A certificate is required in order to enable the calibration of the PI's camera using the [Accuware Dragonfly Demo - Calibration Mode](https://dragonfly-demo.accuware.com) or if you intend to use the UV4L server directly from within the Dragonfly app, i.e. by not utilizing a Signaling Server or Signaling Proxy.

You might consider to use the attached private key, server certificate and root-CA files (see section `Backup` at the end of the gist), before going and create new ones. If this is the case, please copy `raspberrypi.crt` and `raspberrypi.key.pem` to your PI, import the `ca.pem` to the certificate chain of your OS and proceed directly to section [Edit UV4L server configuration on the PI](#edit-uv4l-server-configuration-on-the-pi).

## Create a self signed certificate

It is recommended to [follow this tutorial](https://fabianlee.org/2018/02/17/ubuntu-creating-a-trusted-ca-and-san-certificate-using-openssl-on-ubuntu/) in order to create a valid self-signed certificate with SNA and a trustworthy root CA:

It is also recommended to add the root CA certificate to the trust chain of your OS and/or browser.

Following the tutorial these are the steps which created files in section `Backup`:

```bash
export prefix="raspberrypi"
cp /usr/lib/ssl/openssl.cnf $prefix.cnf
```

You may find `openssl.cnf` in `/System/Library/OpenSSL/` on macOS and `/etc/pki/tls` on Redhat variants.

### Edit `raspberrypi.cnf`

Make these sections look like so:

```ini
[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical, CA:TRUE, pathlen:3
keyUsage = critical, cRLSign, keyCertSign
nsCertType = sslCA, emailCA


[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
#extendedKeyUsage=serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = raspberrypi
```

Uncomment in section [ req ]:

```ini
[ req ]
req_extensions = v3_req
```

### Create CA certificate

You will be asked for a password. You can choose `accuware` or any other password of your choice.

```bash
openssl genrsa -aes256 -out ca.key.pem 2048
chmod 400 ca.key.pem
openssl req -new -x509 -subj "/CN=accuware-self-signed-ca" -extensions v3_ca -days 3650 -key ca.key.pem -sha256 -out ca.pem -config $prefix.cnf
```

`ca.pem` needs to be imported into the various key chains of browser and/or OS later on (see original tutorial).

### Create server certificate signed by CA

```bash
openssl genrsa -out $prefix.key.pem 2048
openssl req -subj "/CN=$prefix" -extensions v3_req -sha256 -new -key $prefix.key.pem -out $prefix.csr -config $prefix.cnf
openssl x509 -req -extensions v3_req -days 3650 -sha256 -in $prefix.csr -CA ca.pem -CAkey ca.key.pem -CAcreateserial -out $prefix.crt -extfile $prefix.cnf

```

Resulting `raspberrypi.crt` and `raspberrypi.key.pem` need to be copied to the PI and configured in `/etc/uv4l/uv4l-raspicam.conf`.

### Edit UV4L server configuration on the PI

```bash
sudo nano /etc/uv4l/uv4l-raspicam.conf
```

Change these options accordingly:

```ini
server-option = --port=443
server-option = --use-ssl=yes
server-option = --ssl-private-key-file=/home/pi/raspberrypi.key.pem
server-option = --ssl-certificate-file=/home/pi/raspberrypi.crt
```

### Restart the service

```bash
sudo service uv4l_raspicam restart
```

### More service options

<https://www.linux-projects.org/documentation/uv4l-raspicam/>

## Verify your installation

- Open `https://<ip-of-pi>/stream/webrtc` in a browser on another machine.
- Select `640x480 (30fps)` and `hardware video codec`
- Hit `Call`

You should see the camera video.

Don't forget to `Hangup` or close the browser window, otherwise the PI remains busy.

> Note: Since the certificate, which is used on your Raspberry, is a self-signed, you might need to accept this first in your browser, especially if you are using Firefox. You might have to add a security exception. Sometimes it might be necessary to repeat this procedure from time to time. Opening `https://<IP_of_PI>/stream` occasionally should be sufficient

## Backup

### Private server key

Copy and paste into a file named `raspberrypi.key.pem`:

```html
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAnXmVwWvLcRybFf91dIQlGmBo+sUfDV8n6BthtXvNQry1Pxl1
fiXKkWug5WU60GOFHyn9eRljbLgUYrVxSc4j9eiWoUgdnv2SEDsjwKzflDPxlQrY
Oqgg10jHQ144r8mYm/dqeU2cnI1GdNOiGiJkApHQYQK+uh6qrlWjPMnqOuvXPQLM
6tqo5n2gXpD62jGF62MyaCUxZy/1Z0GM+m3rP/sRGBELOXlCXhaYDC9rqDufBU6n
9AgBP3SiBxCYXhcskW0/RWSSc2oWsDVj62UdgKKWLETgKzxo+sx3Lg1jCiWgjEsU
ae7IZ9Y0qx8YtEeBKnkj6x+5rQrBfw4usV39XQIDAQABAoIBAQCIkL+5vQTydC9e
tWsj/9G5fTCtgTO7wfD2zoA/Bj1tCgBY13hYTOfOkzs2lUKbJCN3ck/arJTX3Q/4
xoeXzQjMose22LavghIgt1j7KDDA8wcoDP6WZ+YLLZd5KgYZFMifQcL5rcAK2E3o
1Pw4k+iNfezrpJjJCf1VMPlep3DVZQO8Absu4QJkbUBnC8SRF9BEvB46PeuNuYmt
Pp8oK0CbL/J+Hsk0NJn+Olr0Nc3LjYQMtHYlgJwSZPYmYno7O3mL9cQQoXvdAGEQ
/xft2gSIBkIBduboKQMhta6N5ZHH6MeA0mi9RwHw3u/F9FiZ0u8/XZUAQTA0gytX
AKWYB9qVAoGBAMu0P2iHBnilHz2YwJwqpUIe6BM82gISLo/cjbURVbQu6vT0f6VI
XPhGsX5VNUYz455m8WSurEMe5oTL7CbWiHTLphQKrSjGhMeQLIaKgjCj02X3qYcx
PvadVmkDQZwJTMuNvNJGugYPMn1YQ7+N+fonsPiaHh0pyAbe6NGkapT7AoGBAMXn
FcLFBWRSnNVw5OyxP7H3VTIs7ZZPaSaPQ5Tts1tIki5amGfHlQC8VKHMs8olmUjX
8c4en24iyNs7oo9nMz3gSxAMgA2Z+gq7DFn1LRc9DY3i8LzfMvJ1a5F2xNec5jip
fQ775gO8OGvsN7JcQKY80xzoc9CjMJGHIkHh3reHAoGAc3Il9YGAw2Mhf3FQx7DL
k9ucPzrfewj+5n1iulmmrsVgV48xwGRwfCzkbuqvlKfXunAxIpR0AF5E2sIPhjtT
fo3kA7vBQzivC8LD2UQqYJKYPlPL+liIjI/C5yT3TA1hPoOHncyDpOd7/9nEG43F
PGa+P0ZpBrIlMO+oFxgNZ30CgYEAvNA3y8br2QaUyXNXhpepvKLMbv28g/8pxHdF
NE8BIyN/DKi05bbea4BDgsdp9YCf2YbmFhDTbWHUno4sD4OXuP5Iv3wdpFx22kwR
gbZQme5PA0M1Cg4tbnQm9/cH4Oq6H+9c+LHOh1vJvPX3Qb2QlMpNZTRGYxV/Xik+
vvq/4fsCgYEAgNAlnO/4sBNhSDgDwRozq6r3WpseISsoM1XkCJRuPBYp1YfcAVKW
bjsBg7EhGEyj789ysMS2c0SOq2bfc1yc8kncg97j9TBBtVkpBs/gidh7WXBYGb7q
Xl8qhnnDdGtgCYCjGdRUlhp+hZ1w70nJv18zeS7e6KFQ/Roykrg8rt4=
-----END RSA PRIVATE KEY-----
```

### Self-signed server certificate

Copy and paste into a file named `raspberrypi.crt`:

```html
-----BEGIN CERTIFICATE-----
MIIC7TCCAdWgAwIBAgIJAIGL+3Iks2XGMA0GCSqGSIb3DQEBCwUAMCIxIDAeBgNV
BAMMF2FjY3V3YXJlLXNlbGYtc2lnbmVkLWNhMB4XDTE4MDkwOTE1NTc1NFoXDTI4
MDkwNjE1NTc1NFowFjEUMBIGA1UEAwwLcmFzcGJlcnJ5cGkwggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQCdeZXBa8txHJsV/3V0hCUaYGj6xR8NXyfoG2G1
e81CvLU/GXV+JcqRa6DlZTrQY4UfKf15GWNsuBRitXFJziP16JahSB2e/ZIQOyPA
rN+UM/GVCtg6qCDXSMdDXjivyZib92p5TZycjUZ006IaImQCkdBhAr66HqquVaM8
yeo669c9Aszq2qjmfaBekPraMYXrYzJoJTFnL/VnQYz6bes/+xEYEQs5eUJeFpgM
L2uoO58FTqf0CAE/dKIHEJheFyyRbT9FZJJzahawNWPrZR2AopYsROArPGj6zHcu
DWMKJaCMSxRp7shn1jSrHxi0R4EqeSPrH7mtCsF/Di6xXf1dAgMBAAGjMjAwMAkG
A1UdEwQCMAAwCwYDVR0PBAQDAgXgMBYGA1UdEQQPMA2CC3Jhc3BiZXJyeXBpMA0G
CSqGSIb3DQEBCwUAA4IBAQDHO058R8y69AeQyiq9Ip3mhXPhss1rz1vLLcVg19Pp
+VBDmWEhx7eG0fruernJs1EUgH16s6wkS5yNWUXc2TVPrdydcymuy75AqsrQ65WZ
vUlgaaRwRYDENfcHFAhoVEFHvFIuLMQm31EH4jYmnf1jaJtst+r3lVL3JXml1Olb
AJWRc5OGqtYLBAsiIPYEvHSrpk2LGFKa31Cy+QyrgmyRjpEL8igCJoNqEXjVCj2/
Rx+Icuq5Pf4jeQvhj9B5Q6pU57mFpcDhRk6Ie0XB4bczNsC/Ocw3F2cVPzeAOkND
jecho80anNTWk0udmnRQDnAwl9SGA55akM36w1fP6z0p
-----END CERTIFICATE-----
````

### Root-CA certificate

Copy and paste into a file named `ca.pem`:

```html
-----BEGIN CERTIFICATE-----
MIIDOjCCAiKgAwIBAgIJAID9AXwp0GVCMA0GCSqGSIb3DQEBCwUAMCIxIDAeBgNV
BAMMF2FjY3V3YXJlLXNlbGYtc2lnbmVkLWNhMB4XDTE4MDkwOTE1NTcyNloXDTI4
MDkwNjE1NTcyNlowIjEgMB4GA1UEAwwXYWNjdXdhcmUtc2VsZi1zaWduZWQtY2Ew
ggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDgHiBjMUH8vtPk4DDMsKjJ
/nG6d4ZlA646RBB7s9zr04E9QBmcRUgefmEL6AGV4Fr4WGCAGdulG9rIqjBDNIP/
KQmqTZPsTL6y1gx8hWWlLr+Vi6okHEbJUUnFIYl6yM3ZccfPvOZt0vuSsPpM5kaQ
3Ph/g+VoTP4hfzPMipEqVSzS+lZRYjsE5wM/YF61lIGWvn4V7RZk08KWbqcN5Wh8
Do5idCwqrIsndw0t48mpIgi1ER3Z54fIHDXMCvBCTJQIVZF5eGz6b0Rb7W/XxSjH
mlJ+0BwBb0eZQyNLFibs1MPt/RcgWxeUJjqtthS7+zkqMsc+2Why0hHH3i29XHHr
AgMBAAGjczBxMA4GA1UdDwEB/wQEAwIBBjARBglghkgBhvhCAQEEBAMCAQYwHQYD
VR0OBBYEFEqy41OSbduYEVFvZRZvPvkV138IMB8GA1UdIwQYMBaAFEqy41OSbduY
EVFvZRZvPvkV138IMAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBADxI
+Q1cQFAcWjZwF3CjM3ZhDc8r9wPh+GK1XR3ZR3B6Txm/v2pIh7wwJ4sA3V+QdVc9
DYMSjWPu/6DqFmCLpsg31QdGUngnAvvE9od8z6GZGYNShcTPgVWHWYcsESOdZyUI
j5aHz9HKXyTP3F1PmcKDKX3Ld9Ewwv9JKd3dQ8elRD0761/wpRCAYV22eiXb1tkM
WBzxsZpedCRCTwXupXQR0508Vx249TkakON2tymX5SZyrJJPCGnLgJleu5hUMOPn
8hT7UJ7fNy0PsDdc1K0CQ5CLnznl6xxpUTLPX50qrMx3jrgHIAGznMMS1Fgno9OF
Gf+MhBcPpm5hMUMdvz8=
-----END CERTIFICATE-----
````
