# Lab 03 - Additional Podman operations

In this lab we will have a look at some additional Podman operations commands.

## Task 1: Look at what containers are (and were) running

Start a couple of containers:

```
podman container run -d --name nginx_1 -p 8080:80 nginx
podman container run -d --name nginx_2 -p 8081:80 nginx
podman container run --name triangle_small gluobe/draw-triangle:v1 25
podman container run --name triangle_large gluobe/draw-triangle:v1 50
```

To see which containers are currently running use the following command:

```
podman container ls
```

You should see the two nginx containers running:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
3357b8b9b24c        nginx               "nginx -g 'daemon of…"   2 seconds ago       Up 1 second         0.0.0.0:8081->80/tcp   nginx_2
0902eab457a2        nginx               "nginx -g 'daemon of…"   54 seconds ago      Up 53 seconds       0.0.0.0:8080->80/tcp   nginx_1
```

You might wonder where you can see the containers that were started from the
`gluobe/draw-triangle:v1` image.  The `podman ps` command only shows the
containers that are actively running.  To see containers that are stopped or
exited you will need to add the `-a` flag to the command:

```
podman container ls -a
```

The output should look something like:

```
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                     PORTS                  NAMES
3357b8b9b24c        nginx                     "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes               0.0.0.0:8081->80/tcp   nginx_2
a9d5500b0b50        gluobe/draw-triangle:v1   "bash /draw-triangle…"   5 minutes ago       Exited (0) 5 minutes ago                          friendly_lamarr
323a2956f124        gluobe/draw-triangle:v1   "bash /draw-triangle…"   5 minutes ago       Exited (0) 5 minutes ago                          sad_joliot
0902eab457a2        nginx                     "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes               0.0.0.0:8080->80/tcp   nginx_1
```

Looking at the `STATUS` column you can see if a container is running (it will
say how long the container is up for) or if it has exited.

## Task 2: Looking at containers logs

In the above task we have learned how to see which containers are and were
running.  But how can we see what those containers are/were actually doing, in
other words how can we see the logs of containers?

Before we can look at the logs, we first need to generate some logs of course,
so in your browser surf to both http://localhost:8080 and http://localhost:8081
and hit refresh a couple of time to generate some access logs.

To see the logs we are using the `podman logs` command, we only need to specify
the container name or ID, for example:

```
podman container logs nginx_2
```

Instead of using the container name you could also use the container ID, so in
the above example this would be:

```
podman container logs 3357b8b9b24c
```

Or we could use the short ID:

```
podman container logs 33
```

Important to note is that you cannot only see logs of running containers, you
can of course also check the logs of containers that have exited.  This is very
useful if you are troubleshooting why a specific container is not running
anymore:

```
podman container logs triangle_large
```

## Task 3: Stopping and starting containers

To stop a running container we can use the `podman stop` command, again followed
by its name or ID:

```
podman container stop nginx_1
```

When we do a `podman container ls` again we see that we only have one container running:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
3357b8b9b24c        nginx               "nginx -g 'daemon of…"   28 minutes ago      Up 28 minutes       0.0.0.0:8081->80/tcp   nginx_2
```

To start the container again you can use the `podman container start` command:

```
podman container start nginx_1
```

After issue that command you should see two containers running again, pay
special attention to the status as you will see that `nginx_1` is only up for
a couple of seconds:

```
podman container ls

---

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
3357b8b9b24c        nginx               "nginx -g 'daemon of…"   29 minutes ago      Up 29 minutes       0.0.0.0:8081->80/tcp   nginx_2
0902eab457a2        nginx               "nginx -g 'daemon of…"   30 minutes ago      Up 1 second         0.0.0.0:8080->80/tcp   nginx_1
```

Container names are unique, this is something that you need to keep in mind when
you stop a container or when a container is terminated automatically.  What
often happens is that you try to start a container with a specific name and
Docker does not allow you to do this because it claims that that name is already
is use.  Let us similate this issue.

Stop the `nginx_1` container:

```
podman container stop nginx_1
```

Next, we try to relaunch that container:

```
podman container run -d --name nginx_1 -p 8080:80 nginx
```

We are prompted with the following error message:

```
Error: error creating container storage: the container name "nginx_1" is already in use by "e5cb59c823f439abd19eda4d54b40411a1222ba8ffcae17afbbfd6b1e6859c8b". You have to remove that container to be able to reuse that name.: that name is already in use
```

This is because the stopped container still exists under that exact name.  The
sollution to this is to not start a brand new container but simply restart the
existing exited container:

```
podman container start nginx_1
```

## Task 4: Accessing a running container

Containers are immutable by nature, but nevertheless it is sometimes
usefull to access a running container (this is similar to SSH'ing into a running
virtual machine).

To access the `nginx_1` container we use the following command:

```
podman container exec -ti nginx_1 bash
```

Notice that your prompt will change when you execute the above command, this
indicates that you are now inside your container.  You can now browse the
container's filesytem, look at configuration files,...  However always keep in
mind that containers are immutable, so do not make any changes from within the
container as you will loosse these as soon as the container is terminated or
restarted.

```
MacSteven:~ trescst$ podman exec -ti nginx_1 bash
root@0902eab457a2:/#
```

To exit the container, simply enter the `exit` command (you will know that you
have exited your container when your prompt is back to normal).

## Task 5: Inspecting containers and images

