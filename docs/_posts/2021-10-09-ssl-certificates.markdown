---
layout: single
title:  "SSL Certificates"
date:   2021-10-09 11:30:00 +1000
toc: true
categories: openssl ssl certificates
---

This page has handy hints on certificates and how to use them.

## Creating Self Signed Certificate

The below command will create a public certifiacte and private key which will be valid for 365 days.

{% highlight bash %}
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
{% endhighlight %}

## Creating Self Signed Certificate using template

To save entering the certificate values manually a SSL config file can be used to generate a certificate.  The comments give an example commmand to generate a certificate using this config file.

{% highlight ssl %}
######################################################
# OpenSSL config to generate a self-signed certificate
#
# Create certificate with:
# openssl req -x509 -new -nodes -days 720 -keyout selfsigned.key -out selfsigned.pem -config openssl.cnf
#
# Remove the -nodes option if you want to secure your private key with a passphrase
#
######################################################

################ Req Section ################
# This is used by the `openssl req` command
# to create a certificate request and by the
# `openssl req -x509` command to create a
# self-signed certificate.

[ req ]

# The size of the keys in bits:
default_bits       = 2048

# The message digest for self-signing the certificate
# sha1 or sha256 for best compatability, although most
# OpenSSL digest algorithm can be used.
# md4,md5,mdc2,rmd160,sha1,sha256
default_md = sha256

# Don't prompt for the DN, use configured values instead
# This saves having to type in your DN each time.

prompt             = no
string_mask        = default
distinguished_name = req_dn

# Extensions added while singing with the `openssl req -x509` command
x509_extensions = x509_ext

[ req_dn ]

countryName            = AU
stateOrProvinceName    = WA
organizationName       = Some Company
commonName             = localhost

[ x509_ext ]

subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always

# No basicConstraints extension is equal to CA:False
# basicConstraints      = critical, CA:False

keyUsage = critical, digitalSignature, keyEncipherment

extendedKeyUsage = serverAuth

subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = someserver.somedomain
DNS.3 = 127.0.0.1

{% endhighlight %}

## Getting a Public Key from Public Certificate

Use the below command to get a public key in PEM format from a public certificate.

{% highlight bash %}
$ openssl x509 -pubkey -noout -in cert.pem  > pubkey.pem
{% endhighlight %}

## Using Python and Certificates

The `cryptography` library can be used to generate certificates or read public and private keys.  Refer to [Python Cryptograry docs][python-crypt]

Useful functions for reading public and private PEM keys can be found at [Key Serialization][python-crypt-pem-files].

Example code to create private and public certificate is shown below (based on code example from [python cryptography][python-crypt]).

{% highlight python %}
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.x509.oid import NameOID
from cryptography.hazmat.primitives import hashes
import datetime

# Generate our key
key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048,
)
# Write our key to disk for safe keeping
with open("mykey.pem", "wb") as f:
    f.write(key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.TraditionalOpenSSL,
        encryption_algorithm=serialization.BestAvailableEncryption(b"passphrase"),
    ))

# Various details about who we are. For a self-signed certificate the
# subject and issuer are always the same.
subject = issuer = x509.Name([
    x509.NameAttribute(NameOID.COUNTRY_NAME, u"US"),
    x509.NameAttribute(NameOID.STATE_OR_PROVINCE_NAME, u"California"),
    x509.NameAttribute(NameOID.LOCALITY_NAME, u"San Francisco"),
    x509.NameAttribute(NameOID.ORGANIZATION_NAME, u"My Company"),
    x509.NameAttribute(NameOID.COMMON_NAME, u"mysite.com"),
])
cert = x509.CertificateBuilder().subject_name(
    subject
).issuer_name(
    issuer
).public_key(
    key.public_key()
).serial_number(
    x509.random_serial_number()
).not_valid_before(
    datetime.datetime.utcnow()
).not_valid_after(
    # Our certificate will be valid for 10 days
    datetime.datetime.utcnow() + datetime.timedelta(days=10)
).add_extension(
    x509.SubjectAlternativeName([x509.DNSName(u"localhost")]),
    critical=False,
# Sign our certificate with our private key
).sign(key, hashes.SHA256())
# Write our certificate out to disk.
with open("mycertificate.pem", "wb") as f:
    f.write(cert.public_bytes(serialization.Encoding.PEM))
{% endhighlight %}

If a public key is required then the below code can be used to write the public key in PEM format.

{% highlight python %}
with open("mypubkey.pem", "wb") as f:
    f.write(key.public_key().public_bytes(cryptography.hazmat.primitives.serialization.Encoding.PEM, 
            cryptography.hazmat.primitives.serialization.PublicFormat.SubjectPublicKeyInfo)
    )
{% endhighlight %}

Reading public and private keys:

{% highlight python %}
from cryptography.hazmat.primitives import serialization

# read the private key specifying the password
with open("key.pem", "rb") as key_file:
    private_key = serialization.load_pem_private_key(
        key_file.read(),
        password=bytes('thepassword', 'utf-8'),
    )

# read the public key
with open('mypubkey.pem', 'rb') as cert_file:
    public_key = serialization.load_pem_public_key(
        cert_file.read()
    )
{% endhighlight %}


[python-crypt]: https://cryptography.io/
[python-crypt-pem-files]: https://cryptography.io/en/latest/hazmat/primitives/asymmetric/serialization/#cryptography.hazmat.primitives.serialization.load_pem_private_key