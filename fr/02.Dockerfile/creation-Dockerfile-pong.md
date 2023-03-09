# Dockerizez un serveur web simple

## Enoncé

1. Créer un nouveau répertoire et développez un serveur HTTP qui expose le endpoint */ping* sur le port 80 et répond par PONG.

:fire: pour gagner du temps, vous pouvez utiliser l'exemple en Node.js de la correction :)

2. Dans le même répertoire, créez le fichier Dockerfile qui servira à construire l'image de l'application. Ce fichier devra décrire les actions suivantes

- spécification d'une image de base
- installation du runtime correspondant au langage choisi
- installation des dépendances de l’application
- copie du code applicatif
- exposition du port d’écoute de l’application
- spécification de la commande à exécuter pour lancer le serveur

3. Construire l’image en la taguant *pong:v1.0*

4. Lancez un container basé sur cette image en publiant le port 80 sur le port 8080 de la machine hôte

5. Tester l'application

6. Supprimez le container


## Correction

1. Dans cette exemple nous avons choisi Node.js

Le code suivant est le contenu du fichier *pong.js* (code du serveur web)

```
var express = require('express');
var app = express();
app.get('/ping', function(req, res) {
    console.log("received");
    res.setHeader('Content-Type', 'text/plain');
    res.end("PONG");
});
app.listen(80);
```

Le fichier *package.json* contient les dépendances de l'application (dans le cas présent, il s'agit de la librairie *expressjs* utilisée pour la réalisation d'application web dans le monde NodeJs.

```
{
  "name": "pong",
  "version": "0.0.1",
  "main": "pong.js",
  "scripts": {
    "start": "node pong.js"
  },
   "dependencies": {  "express": "^4.14.0" }
}
```

2. Une version du Dockerfile pouvant être utilisé pour créer une image de l'application

```
FROM node:16-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 80
CMD ["npm", "start"]
```

Note: il y a toujours plusieurs approches pour définir le fichier *Dockerfile* d'une application. On aurait par exemple pu partir d'une image de base comme *ubuntu* ou *alpine*, et installer le runtime *nodejs* comme dans l'exemple ci-dessous:

```
FROM alpine:3.14
RUN apk add -u npm
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 80
CMD ["npm", "start"]
```

3. La commande suivante permet de construire l'image à partir du *Dockerfile* précédent

```
docker image build -t pong:1.0 .
```

4. La commande suivante permet de lancer un container basé sur l'image *pong:1.0* et le rend accessible depuis le port 8080 de la machine hôte.

```
docker container run --name pong -d -p 8080:80 pong:1.0
```

Note: assurez-vous que le port n'est pas déjà pris par un autre container ou une autre application. Si c'est le cas, utilisez par exemple le port 8081 dans la commande ci-dessus.

5. Afin de tester le serveur ping, il suffit d'envoyer une requête GET sur le endpoint /ping et vérifier que l'on a bien PONG en retour

```
curl localhost:8080/ping
```

6. Supprimez le container avec la commande suivante

```
docker rm -f pong
```
