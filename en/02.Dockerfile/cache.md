# Taking into account the cache

## Statement

1. Modify the pong server code from the previous exercise. You can for example add a statement that logs a string.

2. Build a new image by tagging it *pong:1.1*.

3. What do you observe in the output of the build command?

4. Modify the *Dockerfile* to ensure that dependencies are not rebuilt if a change is made in the code. Create the *pong:1.2* image from this new *Dockerfile*.

5. Change the application code again and create the *pong:1.3* image. Observe that the cache is taken into account

## Correction

1. Here we modify the *nodejs* server that we made in the previous exercise.

The following code is the new content of the *pong.js* file

```
var express = require('express');
var app = express();
app.get('/ping', function(req, res) {
    console.log("received");
    res.setHeader('Content-Type', 'text/plain');
    console.log("pong"); // <-- comment added
    res.end("PONG");
});
app.listen(80);
```

2. We build the *pong:1.1* image with the following command

```
$ docker image build -t pong:1.1 .
```

3. We can see that each step of the build has been done once again.
It would be interesting to make sure that a simple modification of the source code does not trigger the build of the dependencies.

4. A good practice, to follow when writing a *Dockerfile*, is to make sure that the elements that are modified the most often (application code for example) are positioned lower than the elements that are modified less frequently (dependencies list).

We can then modify the Dockerfile in the following way:

```
FROM node:16-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY .
EXHIBIT 80
CMD ["npm", "start"]
```

The approach followed here is the following:
- copy the *package.json* file that contains the dependencies
- build the dependencies
- copy the source code

We then rebuild the image once again, giving it the *pong:1.2* tag

```
$ docker image build -t pong:1.2 .
```

A cache is created for each step of the build.

1. We make a new change in the *pong.js* file.

```
var express = require('express');
var app = express();
app.get('/ping', function(req, res) {
    console.log("received");
    res.setHeader('Content-Type', 'text/plain');
    console.log("ping-pong"); // <-- edit comment
    res.end("PONG");
});
app.listen(80);
```

We then run the build again, naming the image *pong:1.3*.

```
$ docker image build -t pong:1.3 .
```

We can then observe that the cache is used until step 4. It is then invalidated at step 5 (COPY instruction) because the Docker daemon has detected the code change we made. Taking the cache into account often saves a lot of time during the build phase.
