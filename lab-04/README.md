# Lab 04 - Dockerfile best-practices

The goal when building your own container images is to keep them as small as
possible.  The main reason is of course that the smaller the image, the faster
it can be distributed (pulled/pushed).  Another reason why it is important to
keep an image as small/minimal as possible is that the fewer packages are
installed in the image, the smaller the attack surface of the image (less
possible vulnerabilities).

## Task 1: Looking for the right base image (FROM)

Base images differ greatly in size, so it is important to look at your
requirements when choosing a base image.

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              5cb3aa00f899        11 hours ago        5.53MB
busybox             latest              d8233ab899d4        3 weeks ago         1.2MB
ubuntu              latest              47b19964fb50        4 weeks ago         88.1MB
centos              latest              1e1148e4cc2c        3 months ago        202MB
```

Because of its small size, the alpine image is used as base image for a lot of
the official images.  But which base image you should use totally depends on
your requirements and use-case.  Often specific base images (for example RHEL)
have to be used within companies.

It has already been mentioned before, but it recommened to always use a specific
version (tag) of the base image (and not `latest`).  Not doing this will prevent
you from creating reproduceable images.  For example:

```
FROM centos:7.6.1810
```

## Task 2: Using labels (LABEL)

Labels are often forgotten, but they can be very handy to set some metadata on
your image.  Most commonly there is a label for the maintainer of the image, but
you can basically add whatever metadata you want using labels.

```
FROM centos:7.6.1810

LABEL maintainer="steven.trescinski@gluo.be"
LABEL licence="GPL"
```

As seen in the previous lab you can use the `podman inspect` command to fetch
the labels of an image.

## Task 3: Running commands (RUN)

If you want to run commands (to install packages for example) in the
container/image we will need to use `RUN` in our Dockerfile.  Important to know
is that every `RUN` statement in your Dockerfile will result in an additional
layer in your container image.  So it is best practice to work as few `RUN`
statements as possible in your Dockerfile.

What you do not want to do is the following:

```
FROM centos:7.6.1810

LABEL maintainer="steven.trescinski@gluo.be"
LABEL licence="GPL"

RUN yum -y update
RUN yum -y install elinks
RUN yum -y install wget
RUN yum clean all
```

Create a `Dockerfile` with the above content and build and image from it:

```
podman image build -t centos_multiple_run:v1 .
```

Now we will do the same, but this case with everything stringed into a single
`RUN` statement. Edit your existing `Dockerfile` so it looks like this:

```
FROM centos:7.6.1810

LABEL maintainer="steven.trescinski@gluo.be"
LABEL licence="GPL"

RUN yum -y update && \
    yum -y install elinks wget && \
    yum clean all
```

Build a new image:

```
podman image build -t centos_single_run:v1 .
```

When we compare the two images we can see that there is more than 200MB
difference between these two images:

```
podman image ls

---

REPOSITORY                                  TAG         IMAGE ID      CREATED         SIZE
localhost/centos_single_run                 v1          05daa6da27d0  8 seconds ago   412 MB
localhost/centos_multiple_run               v1          2ea8796ae222  53 seconds ago  863 MB
```

## Task 4: Optimally using the build cache

As already mentioned, each line in a Dockerfile generates a new layer in the
resulting image.  Docker tries to be as efficient as possible when building
images.  So whenever possible it tries to cache as many layers as possible, but
as soon as one layer cannot be used from the cache all next layers will not be
used from the cache either.  This could lead to quite some differences in build
times.

Create a Dockerfile with the following content:

```
FROM centos:7.6.1810

LABEL maintainer="steven.trescinski@gluo.be"
LABEL licence="GPL"

ARG hello_world_string

RUN touch /hello-world.txt && \
    echo "${hello_world_string}" > /hello-world.txt

RUN for i in {0..30}; do echo "$i"; sleep 1; done; echo

CMD ["cat", "/hello-world.txt"]
```

Build two images from the above Dockerfile (we are using different variables):

```
podman image build --build-arg hello_world_string='Hello world 1!' -t centos_hello_world_1 .
podman image build --build-arg hello_world_string='Hello world 2!' -t centos_hello_world_2 .
```

By simply switching the order of the two `RUN` lines and the `ARG` line we can
tremendously speed up the builds, change the content of your `Dockerfile` with this:

```
FROM centos:7.6.1810

LABEL maintainer="steven.trescinski@gluo.be"
LABEL licence="GPL"

RUN for i in {0..30}; do echo "$i"; sleep 1; done; echo

ARG hello_world_string

RUN touch /hello-world.txt && \
    echo "${hello_world_string}" > /hello-world.txt

CMD ["cat", "/hello-world.txt"]
```

Build two new images from the new Dockerfile (we are again using different
variables):

```
podman image build --build-arg hello_world_string='Hello world 3!' -t centos_hello_world_3 .
podman image build --build-arg hello_world_string='Hello world 4!' -t centos_hello_world_4 .
```

Because we are making changes to the `RUN` statement referencing the
`${hello_world_string}` variable all the lines below it will not be taken from
cache and will slow the build.  The initial build is still slow (you will see
the counter), but the build  after that will be a lot quicker as the cache is
used more optimally!

## Task 5: Dockerfile user

In this task we will see how you can use the `USER` instruction in your 
`Dockerfile`to change the user that the container will run as.

First of all we will run the default `nginx` image, we will use the `whoami` 
command to determine the user that the container runs as:

```
podman container run nginx whoami

