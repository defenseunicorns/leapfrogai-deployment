# LeapfrogAI Deployment Guide: CPU-only

## Table of Contents

- [Preparations](#preparations)
  - [Host Dependencies](#host-dependencies)
  - [Required Tools](#required-tools)
- [Installation](#installation)
- [Zarf](#zarf)
- [Assumptions](#assumptions)
- [Installation](#installation)
- [Troubleshooting](#troubleshooting)

## Environments

## Preparations

### Host Dependencies

These should already be on your environment from the start:

- jq
- docker
- build-essential
- iptables
- git
- procps

### Required Tools

These can be brought in and installed using Zarf, as binaries, or through a remote repository:

- k3d
- kubectl
- zarf

## Installation

## Zarf

The following instructions rely on Zarf to easily and quickly install all parts of the stack into a Kubernetes cluster

Deploying or updating things inside a Kubernetes cluster with Zarf is as easy as:

```bash
zarf package create --confirm
zarf package deploy <PACKAGE_TAR_FILE> --confirm
```

Visit https://zarf.dev/ for more details.

## Assumptions

The following assumptions are being made for the writing of these installation steps:

- User has a standard Unix-based operating system installed, with `sudo` access
  - Commands may need to be modified based on specific distribution
  - Commands were executed in an Ubuntu 22.04 LTS bash terminal
- User has a machine with at least the following minimum specifications:
  - CPU with at least 4 cores @ 3.00 GHz
  - RAM with at least 32 GB free memory
  - Storage with at least 128 GB free space
- If using GPU(s), user has a NVIDIA GPU with updated drivers and CUDA toolkit 11.8.x installed
  - VRAM of the GPU(s) determines layer offloading

## Instructions

### Important Notes

1. The following steps and commands must be executed in the order that they are presented within this README, top to bottom.

2. All `cd` commands are done relative to your development environment's "root folder". Assume every new step starts at the root folder.

3. "root folder" means the base directory where you are storing all the project dependencies for this installation.

4. "Internet Access" and "Isolated Network" will be noted when the instructions differ between the two. "Internet Access" refers to a system that can pull and execute remote dependencies from the internet; whereas "Isolated Network" refers to a system that is isolated and cannot reach outside networks or remote repositories

5. For all "Isolated Network" installs, `wget`, `git clone` and `zarf package create` commands are assumed to have been completed prior to entering the isolated network. The outputs from these commands should be stored on a removable media device and uploaded to the isolated machine.

6. For instances where you do not want to download a tagged version of a LeapfrogAI or Defense Unicorns release (e.g., leapfrogai-ctransformers-backend:0.2.0), you can tag and modify the pointers to the tag prior to any of the `zarf package create` commands:

```bash
docker build -t "ghcr.io/defenseunicorns/leapfrogai/<NAME_OF_PACKAGE>:<DESIRED_TAG>" .
# do a find and replace on anything matching the official tag (e.g., zarf.yaml, zarf-config.yaml, pyproject.toml, etc.)
# replace all instances of the old tag with the DESIRED_TAG
zarf package create zarf-package-<NAME_OF_PACKAGE>-*.tar.zst
```

7. If using GPU(s), see the `# [GPU VARIATION]` comments for details on setting up the cluster for GPU discovery and offloading

8. This guide shows you how to deploy these sub-stacks in an opinionated manner using K8s and Zarf. Variations from this guide may lead to unintended consequences or deployment failures. To deploy the sub-stacks using just Docker containers or on Host and Development, please go to the sub-stack's repository and follow the README.

### Switch to Sudo

This step isn't necessarily required, as long as your environment already has access to the `sudo` command prefix in general.

```bash
sudo su # login as required
```

### Install Tools

#### Zarf

_Internet Access:_

```bash
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

### Create Zarf Packages

#### K3d

```bash
# download
git clone https://github.com/defenseunicorns/zarf-package-k3d-airgap.git
cd zarf-package-k3d-airgap

# create
zarf package create --confirm

zarf tools download-init

cd metallb
zarf package create --confirm
```

#### UDS DUBBD

```bash
# download
git clone https://github.com/defenseunicorns/uds-package-dubbd.git

# create
cd uds-package-dubbd/k3d/
docker login registry1.dso.mil # account creation is required
zarf package create --confirm
```

#### LeapfrogAI API

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-api.git
cd leapfrogai-api/

# create
zarf package create --confirm
```

#### (OPTIONAL) Whisper Model

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-whisper.git
cd leapfrogai-backend-whisper # into leapfrogai-backend-whisper folder

# create
zarf package create --confirm
```

#### (OPTIONAL) CTransformers

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-ctransformers.git
cd leapfrogai-backend-ctransformers

# create
zarf package create --confirm
```

#### (OPTIONAL) Leapfrog Transcribe

```bash
# download
git clone https://github.com/defenseunicorns/doug-translate.git
cd doug-translate

# docker build image
docker build . -t ghcr.io/defenseunicorns/doug-translate:0.1.0

# create
zarf package create --confirm
```

#### (OPTIONAL) Leapfrog UI

```bash
# download
git clone https://github.com/defenseunicorns/leapfrog-ui.git
cd doug-translate

# docker build image
docker build . -t ghcr.io/defenseunicorns/leapfrogai/leapfrog-ui:0.0.1

# create
zarf package create --confirm
```

#### Setup the K3d Cluster

```bash
cd zarf-package-k3d-airgap/temp
# [GPU VARIATION] set `enable_gpus=true` for GPU discovery and offloading
zarf package deploy --set enable_traefik=false --set enable_service_lb=true --set enable_metrics_server=false --set enable_gpus=false ../zarf-package-k3d-airgap-*.tar.zst

cd ../
zarf init --components git-server --confirm

cd metallb
zarf package deploy --confirm zarf-package-metallb-*.tar.zst
```

#### Deploy DUBBD

```bash
# deploy
cd uds-package-dubbd/k3d/
zarf package deploy --confirm zarf-package-dubbd-k3d-*.tar.zst
```

#### Deploy LeapfrogAI

```bash
cd leapfrogai-api/

# install
zarf package deploy zarf-package-leapfrogai-api-*.zst
# press "y" for prompt on deployment confirmation
# press "y" for prompt to create and expose new gateway for load balancer access
```

#### (OPTIONAL) Deploy Whisper Model

```bash
# install
cd leapfrogai-backend-whisper # into leapfrogai-backend-whisper folder
zarf package deploy zarf-package-whisper-*.tar.zst --confirm
```

#### (OPTIONAL) Deploy CTransformers

```bash
# install
cd leapfrogai-backend-ctransformers
zarf package deploy zarf-package-ctransformers-*.tar.zst --confirm
```

#### (OPTIONAL) Deploy Leapfrog Transcribe

```bash
# install
cd doug-translate
zarf package deploy zarf-package-doug-translate-*.tar.zst --set DOMAIN=localhost:8080 --confirm
# press "y" for prompt on deployment confirmation
# for "LEAPFROGAI_BASE_URL" prompt, press enter
# for "DOMAIN" prompt type your user facing url in this format "localhost:8080"
# for "SUMMARIZATION_MODEL" prompt, press enter
```

#### (OPTIONAL) Deploy Leapfrog UI

```bash
# install
cd leapfrog-ui
zarf package deploy zarf-package-leapfrog-ui-*.tar.zst --set DOMAIN=https://localhost:8080 --confirm
# press "y" for prompt on deployment confirmation
# press enter for all prompts except the following
# for "DOMAIN" prompt type your user facing url in this format "https://localhost:8080"
```

#### Setup Access

```bash
k3d cluster edit zarf-k3d --port-add "443:30535@loadbalancer"
k3d cluster edit zarf-k3d --port-add "8080:30535@loadbalancer"

# if the load balancer does not restart
k3d cluster start zarf-k3d
```

Go to https://localhost:8083 to hit the Leapfrog Transcribe frontend web application, and run the Transcription and Summarization workflow with your own audio or video files.

## Troubleshooting

Below are some occasional deployment issues that we have encountered that, and you may too.

### Cluster Connection Issues

After performing a restart or restarting the docker service, the cluster cannot be connected with.

```bash
k3d cluster list
# verify that the cluster has `LOADBALANCER` set to true
k3d cluster stop <cluster-name>
k3d cluster start <cluster-name>
```

## Issues and Feature Requests

For issues or questions related to this deployment guide, please submit an issue in this repository.

For issues related to any of the components this guide teaches you to deploy, please go to that sub-stack's repository to submit an issue.
