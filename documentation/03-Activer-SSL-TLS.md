[Retour menu principal](../README.md)

## 3. Activer SSL/TLS pour un accès HTTPS avec un certificat personnalisé

Il est bien entendu possible de se connecter à Gitlab via HTTPS. Dans notre cas nous allons utiliser un certificat auto-signé mais vous pouvez utiliser un certificat officiel.

Pour générer un certificat auto-signé, suivre les instructions suivantes:

Liens utiles: 

https://www.akadia.com/services/ssh_test_certificate.html

https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them

La première étape est de créer une clef privée RSA. Cette clef est de 2048 bits et encodée en AES256 dans cet exemple mais vous pouvez la personnaliser comme vous le souhaitez. Cette clef est stockée en format PEM pour être lisible en ASCII.

1/ Générer la clef privée
```bash
openssl genrsa -aes256 -out server.key 2048
```

Une fois la clef privée obtenue, le CSR peut être généré. Ce CSR peut ensuite être utilisé de 2 façons. Idéalement il est envoyé à une authorité de certification comme Thawte ou Verisign qui va vérifier l'identité du requeteur et fournir un certificat signé. La seconde option est d'auto-signer le CSR.

Au cours de la génération, différentes informations vont être demandées. Ce sont les attributs X.509 du certificat. L'un d'entre eux est important (CN Common Name e.g., YOUR name). Il est important de faire correspondre ce champs avec le FQDN du serveur qui utilisera le certificat. Par exemple, pour "https://gitlab.example.com" il faut renseigner "gitlab.example.com".

2/ Générer un CSR (Certificate Signing Request)
```bash
openssl req -new -key server.key -out server.csr
```
3/ Retirer la passphrase
```bash
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
```
4/ Générer le certificat auto-signé
```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

Une fois votre certificat obtenu, vous devez avoir plusieurs fichiers:

- server.key
- server.csr
- server.crt

Vous pouvez également générer un certificat et une clef privée en une seule ligne en incluant un SAN (mandatory pour kubernetes 1.19+):

```
openssl req \
        -x509 \
        -newkey rsa:4096 \
        -sha256 \
        -days 3560 \
        -nodes \
        -keyout private.key \
        -out cert.crt \
        -subj '/CN=jfrog-argo.devibm.local' \
        -extensions san \
        -config <( \
      echo '[req]'; \
      echo 'distinguished_name=req'; \
      echo '[san]'; \
      echo 'subjectAltName=DNS:jfrog-argo.devibm.local')
```

Pour ajouter une IP au SAN (peut etre utile dans certains cas) modifier la ligne `subjectAltName` comme ceci:

```
echo 'subjectAltName=DNS:jfrog-argo.devibm.local,IP:10.134.54.98')
```

Pour verifier le CN d'un fichier cert :
```
openssl x509 -noout -subject -in server.pem
```
et son SAN :
```
openssl x509 -noout -text -in cert.crt | grep DNS:
```

Ces fichiers vont nous servir pour la nouvelle configuration de notre serveur Gitlab. Copiez ces fichier dans le même dossier que votre fichier docker-compose. On peut ajouter une ligne dans notre fichier .env pour noter le CN utilisé lors de la génération du certificat, variable qui sera utilisée dans notre fichier docker-compose. Voici le nouveau **.env** et **YAML** qui en découlent:

fichier .env:
```bash
## Version control
DOCKER_REGISTRY=gitlab
IMAGE=gitlab-ce
VERSION=latest

## URLs
HOSTNAME=gitlab.example.com
EXT_URL=https://gitlab.example.com
COMMON_NAME=gitlab.example.com

## VOLUMES
MOUNT_PATH=/srv

## PORTS
HOST_SSH_PORT=2222
HOST_HTTP_PORT=2280
```
fichier docker-compose.yml:
```yml
version: '3'
services:
  gitlab:
    image: '${DOCKER_REGISTRY}/${IMAGE}:${VERSION}'
    restart: always
    container_name: gitlab
    hostname: '${HOSTNAME}'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url '${EXT_URL}:2443'
        letsencrypt['enable'] = false 
        nginx['ssl_certificate'] = "/etc/gitlab/ssl/${COMMON_NAME}.crt"
        nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/${COMMON_NAME}.key"
        gitlab_rails['time_zone'] = 'Europe/Paris'
        gitlab_rails['gitlab_shell_ssh_port'] = ${HOST_SSH_PORT}
        gitlab_rails['env'] = {"SSL_CERT_FILE" => "/etc/gitlab/trusted-certs/server.crt"}
    ports:
      - '${HOST_HTTP_PORT}:80'
      - '2443:2443'
      - '${HOST_SSH_PORT}:22'
    volumes:
      - '${MOUNT_PATH}/gitlab/config:/etc/gitlab'
      - '${MOUNT_PATH}/gitlab/logs:/var/log/gitlab'
      - '${MOUNT_PATH}/gitlab/data:/var/opt/gitlab'  
      - ./server.crt:/etc/gitlab/trusted-certs/server.crt
      - ./server.crt:/etc/gitlab/ssl/${COMMON_NAME}.crt
      - ./server.key:/etc/gitlab/ssl/${COMMON_NAME}.key
```

Si vous changez le port HTTPS, il est important de mapper le port VM avec le port container (ex: 2443 host -> 2433 container)

Voici les changements:

- L'URL change de HTTP vers HTTPS.
- Letsencrypt est désactivé.
- Le certificat et la clef privée sont copiés dans le dossier _**'/etc/gitlab/ssl'**_ du container pour NGINX.
- On indique à NGINX ou trouver le certificat et la clef privée.
- Le certificat est copié dans le dossier _**'etc/gitlab/trusted-certs/'**_ du container.

---------------------------------------------------------------------------------------------------------------------------------

[Retour menu principal](../README.md)

[Suivant](04-Configurer-les-logs.md)
