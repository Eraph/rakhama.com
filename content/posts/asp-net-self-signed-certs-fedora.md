---
title: "Asp.NET Self-Signed Certificates and Fedora"
description: "Fixing certificate errors in Fedora Linux."
date: 2022-11-02T16:12:24+10:00
draft: false
categories:
  - Programming
tags:
  - programming
  - .net
  - ssl
  - linux
  - fedora
  - tips
  - future reference
---
Today I spent a bit of time fighting with certificates in an ASP.NET application I'm working on. The scenario is we have Blazor Server communicating with a Minimal API. Debugging locally I was struggling to get the two to communicate, with errors like:

> The remote certificate is invalid because of errors in the certificate chain: UntrustedRoot

<!--more-->

Now I'd followed the [instructions to install the self-signed developer certificate on Linux](https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio#trust-https-certificate-on-linux) with no change in behaviour. Turns out the distribution I'm using for work, [Fedora](https://getfedora.org/), does things a little bit differently. The gory details can be found on [this GitHub issue](https://github.com/dotnet/aspnetcore/issues/32361) but the solution for me was found in [this particular comment](https://github.com/dotnet/aspnetcore/issues/32361#issuecomment-839742072). I'm sharing the script here for posterity's sake.

``` bash
dnf list installed nss-tools >/dev/null 2>&1 ||
  (echo "Installing dependencies." && \
  sudo dnf install -y nss-tools)

echo "Exporting developer certificate."
DEV_CERT="$HOME/aspnet-$USER.pem"
dotnet dev-certs https -ep "$DEV_CERT" --format PEM

CERT_DB=$(echo "$HOME/.mozilla/firefox/*.default-release")
[ -d "$CERT_DB" ] && echo "Adding certificate to Firefox default profile certificates." && \
  certutil -d "$CERT_DB" -A -t "C,," -n localhost -i "$DEV_CERT"

CERT_DB="$HOME/.pki/nssdb"
[ -d "$CERT_DB" ] && echo "Adding certificate to Edge/Chrome certificates." && \
  certutil -d "$CERT_DB" -A -t "C,," -n localhost -i "$DEV_CERT"

echo "Adding certificate to System certificates."
sudo cp "$DEV_CERT" /etc/pki/tls/certs
sudo update-ca-trust

rm "$DEV_CERT"
```

For the inexperienced Linux user:

1. Copy the above lines into a new file with a `.sh` suffix (e.g. `my-script.sh`)
2. Set permissions on the file to make it executable: `chmod +x my-script.sh`
3. Run the script: `./my-script.sh`