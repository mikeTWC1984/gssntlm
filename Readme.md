# Setting up NTLM auth on dotnet alpine images

## 1. Obtain ```gssntlmssp.so```
There is no package yet on official alpine repositories yet. For testing purposes you use on from this repo. To build it from source you can use below Dockerfile. You can replace base image with whatever you are going to use in your app.
**Please note - it will only work on Alpine 3.11 or above**
```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine
RUN apk add --no-cache git curl tini util-linux bash neovim openssl
RUN apk add --no-cache make m4 autoconf automake gcc g++ krb5-dev openssl-dev gettext-dev  
RUN apk add --no-cache libtool libxml2 libxslt libunistring-dev zlib-dev samba-dev

RUN git clone https://github.com/gssapi/gss-ntlmssp
WORKDIR gss-ntlmssp
RUN autoreconf -f -i
RUN ./configure --without-manpages --disable-nls
RUN make install
```
Once image is ready, just run temporary interactive container and copy **gssntlmssp.so** file (from .libs or /usr/local/lib/gssntlmssp)

## 2. Install on other containers/images
### 2.1 Make sure you are using Alpine >= 3.11
### 2.2 copy ```gssntlmssp.so``` to any location (e.g. /usr/local/lib/gssntlmssp)
### copy mech.ntlmssp.conf file to /usr/etc/gss/mech.d (make sure it's refering to proper ```gssntlmssp.so``` location)
### 2.3 install dependencies - on dotnet sdk images it should be sufficient to add **libwbclient**. If not try to add some others:
```bash
apk add libwbclient libunistring libssl1.1 zlib libc6-compat
```
### 2.4 you can test if it work using powershell (comes with dotnetsdk):
```Powershell
irm https://myntlmsite.com -Credential (Get-Credential MyNtlmUser)
```
