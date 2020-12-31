# Portus Clair Registry on Docker compose

## The ip address

Pour cet exemple vous aurez besoins de votre ip à de multiple endroit. 
En effet, vous devrez changer l'ip mise dans le fichier .env du docker-compose, 
ainsi que dans le fichier `nginx/nginx.conf'.

## Certificates

Dans cet exemple l'objectif est d'utiliser des certificats auto-signé. 
Bien sur cela ne doit pas être intégré à la production mais cela était le plus simple 
pour un exemple sur une machine locale et surtout par rapport au la limite de temps
à dispositionpour réaliser le projet. Pour créer votre certificat auto-signé, 
vous pouvez utiliser la commande suivante:

```bash
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout portus.key -out portus.crt
```

```bash
$ echo "subjectAltName = IP:123.123.123.123" > extfile.cnf #You can use your own ip here too
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

Après ça, vous pouvez simplement déplacer les fichiers ``portus.key`` et ``portus.crt``
dans le repertoire /secrets.


## The setup

### Secure example

Dans cet exemple on utilise un container NGinx comme proxy entre les containers Portus
et le Registry. La communication est toujours chiffré cia ssl, bien que non nécessaire
mais plus sécurisé. Cette infrastructure a pour conséquence que le Registry et le Portus 
utiliserons la même ip de la machine. de manière plus technique:
- Lors de la première mise en marche du Registry dans Portus, vous devrez spécifier que vous aurez 
coher la boite "Use SSL" après avoir rentré votre ip.
- depuis le CLI, les images docker devront être prefixé avec l'ip, mais sans spécifier le port
(e.g. "255.255.255.255/opensuse/amd64:latest")

https://media.discordapp.net/attachments/786291827463553024/793981308145500170/Capture_decran_2020-12-31_a_00.18.09.png?width=1616&height=910

Après une attention particulière vous est demandée, en effet il ne faudra pas
déployer ces fichiers à l'aveugle dans votre cluster: Vous devrez d'abord les revoirs
afin d'appliquer les correctifs correspondant à vos besoins. Par exemple, 
si vous avez déjà un LDAP vous pouvez l'intégrer à cette infrastructurePortus 
Again, as advertised [here](../README.md), take the word "secure" with a grain
of salt. Do *not* deploy these files blindly into your cluster: review them
first, and make all the changes you need to fit your purpose.



## Difficultés rencontrés
