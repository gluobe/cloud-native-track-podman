## Lab 00 - Prerequisites

A couple of requirements before we can start with the actual labs.

## Task 0: Docker Hub account

If you do not yet have a Docker Hub account, please sign up for a free one at:

https://hub.docker.com/signup

## Task 1: Install Podman

### Step 0: Install `brew`

If you have not yet installed `brew`, install it :

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Step 1: Install Podman

Install `podman` using the command below:

```
brew install podman
```

Start the Podman-managed VM:

```
podman machine init
podman machine start
```

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

## Task 2: Login to Docker Hub

Next we will login to the Docker Hub with `podman`, this allows us to push/pull private container images:

```
podman login docker.io

Username: <YOUR_USERNAME>
Password:

Login Succeeded!
```