---

root
```

You will see that the container runs as `root`.  You can also see this clearly 
when you run a `bash` shell inside the container:

```
podman container run -ti nginx bash

---

root@1334bbf9f148:/#
```

Above you see clearly that the prompt changes to reflect the fact that you are 
running as the `root` user.

To change the user that the container will run as we will need to use the `USER` 
instruction in our Dockerfile when building an image.  Create the following 
Dockerfile:

```
FROM nginx

USER nginx
```

In the above Dockerfile we are simply starting from the `nginx` image and 
changing the user to `nginx`.

Make a `my_nginx` image from the above Dockerfile:

```
podman image build -t my_nginx .

Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> 53f3fd8007f7
Step 2/2 : USER nginx
 ---> Running in f4a3cf9ba2cf
Removing intermediate container f4a3cf9ba2cf
 ---> 146d0d8f33fd
Successfully built 146d0d8f33fd
Successfully tagged my_nginx:latest
```

Now run the `whoami` command on the newly created image to see which user the 
container is running as:

```
podman container run my_nginx whoami

---

nginx
```

Or run the command below to enter a shell inside the container:

```
podman container run -ti my_nginx bash

---

nginx@5c42d03e77fd:/$
```

It is important to note that the container will run as the user that is 
referenced in the **LAST** `USER` instruction.  As an example edit your 
Dockerfile as below:

```
FROM nginx

USER nginx

USER root
```

Build a new image an check which user this container will be running as:

```
podman image build -t my_root_nginx .

Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM nginx
 ---> 53f3fd8007f7
Step 2/3 : USER nginx
 ---> Using cache
 ---> 146d0d8f33fd
Step 3/3 : USER root
 ---> Running in e65753fe4a8a
Removing intermediate container e65753fe4a8a
 ---> e2b67d2f5677
Successfully built e2b67d2f5677
Successfully tagged my_root_nginx:latest
```

Verify that the container will be running as the `root` user:

```
podman container run my_root_nginx whoami

---

root
```

The above example show the importance to always switch back to a non-root user 
(for security purposes) after you needed to run something as the root user (for 
example to install additional packages):

```
FROM nginx

USER root

RUN apt-get update && \
    apt-get -y install elinks && \
    rm -rf /var/lib/apt/lists/*

USER nginx
```

## Task 6: ENTRYPOINT vs. CMD  

In a nutshell, `CMD` sets default command and/or parameters, which can be 
overwritten from the command line when podman runs.

`ENTRYPOINT` configures a container that will run as an executable.

So let's start with creating an image with an `ENTRYPOINT` that does `echo Hello
World!`.

```
FROM nginx

ENTRYPOINT ["/bin/echo" , "Hello World!"]
```

Build the `hello-world` image with this `Dockerfile`.

```
podman image build -t hello-world .

---

Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> 53f3fd8007f7
Step 2/2 : ENTRYPOINT ["/bin/echo" , "Hello World!"]
 ---> Running in 50c122fdb47c
Removing intermediate container 50c122fdb47c
 ---> 44314b2c9cc9
Successfully built 44314b2c9cc9
Successfully tagged hello-world:latest
```

The image is now created with an `ENTRYPOINT` this means that we won't be able 
to overwrite the initial `command` the container is going to do. It is possible 
to add more arguments to this command though.

Without any argument:

```
podman container run -ti hello-world

---

Hello World!
```

With an additional argument:

```
podman run -ti hello-world argument

---

Hello World! argument
```

So this means that if you are going to do `-ti bash` you are not actually
overwriting the `echo` command but you are just adding the `bash` string to the
`echo` command.

```
podman container run -ti hello-world bash

Hello World! bash
```

> NOTE: there is a way to override the entrypoint using the `--entrypoint` 
> argument like so: `podman container run --entrypoint "bash" -ti hello-world`, this 
> command allows you to override the default entrypoint and replace it with the 
> `bash` command

```
podman container run --entrypoint "bash" -ti hello-world

---

root@e872c46373ff:/# exit
```

This is totally different from the `CMD` line in the `Dockerfile`. Create the
image now with the `CMD` line.

```
FROM nginx

CMD ["/bin/echo" , "Hello World!"]
```

Build the image with the edited `Dockerfile`.

```
podman image build -t hello-world .

---

Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> 53f3fd8007f7
Step 2/2 : CMD ["/bin/echo" , "Hello World!"]
 ---> Running in 042aca86ca67
Removing intermediate container 042aca86ca67
 ---> a2ec1c7081a6
Successfully built a2ec1c7081a6
```

If you will run the image now you will see that you get the same output as the
`ENTRYPOINT` line. This is because we are currently not overwriting the command. 

```
podman container run -ti hello-world

---

Hello World!
```

If we are going to add `bash` to the `run` command you will see that we will
`exec` into the container, thus overwrite the initial command.

```
podman container run -ti hello-world bash

---

root@fa9a3a2a62f1:/#
```

## Task 7: clean up

To clean up everything run the following commands:

```
podman system prune
```

As this is the last lab on `podman`, now is a good time to disable `podman` (as podman runs inside a VM it will use quite a lot of CPU and memory if you leave it running)):

```
podman machine stop
```