# Prise en compte du cache

## Enoncé

1. Modifiez le code du serveur pong de l’exercice précédent. Vous pouvez par exemple ajouter une instruction qui loggue une chaine de caractère.

2. Construisez une nouvelle image en la taguant *pong:1.1*

3. Qu’observez-vous dans la sortie de la commande de build ?

4. Modifiez le *Dockerfile* pour faire en sorte que les dépendances ne soient pas rebuildées si un changement est effectué dans le code. Créez l'image *pong:1.2* à partir de ce nouveau *Dockerfile*.

5. Modifiez une nouvelle fois le code de l'application et créez l'image *pong:1.3*. Observez la prise en compte du cache

## Correction

1. Nous modifions ici le serveur *nodejs* que nous avons réalisé dans l'exercice précédent.

Le code suivant est le nouveau contenu du fichier *pong.js*

```
var express = require('express');
var app = express();
app.get('/ping', function(req, res) {
    console.log("received");
    res.setHeader('Content-Type', 'text/plain');
    console.log("pong"); // <-- commentaire ajouté
    res.end("PONG");
});
app.listen(80);
```

2. Nous buildons l'image *pong:1.1* avec la commande suivante

```
$ docker image build -t pong:1.1 .
```

3. Nous observons que chaque étape du build à été effectuée une nouvelle fois.
Il serait intéressant de faire en sorte qu'une simple modification du code source ne déclenche pas le build des dépendances.

4. Une bonne pratique, à suivre dans l'écriture d'un *Dockerfile*, est de faire en sorte que les éléments qui sont modifiés le plus souvent (code de l'application par exemple) soient positionnés plus bas que les éléments qui sont modifiés moins fréquemment (liste des dépendances).

On peut alors modifier le Dockerfile de la façon suivante:

```
FROM node:16-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 80
CMD ["npm", "start"]
```

L'approche suivie ici est la suivante:
- copie du fichier *package.json* qui contient les dépendances
- build des dépendances
- copie du code source

On rebuild alors une nouvelle fois l'image en lui donnant le tag *pong:1.2*

```
$ docker image build -t pong:1.2 .
```

Un cache est créé pour chaque étape du build.

1. Nous faisons une nouvelle modification dans le fichier *pong.js*.

```
var express = require('express');
var app = express();
app.get('/ping', function(req, res) {
    console.log("received");
    res.setHeader('Content-Type', 'text/plain');
    console.log("ping-pong"); // <-- modification du commentaire
    res.end("PONG");
});
app.listen(80);
```

Nous lançons alors le build une nouvelle fois en nommant l'image *pong:1.3*.

```
$ docker image build -t pong:1.3 .
```

Nous pouvons alors observer que le cache est utilisé jusqu'au step 4. Il est ensuite invalidé lors du step 5 (instruction COPY) car le daemon Docker a détecté le changement de code que nous avons effectué. La prise en compte du cache permet souvent de gagner beaucoup de temps lors de la phase de build.
