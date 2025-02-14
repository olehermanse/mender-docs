---
title: Deploy a Container update
taxonomy:
    category: docs
    label: tutorial
---

!!! This tutorial is not supported by the virtual device because it does not come
!!! with a package manager to install the needed dependencies.

This tutorial will walk you through how to deploy Docker container updates with
Mender. We will be using the
[Docker Update Module](https://hub.mender.io/t/docker/324?target=_blank) which
allows you to specify a list of container images and their versions in a
[Mender Artifact](../../02.Overview/03.Artifact/docs.md) which
can later be deployed to your device using the hosted Mender Server.

## Prerequisites

To follow this tutorial, you will need to install:

* [Docker Engine](https://docs.docker.com/engine/install?target=_blank) on your
workstation.

It is also assumed that you have completed the following tutorials:

* [Prepare a Raspberry Pi device](../01.Preparation/01.Prepare-a-Raspberry-Pi-device/docs.md)
* [Deploy an application update](../02.Deploy-an-application-update/docs.md)

## Step 1 - Install the `docker`  update module

!!! This step is required with Mender Client 5.0 or newer. Previous Mender Client versions shipped
!!! this update module with the default installation

Log in to your Raspberry Pi and install the update module with the following command:

<!--AUTOVERSION: "mender-update-modules/%/docker"/ignore-->
```bash
mkdir -p /usr/share/mender/modules/v3 && wget -P /usr/share/mender/modules/v3 https://raw.githubusercontent.com/mendersoftware/mender-update-modules/master/docker/module/docker && chmod +x /usr/share/mender/modules/v3/docker
```

## Step 2 - Install Docker Engine on your Raspberry Pi

Log in to your Raspberry Pi and run the commands outlined below.

The [Docker Update Module](https://hub.mender.io/t/docker/324?target=_blank) has
a dependency on the `jq` utility, run the following command to install it:

```bash
sudo apt-get install jq
```

Download the Docker install script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

Execute the Docker install script:

```bash
sudo sh get-docker.sh
```

Once the installation script has finished, verify that you can run the
`sudo docker images` command.

You will get similar output to below:

```bash
sudo docker images
```
> ```console
> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> ```

As you can see, there are no Docker images on the device. In the next step we
will generate a
[Mender Artifact](../../02.Overview/03.Artifact/docs.md) which will
download an image.

### Step 3 - Download the mender-artifact utility on your workstation

!!! The simplest installation instructions for `mender-artifact` are covered below, see
!!! [Downloads](../../10.Downloads/docs.md#mender-artifact) for installation alternatives such as
!!! setting up package repositories.

On Linux, download the `mender-artifact` deb package and install it:

<!--AUTOVERSION: "mender-artifact_%-1"/mender-artifact -->
```bash
wget https://downloads.mender.io/repos/debian/pool/main/m/mender-artifact/mender-artifact_4.0.0-1%2B$(. /etc/os-release; echo $ID)%2B$(. /etc/os-release; echo $VERSION_CODENAME)_amd64.deb
sudo dpkg --install mender-artifact_4.0.0-1+$(. /etc/os-release; echo $ID)+$(. /etc/os-release; echo $VERSION_CODENAME)_amd64.deb
```

On MacOS, download the `mender-artifact` binary, give exec permissions, and add it to your path:

<!--AUTOVERSION: "mender-artifact/%/"/mender-artifact -->
```bash
mkdir -p ${HOME}/bin
wget https://downloads.mender.io/mender-artifact/4.0.0/darwin/mender-artifact -O ${HOME}/bin/mender-artifact
chmod +x ${HOME}/bin/mender-artifact
export PATH="${PATH}:${HOME}/bin"
```

!!! Add the last line from above to `~/.bashrc` or equivalent to make it persistent across multiple
!!! terminal sessions.

## Step 4 - Prepare a Mender Artifact on your workstation

Prepare a workspace and change directory to it:

```bash
mkdir "${HOME}/mender-docker" && cd "${HOME}/mender-docker"
```

Download the `docker-artifact-gen` utility script:

<!--AUTOVERSION: "mender-update-modules/%/docker"/ignore-->
```bash
wget https://raw.githubusercontent.com/mendersoftware/mender-update-modules/master/docker/module-artifact-gen/docker-artifact-gen
```

Make `docker-artifact-gen` executable:

```bash
chmod +x docker-artifact-gen
```

Set the target device type:

```bash
DEVICE_TYPE="raspberrypi4"
```

!!! Change `raspberrypi4` to `raspberrypi3` if you are using a Raspberry Pi 3


Generate a Mender Artifact that will deploy the `hello-world` Docker container image:

```bash
./docker-artifact-gen \
    -n "hello-world-container-update" \
    -t "${DEVICE_TYPE}" \
    -o "hello-world-container-update.mender" \
    "hello-world"
```

!!! [hello-world](https://hub.docker.com/_/hello-world?target=_blank) is the name of the Docker
!!! image that we want to download on the device when we deploy the generated
!!! [Mender Artifact](../../02.Overview/03.Artifact/docs.md)

## Step 5 - Deploy the Docker update

Upload the file `hello-world-container-update.mender` from the previous step
to the hosted Mender. Go to the **RELEASES** tab in the UI and upload it.

Once uploaded, click **CREATE DEPLOYMENT WITH THIS RELEASE** in order to deploy
it to your device.

If the deployment of `hello-world-container-update.mender` is successful, you
will see that there is a image downloaded on the device:

```bash
sudo docker images
```
> ```console
> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> hello-world         <none>              851163c78e4a        4 months ago        4.85kB
> ```

You can also see that the container was started, but since the hello-world
container does not contain a daemon it exited immediately:

```bash
sudo docker ps -a | head
```
> ```console
> CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
> 72e6cf80fa6a   hello-world   "/hello"   45 seconds ago   Exited (0) 42 seconds ago             optimistic_shaw
> ```

The [Docker Update Module](https://hub.mender.io/t/docker/324?target=_blank)
will download and run the specified images from e.g
[https://hub.docker.com](https://hub.docker.com).

## Read more

There is an alternative option for deploying containerized updates:

* *[Docker Compose Update Module](../../06.Artifact-creation/01.Create-an-Artifact/10.Docker-Compose)* provides an interface for deploying container workloads managed by Docker Compose.

You can explore other types of updates available by extending the Mender Client
in the
[Update Modules category in the Mender Hub community platform](https://hub.mender.io/c/update-modules/13?target=_blank).