Sometimes you would want some additional detailed information about a container
and/or image.  For this the `podman inspect` commmand can be used.

The output is pretty lengthy but contains a lot of usefull information about
either the container of the image.

```
podman inspect nginx_1

---

[
    {
        "Id": "0902eab457a2881a12ed5149799af85e515460a42720262e5e15998e97d79263",
        "Created": "2019-03-08T05:55:46.578041824Z",
        "Path": "nginx",
        "Args": [
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 3293,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-03-08T06:51:23.195322991Z",
            "FinishedAt": "2019-03-08T06:35:25.817618675Z"
        },

<...redacted..>

            "SandboxKey": "/run/user/1000/netns/netns/80fd8c9a5fbf",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "5d7841039488450bb6af0b6646b8d4d5e3d488769b07548929df79624b3f2ce9",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "23136b6180660f4456ad3c30b53ab43c18d2d9711d909de2924db3b9c73d2cb8",
                    "EndpointID": "5d7841039488450bb6af0b6646b8d4d5e3d488769b07548929df79624b3f2ce9",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

You can do the same for an image:

```
podman image inspect nginx

---

[
    {
        "Id": "sha256:d6be02065b0de8aa71b5d9e96e3861745c3ec3e3229f4ef774c9cb57ca2af76b",
        "RepoTags": [
            "gluobe/draw-triangle:v1"
        ],
        "RepoDigests": [],
        "Parent": "sha256:3d7129358ffe54353dde861c2c8c23e7516c63bd31601b140eafe87e64dff3b5",
        "Comment": "",
        "Created": "2019-03-07T14:40:07.544509399Z",
        "Container": "a7d23d7f480a2fea98d9f572108d2394585d02cd9d829c52013bab3e71749a59",
        "ContainerConfig": {
            "Hostname": "a7d23d7f480a",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "ENTRYPOINT [\"bash\" \"/draw-triangle.sh\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:3d7129358ffe54353dde861c2c8c23e7516c63bd31601b140eafe87e64dff3b5",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "bash",
                "/draw-triangle.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "steven.trescinski@gluo.be"
            }
        },

<...redacted...>

        "Architecture": "amd64",
        "Os": "linux",
        "Size": 9559052,
        "VirtualSize": 9559052,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/home/$CURRENT_USER/.local/share/containers/storage/overlay/50420f58ef88469a9f3c3213b988ae3893f24b40227effe923f4c868e7ad8564/diff:/home/$CURRENT_USER/.local/share/containers/storage/overlay/0a3cb54017c2bfbba2b89c3badecfd18bd2e2ae29f06e954fcc73ebf62eb4bf2/diff",
                "MergedDir": "/home/$CURRENT_USER/.local/share/containers/storage/overlay/9457b5820801d34c7395e3514298349c8193d425a5aac7e94762de2b66011369/merged",
                "UpperDir": "/home/$CURRENT_USER/.local/share/containers/storage/overlay/9457b5820801d34c7395e3514298349c8193d425a5aac7e94762de2b66011369/diff",
                "WorkDir": "/home/$CURRENT_USER/.local/share/containers/storage/overlay/9457b5820801d34c7395e3514298349c8193d425a5aac7e94762de2b66011369/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:503e53e365f34399c4d58d8f4e23c161106cfbce4400e3d0a0357967bad69390",
                "sha256:a413d5da852a5a5c12957ee1ed8f09ed6c5dbe7595f4f6f3de409cde3e88b990",
                "sha256:09fe3af2184ac4ede8b15a88897c7c7680e12ad42e6e384ff46f1960b1949f86"
            ]
        },
        "Metadata": {
            "LastTagTime": "2019-03-07T14:40:07.593543108Z"
        }
    }
]
```

In most cases however we only need some specific information about either the
container or the image.  For this we can use the `-f` option.

Example to get the IP address of a container:

```
podman inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx_1
```

Example to get the maintainer of an image:

```
podman image inspect -f '{{ index .Config.Labels "maintainer"}}' gluobe/draw-triangle:v1
```

## Task 5: Deleting containers and images

So far we have focussed on creating, stopping and starting containers.  But we
obviously sometimes need to get rid of some containers and images.

For this we have a couple of options, to manually delete containers and images
we have the `podman rm` and `podman image rm` commands.

One important thing to note is that before you can delete an image, you will
first need to delete all the running/terminated containers that are using that
image as Podman will prevent you from deleting images that are still in use, for
example `podman image rm gluobe/draw-triangle:v1` will result in the following error:

```
podman image rm gluobe/draw-triangle:v1

---

Error: Image used by 87b4b1d97d23cee8a0ef8180d4d01df06239873238073a76013d4c3bc21b96b3: image is in use by a container
```

So we first need to remove the containers that are referencing this image:

```
podman container rm triangle_small
podman container rm triangle_large
```

After that the `podman image rm gluobe/draw-triangle:v1` command should work:

```
podman image rm gluobe/draw-triangle:v1

---

Untagged: docker.io/gluobe/draw-triangle:v1
Deleted: 26b13e8a4c2273d9689f735549a35a400ac6271661dc3a0f679be9c2ebbe7995
```

## Task 6: clean up

If you are running a lot of different (and especially large images and
containers) it could happen that the filesystem will run out of space.  To clean
up all unused containers/images you can use the following command:

```
podman system prune
```