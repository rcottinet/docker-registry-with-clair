
## The ip address

Pour cet exemple vous aurez besoins de votre ip à de multiple endroits. 
En effet, vous devrez changer l'ip mise dans le fichier `.env` du docker-compose, pour la creation du certificat,
ainsi que dans le fichier `nginx/nginx.conf`.

## Certificates

Dans cet exemple l'objectif est d'utiliser un certificat auto-signé. 
Bien sûr, cela ne doit pas être intégré à la production mais c'était plus simple 
pour un exemple sur une machine locale et surtout par rapport à la limite de temps
à disposition pour réaliser le projet. Pour créer votre certificat auto-signé, 
vous pouvez utiliser les commandes suivantes:

```bash
$ cd Portus
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
dans le repertoire /secrets (si le certificat n'est pas pris en compte relancez les containers).


A partir de maintenant, vous devez pouvoir lancer les containers par la commande :

```bash
docker-compose up -d
```
Normalement les applications doivent demarrer sans erreurs comme ceci :
https://media.discordapp.net/attachments/786291827463553024/793976388683169832/Capture_decran_2020-12-30_a_23.58.33.png


## The setup

Dans cet exemple on utilise un container NGinx comme proxy entre les containers Portus
et le Registry. La communication est toujours chiffrée via ssl, bien que non nécessaire
mais plus sécurisé. Cette infrastructure a pour conséquence que le Registry et le Portus 
utiliserons la même ip de la machine. de manière plus technique:
- Lors de la première mise en marche du Registry dans Portus, vous devrez spécifier que vous aurez 
coché la boite "Use SSL" après avoir rentré votre ip, sans spécifier le port.
- depuis le CLI, les images docker devront être prefixé avec l'ip, mais sans spécifier le port
(e.g. "255.255.255.255/opensuse/amd64:latest")

Une fois votre Registry ajouté vous devriez pouvoir le voir tel que l'image suivante dans Portus:
https://media.discordapp.net/attachments/786291827463553024/793981308145500170/Capture_decran_2020-12-31_a_00.18.09.png?width=1616&height=910

Puis vous pourrez vous authentifier avec l'utilisateur Portus (ou LDAP si intégré avec le Portus), 
comme ceci:
https://media.discordapp.net/attachments/786291827463553024/793976388683169832/Capture_decran_2020-12-30_a_23.58.33.png

Pour ensuite, que vous puissiez push une image dans votre Registry afin qu'elle soit scanné par Clair:
https://media.discordapp.net/attachments/786291827463553024/793981546524704788/Capture_decran_2020-12-31_a_00.19.07.png

Néanmoins une attention particulière vous est demandée, en effet il ne faudra pas
déployer ces fichiers à l'aveugle dans votre cluster: Vous devrez d'abord les revoirs
afin d'appliquer les correctifs correspondant à vos besoins. Par exemple, 
si vous avez déjà un LDAP vous pouvez l'intégrer à cette infrastructure en suivant cette doc 
(optionnel une authentification étant déjà permise dans Portus):
http://port.us.org/features/2_LDAP-support.html

Nous n'avons malheureusement pas pu finaliser le test pour analyser une image avec Clair. 
Suite à un problème avec le certificat auto-signé qui ne permettaient pas au Registry de notifier 
à Portus et Clair le push d'une nouvelle image:

```bash
time="2020-12-30T23:32:48Z" level=warning msg="httpSink{https://192.168.1.25/v2/webhooks/events%7D encountered too many errors, backing off"

time="2020-12-30T23:32:49Z" level=error msg="retryingsink: error writing events: httpSink{https://192.168.1.25/v2/webhooks/events%7D: error posting: Post https://192.168.1.25/v2/webhooks/events: x509: certificate signed by unknown authority, retrying"

time="2020-12-30T23:32:49Z" level=warning msg="httpSink{https://192.168.1.25/v2/webhooks/events%7D encountered too many errors, backing off"

time="2020-12-30T23:32:50Z" level=error msg="retryingsink: error writing events: httpSink{https://192.168.1.25/v2/webhooks/events%7D: error posting: Post https://192.168.1.25/v2/webhooks/events: x509: certificate signed by unknown authority, retrying"

time="2020-12-30T23:32:50Z" level=warning msg="httpSink{https://192.168.1.25/v2/webhooks/events%7D encountered too many errors, backing off"
```

## Difficultés rencontrés

La première a été le manque de temps pouvant être consacré au projet, ensuite la génération des 
certificats avec la bonne configuration afin de permettre la connexion en HTTPS vers Portus, 
avec bien sur l'analyse clair qui, comme dis précédemment, n'a pas pu aboutir suite au soucis de 
communication avec le certificat auto-signé. Avec plus de temps nous aurions pu finaliser ce test
et automatiser cette analyse en passant de docker-compose à Kubernetes et en utilisant un job qui 
aurait automatisé l'analyse à la suite de chaque livraison sur le registry. Enfin un Web Service 
aurait aussi pu être intégré afin d'avoir une GUI permettant de visualiser les résultats des analyses
Tout en donnant des recommandations permettant de réaliser les fixs.
