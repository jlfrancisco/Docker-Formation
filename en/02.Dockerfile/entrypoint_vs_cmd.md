# ENTRYPOINT and CMD

We will illustrate on several examples the use of ENTRYPOINT and CMD instructions. These instructions are used in a Dockerfile to define the command that will be launched in a container.

## Format

In a Dockerfile, the ENTRYPOINT and CMD statements can be specified in 2 formats:

* shell format, e.g. `ENTRYPOINT /usr/bin/node index.js`.
  A command specified in this format will be executed via a shell present in the image. This can be problematic because signals are not forwarded to the forked processes.

* the exec format, e.g. CMD `["node", "index.js"]`.
  A command specified in this format will not require the presence of a shell in the image. We often use the exec format to avoid problems if no shell is present.

## Re-writing at the execution of a container

ENTRYPOINT and CMD are 2 instructions in the Dockerfile, but they can be overwritten when a container is launched:

* to specify another value for the ENTRYPOINT, we would use the --entrypoint option, for example:
docker container run --entrypoint echo alpine hello`

* to specify another value for CMD, we specify it after the image name, for example:
`docker container run alpine echo hello`

## ENTRYPOINT instruction used alone

Using the ENTRYPOINT instruction alone allows to create a wrapper around the application. We can define a basic command and give it additional parameters, if necessary, when launching a container.

In this first example, you will create a Dockerfile-v1 containing the following instructions:

```
FROM alpine
ENTRYPOINT ["ping"]
```

Then create an image, named ping:1.0, from this file.

```
$ docker image build -f Dockerfile-v1 -t ping:1.0 .
```

Now start a container based on the ping:1.0 image

```
$ docker container run ping:1.0
```

The ping command is run in the container (because it is specified in ENTRYPOINT), which produces the following message:

```
BusyBox v1.26.2 (2017-05-23 16:46:25 GMT) multi-call binary.

Usage: ping [OPTIONS] HOST

Send ICMP ECHO_REQUEST packets to network hosts

        -4,-6 Force IP or IPv6 name resolution
        -c CNT Send only CNT pings
        -s SIZE Send SIZE data bytes in packets (default:56)
        -t TTL Set TTL
        -I IFACE/IP Use interface or IP address as source
        -W SEC Seconds to wait for the first response (default:10)
                        (after all -c CNT packets are sent)
        -w SEC Seconds until ping exits (default:infinite)
                        (can exit earlier with -c CNT)
        -q Quiet, only display output at start
                        and when finished
        -p Pattern to use for payload
```

By default, no host machine is targeted, and each time a container is launched it is necessary to specify an FQDN or an IP. The following command launches a new container by giving it the IP address of a Google DNS (8.8.8.8), we also add the -c 3 option to limit the number of pings sent.

```
$ docker container run ping:1.0 -c 3 8.8.8.8
```

We get the following result.

```
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=37 time=8.731 ms
64 bytes from 8.8.8.8: seq=1 ttl=37 time=8.503 ms
64 bytes from 8.8.8.8: seq=2 ttl=37 time=8.507 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 8.503/8.580/8.731 ms
```

The command launched in the container is therefore the concatenation of the ENTRYPOINT and the command specified when launching the container (everything after the image name).

As we can see in this example, the image we have created is a wrapper around the ping utility and requires additional parameters to be specified when launching a container.

## CMD instructions used alone

In the same way, it is possible to use only the CMD instruction in a Dockerfile, this is very often the approach that is used because it is easier to manipulate CMD instructions than ENTRYPOINT.

Create a Dockerfile-v2 file containing the following instructions:

```
FROM alpine
CMD ["ping"]
```

Create an image, named ping:2.0, from this file.

```
$ docker image build -f Dockerfile-v2 -t ping:2.0 .
```

If we now start a new container, it will run the ping command as it did with the previous example in which only the ENTRYPOINT was defined.

```
$ docker container run ping:2.0
BusyBox v1.26.2 (2017-05-23 16:46:25 GMT) multi-call binary.

Usage: ping [OPTIONS] HOST

Send ICMP ECHO_REQUEST packets to network hosts

        -4,-6 Force IP or IPv6 name resolution
        -c CNT Send only CNT pings
        -s SIZE Send SIZE data bytes in packets (default:56)
        -t TTL Set TTL
        -I IFACE/IP Use interface or IP address as source
        -W SEC Seconds to wait for the first response (default:10)
                        (after all -c CNT packets are sent)
        -w SEC Seconds until ping exits (default:infinite)
                        (can exit earlier with -c CNT)
        -q Quiet, only display output at start
                        and when finished
        -p Pattern to use for payload
```

However, we don't have the same behavior as before, because in order to specify the machine to target, we have to redefine the whole command after the image name.

If we specify only the parameters of the ping command, we get an error message because the command issued in the container cannot be interpreted.

```
$ docker container run ping:2.0 -c 3 8.8.8.8
```

You should then get the following error:

```
container_linux.go:247: starting container process caused "exec: \"-c\": executable file not found in $PATH"
docker: Error response from daemon: oci runtime error: container_linux.go:247: starting container process ca
used "exec: \"-c\": executable file not found in $PATH".
ERRO[0000] error getting events from daemon: net/http: request cancelled
```

The entire command must be redefined, which is done by specifying it after the image name

```
$ docker container run ping:2.0 ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=37 time=10.223 ms
64 bytes from 8.8.8.8: seq=1 ttl=37 time=8.523 ms
64 bytes from 8.8.8.8: seq=2 ttl=37 time=8.512 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 8.512/9.086/10.223 ms
```

## ENTRYPOINT and CMD instructions

It is also possible to use ENTRYPOINT and CMD at the same time in a Dockerfile, which allows both to create a wrapper around an application and to specify a default behavior.

We will illustrate this on a new example and create a Dockerfile-v3 file containing the following instructions:

```
FROM alpine
ENTRYPOINT ["ping"]
CMD ["-c3", "localhost"]
```

Here we define ENTRYPOINT and CMD, the command launched in a container will be the concatenation of these 2 instructions: ping -c3 localhost.

Create an image from this Dockerfile, name it ping:3.0, and launch a new container from it.

```
$ docker image build -f Dockerfile-v3 -t ping:3.0 .
$ docker container run ping:3.0
```

You should then get the following result:

```
PING localhost (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.062 ms
64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.102 ms
64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.048 ms

--- localhost ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.048/0.070/0.102 ms
```

We can override the default command and specify another IP address

```
$ docker container run ping:3.0 8.8.8.8
```

We then get the following result:

```
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=38 time=9.235 ms
64 bytes from 8.8.8.8: seq=1 ttl=38 time=8.590 ms
64 bytes from 8.8.8.8: seq=2 ttl=38 time=8.585 ms
```

You have to do a CTRL-C to stop the container because the -c3 option limiting the number of pings has not been specified.

This allows us to have a default behavior and to easily modify it by specifying another command.