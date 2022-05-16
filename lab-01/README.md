# Lab 01 - Running your first Podman containers

In this first lab we will focus on running containers from already existing
images.

## Task 1: Hello world!

To run your very first (one-off) container, simply execute the following command 
in your terminal:

```
podman run gluobe/hello-world
```

When succesful you will be prompted with the following output (read the output
carefully to better understand all the technical things that happened behind the 
scenes in order to produce this output):

```
##############################################################################
###                               Hello world!                             ###
##############################################################################


                                   ##         .
                             ## ## ##        ==
                          ## ## ## ## ##    ===
                      /"""""""""""""""""\___/ ===
                     {                       /  ===-
                      \______ O           __/
                        \    \         __/
                         \____\_______/


##############################################################################

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "gluobe/hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
 5. As soon as the executable finished the container is terminated by the Docker
    daemon.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/#

##############################################################################
```

## Task 2: running an interactive container

The container above ran a single command and was terminated as soon as that 
command was completed.  Often we want to interact with our container, to do so 
we will need to run an interactive container.

To run an interactive container we need to add some additonal options and 
parameters to the basic `podman run` command that we previously used:
* `-ti` to make the container interactive
* `bash` to  specify the command

Execute the command below to run an interactive Ubuntu container.  After the Ubuntu image has been downloaded you should see that your prompt will 
automatically change.

```
podman run -ti ubuntu bash

---

root@5b91b65eb29f:/#
```

You are now "inside" the Ubuntu container and can now do pretty much everything
that you would be able to do on a regular Ubuntu system.

Check the Ubuntu version with the command below:

```
root@5b91b65eb29f:/# cat /etc/lsb-release

---

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.1 LTS"
```

Next update the package list:

```
root@5b91b65eb29f:/# apt-get update

---

Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]                 
Get:3 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [61.3 kB]
Get:4 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 Packages [61.0 kB]
Get:5 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [84.2 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [109 kB]                   
Get:7 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [90.7 kB]
Get:8 http://archive.ubuntu.com/ubuntu jammy/restricted amd64 Packages [164 kB]
Get:9 http://archive.ubuntu.com/ubuntu jammy/universe amd64 Packages [17.5 MB]
Get:10 http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages [1792 kB]
Get:11 http://archive.ubuntu.com/ubuntu jammy/multiverse amd64 Packages [266 kB]
Get:12 http://archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 Packages [68.6 kB]
Get:13 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [157 kB]
Get:14 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [97.5 kB]
Fetched 20.8 MB in 4s (4979 kB/s)                        
Reading package lists... Done
```

```
root@5b91b65eb29f:/# apt-cache search elinks

---

elinks - advanced text-mode WWW browser
elinks-data - advanced text-mode WWW browser - data files
elinks-doc - advanced text-mode WWW browser - documentation
```

To "exit" our interactive Ubuntu container simply execute the `exit` command,
notice how the prompt changes back to your normal prompt. You have exited the 
container and are now back on your default shell of you laptop.

```
root@5b91b65eb29f:/# exit

---

exit
macsteven:~ trescst$
```

> NOTE: by exiting the interactive container we stop the `bash` command and our
> container is automatically terminated.

When we first started our interactive Ubuntu container it took some time before 
we were inside of our container.  Most of this time was because we had to 
download our Ubuntu image first.  But now that we have already downloaded this 
image (it is in our Podman cache), we can start our container in a matter of 
seconds (often even less than a second).

To see how little time it takes to start a new container, we are running the 
same command as above to start a new interactive Ubuntu container:

```
podman run -ti ubuntu bash
```

Now you can exit the container again by typing `exit`.

## Task 3: running services in containers

Altough it might be useful to run one-off containers and interactive containers, 
in most cases however containers are used to run services (for example a web 
service), to do this we will need to introduce a couple additional parameters:
* `-d` to run the container in detached mode (in the background)
* `--name nginx_container` to specify the name for the container
* `-p 8080:80` to expose port 80 of the container to port 8080 on the host 

The following command will run an nginx container that will be accessible 
through port 8080 of you laptop:
```
podman run -d --name nginx_container -p 8080:80 nginx
```

Once the container has started surf to http://localhost:8080 using a browser on
your laptop, you should be greeted with the following page:
![Nginx](images/lab-01-nginx.png)

If nginx is working stop and remove the container again using the commands 
below:

```
podman stop nginx_container
podman rm nginx_container
```

## Task 4: working with volumes

Often it is needed that files on your laptop are available inside the
container that you are running (for development purposes or for persistent
state reasons for example).  For this we need to use volumes, and again it is
simply a matter of adding an additional parameter (`-v`)

Create a new directory on your laptop in your **home directory**, `cd` into the directory and create a new file in the directory `index.html` with some contents:

```
mkdir ~/lab-01
cd ~/lab-01
vi index.html (Type "Podman Rocks!" and save the file)
```

Run the command below to mount the directory with the html files from your
laptop into the container:

```
cd ~/lab-01
podman run -d -p 8080:80 --name nginx_container_volume -v $(pwd):/usr/share/nginx/html nginx
```

Once the container is running surf to http://localhost:8080, you should see the text 

On your laptop make some changes to the `index.html` file (using any editor you have available, for
example add the current date.  Once you made some changes and saved those on
your laptop (hard) refresh the page in your browser to see the changes.

If nginx is working with the volume stop and remove the container again using 
the commands below:

```
podman stop nginx_container_volume
podman rm nginx_container_volume
```

## Task 5: clean up

To clean up run the following command:

```
podman system prune

---

WARNING! This command removes:
	- all stopped containers
	- all networks not used by at least one container
	- all dangling images
	- all dangling build cache

Are you sure you want to continue? [y/N] y
Deleted Containers
a730c516d5371fa0621e8b94e77659931b0da1d5037b72884be0de0526c0a214
ea6cc7def08c0451bfb4ee36b10d696553d4f2311d4b0bbe69fafbbf5e0f1a04
f38e4e2b6702b7138f038840259086adfcd74c477e710736ea660d3483bbff0b
Deleted Images
Total reclaimed space: 33.32MB
```