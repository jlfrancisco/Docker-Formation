# Multistage build

Dans cet exercice, nous allons illustrer le multi stage build

## Rappel

Comme nous l'avons vu, le Dockerfile contient une liste d'instructions qui permet de créer une image. La première instruction est FROM, elle définit l'image de base utilisée. Cette image de base contient souvent beaucoup d'éléments (binaires et librairies) dont l'application finale n'a pas besoin (compilateur, ...). Ceci qui peut impacter de façon considérable la taille de l'image et également sa sécurité puisque cela peut considérablement augmenter sa surface d'attaque. C'est la qu'intervint le multistage build...

## Un serveur http écrit en Go

Prenons l'exemple du programme suivant écrit en Go.

Dans un nouveau répertoire, créez le fichier *http.go* contenant le code suivant. Celui-ci définit un simple serveur http qui écoute sur le port 8080 et qui expose le endpoint /whoami en GET. A chaque requête, il renvoie le nom de la machine hôte sur laquelle il tourne.

```
package main

import (
        "io"
        "net/http"
        "os"
)

func handler(w http.ResponseWriter, req *http.Request) {
        host, err := os.Hostname()
        if err != nil {
          io.WriteString(w, "unknown")
        } else {
          io.WriteString(w, host)
        }
}

func main() {
        http.HandleFunc("/whoami", handler)
        http.ListenAndServe(":8080", nil)
}
```

## Dockerfile _traditionel_

Afin de créer une image pour cette application, créez tout dabord le fichier Dockerfile avec le contenu suivant (placez ce fichier dans le même répertoire que *http.go*):

```
FROM golang:1.17
WORKDIR /go/src/app
COPY http.go .
RUN go mod init
RUN CGO_ENABLED=0 GOOS=linux go build -o http .
CMD ["./http"]
```

Note: dans ce *Dockerfile*, l'image officielle *golang* est utilisée comme image de base,  le fichier source *http.go* est copié puis compilé.

Vous pouvez ensuite builder l'image et la nommer *whoami:1.0*:.

```
$ docker image build -t whoami:1.0 .
```

Listez les images présentes et notez la taille de l'image *whoami:1.0*


```
$ docker image ls whoami
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
whoami       1.0       16795cf36deb   2 seconds ago   962MB
```

L'image obtenue a une taille très conséquente car elle contient l'ensemble de la toolchain du langage Go. Or, une fois que le binaire a été compilé, nous n'avons plus besoin du compilateur dans l'image finale.


## Dockerfile utilisant un build multi-stage

Le multi-stage build, introduit dans la version 17.05 de Docker permet, au sein d'un seul Dockerfile, d'effectuer le process de build en plusieurs étapes. Chacune des étapes peut réutiliser des artefacts (fichiers résultant de compilation, assets web, ...) créés lors des étapes précédentes. Ce Dockerfile aura plusieurs instructions FROM mais seule la dernière sera utilisée pour la construction de l'image finale.

Si nous reprenons l'exemple du serveur http ci dessus, nous pouvons dans un premier temps compiler le code source en utilisant l'image *golang* contenant le compilateur. Une fois le binaire créé, nous pouvons utiliser une image de base vide, nommée scratch, et copier le binaire généré précédemment.

Remplacer le contenu du fichier Dockerfile avec les instructions suivantes:

```
FROM golang:1.17 as build
WORKDIR /go/src/app
COPY http.go .
RUN go mod init
RUN CGO_ENABLED=0 GOOS=linux go build -o http .

FROM scratch
COPY --from=build /go/src/app .
CMD ["./http"]
```

> L'exemple que nous avons utilisé ici se base sur une application écrite en Go. ce langage a la particularité de pouvoir être compilé en un binaire static, c'est à dire ne nécessitant pas d'être "linké" à des librairies externes. C'est la raison pour laquelle nous pouvons partir de l'image scratch. Pour d'autres langages, l'image de base utilisée lors de la dernière étape du build pourra être différente (alpine, ...)


Buildez l'image dans sa version 2 avec la commande suivante.

```
$ docker image build -t whoami:2.0 .
```

Listez les images et observez la différence de taille entre celles-ci:

```
$ docker image ls whoami
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
whoami       2.0       0a97315aeaaa   6 seconds ago   6.07MB
whoami       1.0       16795cf36deb   2 minutes ago   962MB
```

Lancez un container basé sur l'image *whoami:2.0*

```
$ docker container run -p 8080:8080 whoami:2.0
```

A l'aide de la commande curl, envoyez une requête GET sur le endpoint exposé. Vous devriez avoir, en retour, l'identifiant du container qui a traité la requète.

```
$ curl localhost:8080/whoami
7562306c6c5e
```

Pour cette simple application, le multistage build a permit de supprimer les binaires et librairies dont la présence est inutile dans l'image finale. L'exemple d'une application écrite en **go** est extrème, mais le multistage build fait partie des bonnes pratiques à adopter pour de nombreux languages de développement.