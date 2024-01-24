# LeapfrogAI Deployment Guide: CPU-only

## Table of Contents

- [Preparations](#preparations)
- [Assumptions](#assumptions)
- [Important Notes](#important-notes)
- [Instructions](#instructions)
- [Troubleshooting](#troubleshooting)

## Preparations

### Host Dependencies

These tools and packages should already be in your environment from the start:

- jq
- docker
- build-essential
- iptables
- git
- procps

### Required Tools

These can be brought in and installed using Zarf, as binaries, or through a remote repository:

- k3d (>= 1.27.x)
- zarf (>= 0.30.x)

## Assumptions

The following assumptions are being made for the writing of these installation steps:

- User has a standard Unix-based operating system installed
  - Some commands may need to be modified depending on your CLI and package manager
- User has root (`sudo su`) access
  - Rootless mode details can be found here in [Docker's documentation](https://docs.docker.com/engine/security/rootless/)

## Important Notes

1. The following steps and commands must be executed in the order that they are presented in the [Instructions](#instructions) section

2. All `cd` commands are done relative to your development environment's project working directory (PWD)

   - You should assume every new step starts at the root of that directory
   - We recommend creating a new PWD named `/leapfrogai` in your home directory and putting everything in there

3. "_Internet Access_" and "_Isolated Network_" will be noted when the instructions differ between the two

   - "_Internet Access_" refers to a system that can pull and execute on remote dependencies from the internet
   - "_Isolated Network_" refers to a system that is isolated and cannot reach outside networks or remote repositories
   - "_Isolated Network_" instructions should also work with devices that have internet access

4. For all "_Isolated Network_" installs, `wget`, `git clone` and `zarf package create` commands are assumed to have been completed prior to entering the isolated network

   - The files and binaries from these commands should be stored on a removable media device and uploaded to the isolated machine

5. For instances where you want a specific version of a tool being installed, we recommend following the "_Isolated Network_" instructions

6. For instances where you do not want to download a tagged version of a LeapfrogAI or Defense Unicorns release (e.g., leapfrogai-api:0.3.0), you can build an image from source before you execute `zarf package create`:

```bash
docker build -t "ghcr.io/defenseunicorns/leapfrogai/<NAME_OF_PACKAGE>:<DESIRED_TAG>" .
# find and replace any manifests noting the image tag (e.g., zarf.yaml, zarf-config.yaml, etc.)
zarf package create zarf-package-<NAME_OF_PACKAGE>-*.tar.zst
```

7. When building your own Docker image from source, you should re-tag and push these images to a local registry container to improve the speed of zarf package creation. Below is an example of how to do so with our whisper backend:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker build -t ghcr.io/defenseunicorns/leapfrogai/whisper:0.4.0 .
docker tag ghcr.io/defenseunicorns/leapfrogai/whisper:0.4.0 localhost:5000/defenseunicorns/leapfrogai/whisper:0.4.0
docker push localhost:5000/defenseunicorns/leapfrogai/whisper:0.4.0
zarf package create --registry-override ghcr.io=localhost:5000 --set IMG=defenseunicorns/leapfrogai/whisper:0.4.0
```

## Instructions

### Switch to Sudo

```bash
sudo su # login as required
```

### Install Tools

#### Zarf

_Internet Access:_

```bash
# installs latest version of Zarf
brew install zarf
```

_Isolated Network:_

```bash
# download and store on removable media
wget https://github.com/defenseunicorns/zarf/releases/download/v0.31.0/zarf_v0.31.0_Linux_amd64

# upload from removable media and install
mv zarf_v0.31.0_Linux_amd64 /usr/local/bin/zarf
chmod +x /usr/local/bin/zarf

# check
zarf version
```

#### Kubectl

_Internet Access:_

```bash
apt install kubectl
```

_Isolated Network:_

```bash
# download and store on removable media
wget https://dl.k8s.io/release/v1.28.3/bin/linux/amd64/kubectl

# upload from removable media and install
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# check
kubectl version
```

### Deploy Kubernetes Cluster

> [!IMPORTANT]
> The following steps are split into _download_, _create_, and _deploy_. The only different between "_Internet Access_" and "_Isolated Network_" is that you perform _download_ and _create_ outside of the isolated network's environment, and then perform _deploy_ in the isolated network environment.

#### Bootstrap K3d

```bash
# download
git clone https://github.com/defenseunicorns/zarf-package-k3d-airgap.git
cd zarf-package-k3d-airgap

# create
zarf package create --confirm

zarf tools download-init

cd metallb
zarf package create --confirm

# install
cd ../ # if still in metallb folder
mkdir temp && cd temp
zarf package deploy --set enable_traefik=false --set enable_service_lb=true --set enable_metrics_server=false --set enable_gpus=false ../zarf-package-*.tar.zst

cd ../
zarf init --components git-server --confirm

cd metallb
zarf package deploy --confirm zarf-package-*.tar.zst
```

#### UDS DUBBD

```bash
# download
git clone https://github.com/defenseunicorns/uds-package-dubbd.git
cd uds-package-dubbd/k3d/

# create
docker login registry1.dso.mil # account creation is required
zarf package create --confirm

# install
zarf package deploy --confirm zarf-package-*.tar.zst
```

#### Kyverno Configuration

As of UDS DUBBD, v0.12+, a new Kyverno policy prevents some LeapfrogAI pods from running. As we work through some refactoring to overcome these policies, the following are instructions for temporarily changing the policy from `Enforce` to `Audit`.

```bash
# open the built-in k9s CLI tool
zarf tools monitor
# perform the following key presses and actions:
#     press ":", a CLI should pop up
#     type "kyverno", press "tab" to autocomplete, press "ENTER" to go to resource
#     press "/", a search box should pop up,
#     type "require-non-root-user", press "ENTER"
#     press "E" to edit YAML
#     press "/", a search box should pop up
#     type "Enforce", press "ENTER"
#     press "I" to enter insert mode
#     change "Enforce" to "Audit"
#     press "ESC", type ":wq", press "ENTER"
#     press ":",  a CLI should pop up
#     type "quit" to exist the tool
```

### Deploy LeapfrogAI

#### LeapfrogAI API

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-api.git
cd leapfrogai-api/

# create
zarf package create --confirm

# install
zarf package deploy zarf-package-*.zst --set ISTIO_ENABLED=true --set PREFIX=leapfrogai-api
# if used without the `--confirm` flag, there are many prompted variables
# please read the variable descriptions in the zarf.yaml for more details
```

#### (OPTIONAL) Whisper Model

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-whisper.git
cd leapfrogai-backend-whisper

# create
zarf package create --confirm

# install
zarf package deploy zarf-package-*.tar.zst --confirm
```

#### (OPTIONAL) LLaMA CPP Python

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-llama-cpp-python.git
cd leapfrogai-backend-llama-cpp-python

# create
zarf package create --confirm
```

#### (OPTIONAL) Leapfrog UI

```bash
# download
git clone https://github.com/defenseunicorns/leapfrog-ui.git
cd leapfrog-ui

# create
zarf package create --confirm

# install
cd leapfrog-ui
zarf package deploy zarf-package-*.tar.zst --confirm
# if used without the `--confirm` flag, there are many prompted variables
# please read the variable descriptions in the zarf.yaml for more details
```

### Setup Ingress/Egress

```bash
k3d cluster edit zarf-k3d --port-add "443:30535@loadbalancer"
k3d cluster edit zarf-k3d --port-add "8080:30535@loadbalancer"

# if the load balancer does not restart
k3d cluster start zarf-k3d
```

### Stopping and Clean-up

#### Stop K3d Cluster

Do one of the following, with the K3d-based command being preferred:

```bash
k3d cluster stop zarf-k3d
```

```bash
docker ps
# grab the k3d cluster's container ID
docker stop <K3D_CLUSTER_CONTAINER_ID>
```

#### Stop Zarf Registry

```bash
docker ps
# grab the registry container ID
docker stop <REGISTRY_CONTAINER_ID>
```

#### Clean-up

> [!CAUTION]
> This will remove EVERYTHING that is not attached to an active process

```bash
docker system prune -a -f && docker volume prune -f
zarf tools clear-cache
rm -rf /tmp/zarf-*
```

## Troubleshooting

Below are some occasional deployment issues that we have encountered that, and you may too.

### Cluster Connection

After performing a restart or restarting the docker service, the cluster cannot be connected with.

```bash
k3d cluster list
# verify that the cluster has `LOADBALANCER` set to true
# if not, try the following
k3d cluster stop zarf-k3d
k3d cluster start zarf-k3d
```

### Disk Pressure

In some cases, there may be storage issues if you are uploading multiple large AI models. There are several things you can try to reduce disk space or to increase space in a specific partition.

#### Remove Unused Files and Images

You can run the following sets of commands below to remove dangling or extraneous items. You can also go ahead and delete any already-deployed Zarf packages to save some space as well.

> [!CAUTION]
> Some of these will remove EVERYTHING that is not attached to an active process. Please know what command you are executing before doing so.

```bash
# prune images stored in the local registry
zarf tools registry prune --confirm
# prune docker images, press "y" to confirm
docker image prune
# prune volumes, press "y" to confirm
docker volume prune
# clear zarf cache and temp files
zarf tools clear-cache
rm -rf /tmp/zarf-*
```

#### Check Storage

First, check your disk's or mount's remaining space and utilization.

```bash
df -h
```

Go to the disk or mount in question, and check on the following paths:

```bash
ls -la /tmp
ls -la /var/lib/docker
```

In addition to your PWD, those paths are the usual suspects for taking up too much space. You may need to perform manual clean-up, or provision more space for the disks or mounts that these live in.
