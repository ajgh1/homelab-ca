# homelab-ca
Scripts to create a simple Certificate Authority suitable for a home
lab environment.

# Install

Put the scripts in a location that can be secured, i.e. a flash drive
or other removable media that can be stored securely, preferably on an
encrypted filesystem.  You will be making a root certificate and its
private key, which can be used to create certificates the devices on
your home network will trust.  **Anyone with access to the root
private key will be able to create trusted certificates.**

Make sure both the `mkCA` and `mkcert` commands are executable by the
user that will create the certificate authority and end-entity
certificate and key files.  This most likely should be your root or
other administration-level user, as again, access to your CA's root
key should be restricted.

# Setup Your Certificate Authority

## Root Certificate Parameters

To set up your certificate authority (CA), you will first need to
set some of the parameters for the CA in the `cavars` file.  These
parameters will be incorporated into your CA's root certificate.

Example `cavars` file:
```
# name of CA
CANAME="My-RootCA"

# organization
O="My Homelab"

# city
L="Cityville"

# state/province
ST="PA"

# two-digit country code
C="US"
```

## Generate Root Certificate and Private Key

Once the `cavars` contains the parameters you want for your root
certificate, run the `mkCA` command:

```
./mkCA
```

to create a root private key, a certificate signing request (CSR) for
your root certificate, and to sign the CSR with the private key.  The
`mkCA` script will use the values you set in the `cavars` file to
create the certificate.  You will be prompted for a password for the
private key; make sure the password is long enough and random enough
to make it relatively secure; anyone with access to the private key
and this password will be able to create trusted certificates! 
(Also, do keep this password in a safe place; without it, *you* will
not be able to create certificates either!)

Once the `mkCA` command has created your CA's root private key and
signed certificate, it will copy these files, the `mkcert` command,
and your `cavars` file to a new directory with the same name as your
CA (the value of `$CANAME` in the `cavars` file).  At this point,
your mini CA is set up and ready to generate certificates for your
homelab network.

# Generate Certificates

Once your CA has been set up, you can simply change to the CA
directory created above and use the `mkcert` command:

```
./mkcert www.example.lan
```

This command will make a new certificate and private key in PEM
format.  You will be prompted for the password to your CA's root
private key so your new certificate can be signed by your CA.

You may specify as many fully-qualified domain names on the command
line as you like.  You may also specify a wildcard domain e.g.
\*.example.lan, to cover your homelab's entire domain.  For example:

```
./mkcert www.example.lan media.example.lan *.example.lan
```

The `mkcert` script will generate a certificate valid for all of the
given domain names, and also the equivalent short names, so devices on
your homelab network can be referenced without their domain names and
still have validate connections i.e. you can refer to "www.example.lan"
as "www" and, if your DNS is setup correctly, the TLS connection will
be valid.

Once the `mkcert` script finishes, there will be three new files in
in the directory, prefixed with the first hostname you specified on 
the `mkcert` command line:

- a `.cert.pem` file, containing the generated certificate,
- a `.csr.pem` file, containing the certificate's signing request, and
- a `.key.pem` file, containing the certificate's private key.

You should be able to take the cert and key files and incorporate them
into your homelab's web, proxy, or other server that uses TLS.

# Configuring Clients to Trust Your CA

The `$CANAME.cert.pem` file contains your CA's trusted root
certificate.  Add that file to your clients' store of trusted CA
certificates, and they will be able to trust any end-entity
certificate you create.  The steps for adding the root certificate
will vary by client operating system.
