# Cube and Log Dashboard Development Environment

## Getting Started

Install Vagrant and Virtualbox.

Initialize cube and log-dashboard git submodules

```
$ git submodule update --init
```

Start up VM

```
$ vagrant up
```

Install generated self-signed certificate into your keystore.

### OS X
```
$ security add-trusted-cert stunnel/cert.pem
```

### Linux
```
$ certutil -d sql:$HOME/.pki/nssdb -N
$ certutil -d sql:$HOME/.pki/nssdb -A -t C -n "Cube Dev Environment" -i stunnel/cert.pem
```
