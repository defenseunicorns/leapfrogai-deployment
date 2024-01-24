# LeapfrogAI Deployment Guide: NVIDIA GPUs

> [!WARNING]  
> GPU and CPU installation are very similar, so if you are installing with GPU support, bring the INSTALL-CPU.md as well, since all the instructions in this markdown only point out the steps that vary from the CPU instructions.

## Table of Contents

- [Preparations](#preparations)
- [Assumptions](#assumptions)
- [Important Notes](#important-notes)
- [Instructions](#instructions)
- [Troubleshooting](#troubleshooting)

## Preparations

### Host Dependencies

These tools and packages should already be in your environment from the start:

- nvidia-driver

### Required Tools

These can be brought in and installed using Zarf, as binaries, or through a remote repository:

- k3d (>= 1.27.x)
- zarf (>= 0.30.x)
- nvidia-container-toolkit (>=1.14.x)
- nvidia-cuda-toolkit (>=12.2.x)

## Assumptions

The following assumptions are being made for the writing of these installation steps:

- User has a standard Unix-based operating system installed
- User has root (`sudo su`) access
  - Rootless mode details can be found here in [Docker's documentation](https://docs.docker.com/engine/security/rootless/)
- Your NVIDIA GPU has the most up-to-date drivers installed
- Your NVIDIA GPU drivers can handle CUDA >= 12.2

## Important Notes

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

_Internet Access:_

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

### Deploy LeapfrogAI

#### LeapfrogAI API

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-api.git
cd leapfrogai-api/

# create
zarf package create --confirm

# install
zarf package deploy zarf-package-leapfrogai-api-*.zst
# press "y" for prompt on deployment confirmation
# press "y" for prompt to create and expose new ingress gateway
```

#### (OPTIONAL) Whisper Model

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-whisper.git
cd leapfrogai-backend-whisper

# create
zarf package create --confirm

# install
zarf package deploy zarf-package-whisper-*.tar.zst --confirm
```

#### (OPTIONAL) (DEPRECATED) CTransformers

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-ctransformers.git
cd leapfrogai-backend-ctransformers

# create
zarf package create --confirm
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
zarf package deploy zarf-package-*.tar.zst
# press "y" for prompt on deployment confirmation
# press enter if you want to skip a prompted variable and keep the default
# for "DOMAIN" prompt type your user facing url in this format "https://localhost:8080"
# for "PREFIX" prompt, add your desired route prefixes or leave empty
# for "MODEL" and "TRANSCRIPTION_MODEL", choose the name of the backend you deployed
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
