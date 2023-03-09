# Open Source Registry

In this lab, we will use the open source registry of Docker.

## Launching

We will use the official registry image available in the Docker Hub.

From the *registry:2.6* image, launch a container specifying that it should run in background and expose its port 5000 on the host machine's port 5000.

```
$ docker container run --name registry -d -p 5000:5000 registry:2.6
```

## List of registry images

The registry exposes an HTTP Rest API on port 5000. Using a GET request on the endpoint */v2/_catalog* we get the list of images present. This list is empty because we haven't uploaded any images to the registry yet.

```
$ curl localhost:5000/v2/_catalog
{ "repositories":[]}
```

## Retrieving an image from the Docker Hub

Download the *nginx* image from the Docker Hub. We will use this image later to send it to the registry launched earlier.

```
$ docker image pull nginx
Using default tag: latest
latest: Pulling from library/nginx
ff3d52d8f55f: Pull complete
226f4ec56ba3: Pull complete
53d7dd52b97d: Pull complete
Digest: sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268
Status: Downloaded newer image for nginx:latest
```

## Image tag in registry format

In order for the image to be uploaded to the registry, it must follow a specific format. The image name must be prefixed by the FQDN and the registry port.

**localhost:5000/NAME:VERSION**

Use the tag command to add a tag to the nginx image we downloaded above. So the tag will start with localhost:5000, we also set the image name, w3 and the version, 1.0.

```
$ docker image tag nginx localhost:5000/w3:1.0
```

List the locally available images. You will see the *registry:2.6* image, the manually uploaded nginx:latest image, and the localhost:5000/w3:1.0 image you just created.

```
$ docker image ls
REPOSITORY TAG IMAGE ID CREATED SIZE
localhost:5000/w3 1.0 958a7ae9e569 2 weeks ago 109MB
nginx latest 958a7ae9e569 2 weeks ago 109MB
registry 2.6 9d0c4eabab4d 5 weeks ago 33.2MB
```

## Upload the image to the registry

Now that the image tag is properly formatted, upload it to your registry.

```
$ docker image push localhost:5000/w3:1.0
The push refers to a repository [localhost:5000/w3]
a552ca691e49: Pushed
7487bf0353a7: Pushed
8781ec54ba04: Pushed
1.0: digest: sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268 size: 948
```

If you list the images in the registry again, you can see that the image tagged *localhost:5000/w3:1.0* is present.

```
curl localhost:5000/v2/_catalog
{"repositories":["w3"]}
```

## Cleanup

Delete the registry with the following command:

```
$ docker container rm -f registry
```

## In summary

This simple example illustrates how the open source registry works. However, there are several points to note:

By default, the Docker daemon cannot communicate with a registry that is not secure (i.e. without TLS), which is the case with the open source registry. There are however 3 possible options:
* change the daemon configuration and restart it with the *--insecure-registry* flag
* use the localhost exception which allows communication via localhost even without the *--insecure-registry* flag. This is the approach we use in this example.
* Create a certificate and public/private key pairs, signed by this certificate, for the daemon and the registry.

When a registry other than the Docker Hub is used, the tags of the images in this registry must be prefixed with **REGISTRY_HOSTNAME:REGISTRY_PORT**.