# Portus Clair Registry on Docker compose

## The hostname

This example needs the hostname in multiple places. All this has been delegated
into Compose's support of the `.env` file. For this reason, you will need to
change hostname set in this file, and also in the `nginx/nginx.conf` file.

## Certificates

This example is set up in a way so you can use self-signed certificates. Of
course this is not something you would want to do in production, but this way we
ease up the task for those who are curious to try it out.
In order to create self-signed certificates, you could use the following command:

```bash
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout portus.key -out portus.crt
```

```bash
$ echo "subjectAltName = IP:123.123.123.123" > extfile.cnf #You can use DNS:domain.tld too
$ openssl genrsa -out secrets/rootca.key 2048 -nodes
$openssl req -x509 -new -nodes -key secrets/rootca.key \
 -subj "/C=US/ST=CA/O=Acme, Inc." \
 -sha256 -days 1024 -out secrets/rootca.crt
$ openssl genrsa -out secrets/portus.key 2048
$ openssl req -new -key secrets/portus.key -out secrets/portus.csr \
 -subj "/C=US/ST=CA/O=Acme, Inc./CN=123.123.123.123"
$ openssl x509 -req -in secrets/portus.csr -CA secrets/rootca.crt -extfile \
 extfile.cnf -CAkey secrets/rootca.key -CAcreateserial \
 -out secrets/portus.crt -days 500 -sha256
```

After that, you can simply move the ``portus.key`` and the ``portus.crt`` files
into the secrets directory.

## The setup

### Secure example

The secure example uses an NGinx container that proxies between the Portus and
the Registry containers. Communication is always encrypted, but note that this
is not strictly necessary. Because of this proxy setup, both Portus and the
registry end up using the same hostname. Practically speaking:

- When setting up the registry for the first time in Portus, you have to check
  the "Use SSL" box and enter the hostname without specifying any ports.
- From the CLI, docker images should be prefixed with the hostname, but without
  specifying any ports (e.g. "my.hostname.com/opensuse/amd64:latest")

Again, as advertised [here](../README.md), take the word "secure" with a grain
of salt. Do *not* deploy these files blindly into your cluster: review them
first, and make all the changes you need to fit your purpose.



## Difficultés rencontrés
