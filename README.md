# SSL Certificate Creation

This repository contains all manners of scripts for handling SSL
certificates. It includes also all data necessary for those scripts
like intermediate and root certificates for example.

## Creation of Certificate Signing Requests

### Introduction

This script comes with a template for generating a Certificate
Signing Request (CSR)). It relies on the [OpenSSL](http://www.openssl.org/)
utility [`openssl`](https://www.openssl.org/docs/apps/openssl.html)
for generating the certificates using a template.

### Customization

Edit the `openssl_csr.conf` template and/or the `defaults.sh` file to
fit your needs.

You also have to add intermediate certificates from other CAs for
generating the bundles you must then modify the `create_bundle` script
to fit the new authority. Please fork and issue a pull
request. Currently the supported certificates are:

 * Thawte Class 2 certificates: wildcard and normal.
 * StartSSL Class 1 (free) certificates.

### Usage

    generate_csr -d <domain> -p <private key> -t openssl_csr.conf

This makes `certtool` generate a CSR for the domain and/or Common Name
`<domain>` (`-d` option)  using the **template** `openssl_csr.conf`
(`-t` option). The CSR will be created in the current directory.

The options for the script are:

 * `-d`:  domain of the CSR to be generated.

 * `-e`: days for the certficate validity. This can be specified as a
   multiple of 365 or as `1y` for one year, `2y` for two years and so
   on (default: 365 or 1y). 

 * `-o`: organization for which the CSR is generated.

 * `-t`: template for generating the certificates. Check the
         accompanying template for examples.

 * `-v`: verbose output. It shows the generated CSR.

### Example

    generate_csr -d *.example.com -o" ACME Foobar Inc" -t openssl_csr.conf -v

This command generates a CSR for a wildcard `*.example.com` SSL
certificate named `wildcard.example.com_csr.prm` for the `ACME Foobar
Inc` organization using the private key
`wirldcard.example.com_key.pem` using the `openssl_csr.conf` template
and it prints the CSR after generating it.

## Creation of certificate bundles

### Introduction

Since [nginx](http://nginx.org) doesn't have a directive for
Certificate Authority bundles the all chain from web server
certificate up to the root CA certificate has to be inside a single
file. This is the file that is used by the [`ssl_certificate`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) directive.

This script creates a bundle given all the certificates on the chain.

The Certificate Authority intermediate and root certificates are
placed in a directory that is named as the Certificate Authority,
e.g., `thawte`.

### Usage

    create_bundle -c thawte -w example.com_cert.pem

this creates a bundle with the **web server certificate**, the
**intermediate** and the **root** certificates of the Certificate
Authority [Thawte](https://www.thawte.com/). This bundle is created in
place of the **original** `example.com_cert.pem` certificate file.

The original certificate, as supplied by the CA is renamed to
`example.com_cert_orig.pem`.

This is the file to be **entered** in the nginx configuration.

The options for the script are:

 * `-t`: the directory name where the CA intermediate and root
   certificates reside. This should be named using the CA name:
   startssl, thawte, geotrust, etc.
   
 * `-w`: wether the certificate is or not a wild card certificate.

 * `<server certificate>`: the file name of the web server
   certificate as issued by the CA.

## Putting things into place: Security considerations

 1. The **private key** must be set to **modeg** `600` and **owned**
    `root.root`. The key must be placed at `/etc/ssl/private`.

 2. The certificate must be moved to `/etc/ssl/certs`.

 3. Done.

## What's next

Configuring nginx.

