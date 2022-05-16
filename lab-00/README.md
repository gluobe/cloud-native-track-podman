## Lab 00 - Prerequisites

A couple of requirements before we can start with the actual labs.

## Task 1: Docker Hub account

If you do not yet have a Docker Hub account, please sign up for a free one at:

https://hub.docker.com/signup

## Task 2: Enable Podman

### Step 0: Install Podman

If you already followed all steps in the prerequisites repo you can skip this step and move to `Step 1`.

If you did not, follow all steps in the [prerequisites repo](https://github.com/gluobe/cloud-native-track-prerequisites/tree/main/prereq-01-podman)

### Step 1: Start Podman VM

```
podman machine start
```

### Step 2: Test Podman

Ensure that everything is working:

```
podman version

---

Client:       Podman Engine
Version:      4.0.3
API Version:  4.0.3
Go Version:   go1.18
Built:        Fri Apr  1 17:28:59 2022
OS/Arch:      darwin/amd64

Server:       Podman Engine
Version:      4.0.2
API Version:  4.0.2
Go Version:   go1.16.14
Built:        Thu Mar  3 15:56:56 2022
OS/Arch:      linux/amd64
```

## Task 3: Login to Docker Hub

Next we will login to the Docker Hub with `podman`, this allows us to push/pull private container images:

```
podman login docker.io

Username: <YOUR_USERNAME>
Password:

Login Succeeded!
```