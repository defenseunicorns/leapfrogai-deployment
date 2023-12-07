# LeapfrogAI Deployment Guide

This repository contains documentation for deploying LeapfrogAI, an AI-as-a-service (AaaS) platform that brings the capabilities of AI models to egress-limited environments. LeapfrogAI allows teams to deploy APIs that mirror OpenAI's spec, enabling them to use tools built around OpenAI's models in their own environment without the need to release sensitive data to SaaS tools.

## Table of Contents
- [Environments](#environments)
    - [Tested Operating System](#tested-operating-system)
    - [Tested Hardware](#tested-hardware)
- [Preparations](#preparations)
    - [Stack](#stack)
    - [Tools](#tools)
- [Installation](#installation)
- [Zarf](#zarf)
- [Assumptions](#assumptions)
- [Instructions](#instructions)
    - [0. Switch to Sudo](#0-switch-to-sudo)
    - [1. Install Tools](#1-install-tools)
    - [2. Create Zarf Packages](#2-create-zarf-packages)
        - [K3d](#k3d)
        - [DUBBD](#dubbd)
        - [LeapfrogAI](#leapfrogai)
        - [Whisper Model [Optional]](#whisper-model)
        - [CTransformers](#ctransformers)
        - [Leapfrog Transcribe [Optional]](#leapfrog-transcribe)
        - [Leapfrog UI [Optional]](#leapfrog-ui)
    - [3. Install Zarf Packages](#3-install-zarf-packages)
        - [Setup the K3d Cluster](#setup-the-k3d-cluster)
        - [DUBBD](#deploy-dubbd)
        - [LeapfrogAI](#deploy-leapfrogai)
        - [Whisper Model [Optional]](#deploy-whisper-model)
        - [CTransformers](#deploy-ctransformers)
        - [Leapfrog Transcribe [Optional]](#deploy-leapfrog-transcribe)
        - [Leapfrog UI [Optional]](#deploy-leapfrog-ui)
    - [4. Setup Access](#4-setup-access)
    - [5. Test Access](#5-test-access)
    - [6. Extended API Testing [Optional]](#6-extended-api-testing-optional)
- [Troubleshooting](#troubleshooting)
- [Disclaimers](#disclaimers)

## Environments

### Tested Operating System

* Ubuntu LTS (jammy)
    * 22.04.5
    * 22.04.4
    * 22.04.3
* MacOS 13.0.1 (Ventura)

### Tested Hardware
* 64 logical CPU cores (?) and ~250 GB of free RAM, distributed, no gpu
* 32 logical CPU cores (`AMD Ryzen Threadripper PRO 5955WX`) and ~250 GB of free RAM, 2x `NVIDIA RTX A4000`
* 64 logical CPU cores (`Intel(R) Xeon(R) Platinum 8358 CPU`) and ~200 GB of free RAM, 1x `NVIDIA RTX A10`
* 10 logical CPU cores (`Apple M1 Pro`) and ~32 GB of free RAM, 1x `Apple M1 Pro`
* 32 logical CPU cores (`13th Gen Intel® Core™ i9-13900KF`) and ~190GB of free RAM, 1x `NVIDIA RTX 4090`

## Preparations

### Stack

Base
1. [K3d Air-gap Zarf Package](https://github.com/defenseunicorns/zarf-package-k3d-airgap)
2. [UDS' DUBBD K3d Software Factory Package](https://github.com/defenseunicorns/uds-package-dubbd/tree/main/k3d)
3. [LeapfrogAI Python API](https://github.com/defenseunicorns/leapfrogai-api)

Transcription
1. [LeapfrogAI Whisper Backend, faster-whisper and whisper-base](https://github.com/defenseunicorns/leapfrogai-backend-whisper)
2. [Leapfrog Transcribe](https://github.com/defenseunicorns/doug-translate)

Summarization
1. [LeapfrogAI CTransformers Backend, wrapping The Bloke's Quantized Synthia-7b (Q4, K, M GGUF)](https://github.com/defenseunicorns/leapfrogai-backend-ctransformers)

### Tools

1. jq
2. docker
3. k3d
4. kubectl
5. zarf


## Installation

## Zarf

The following instructions rely on Zarf to easily and quickly install all parts of the stack into a Kubernetes cluster

Deploying or updating things inside a Kubernetes cluster with Zarf is as easy as:

```bash
zarf package create --confirm
zarf package deploy --confirm
```

See https://zarf.dev/ for more details.

## Assumptions

The following assumptions are being made for the writing of these installation steps:

- User has a standard Unix-based operating system installed, with `sudo` access
    - Commands may need to be modified based on specific distribution
    - Commands were executed in an Ubuntu 22.04 LTS bash terminal
- User has a machine with at least the following minimum specifications:
    - CPU with at least 4 cores @ 3.00 GHz
    - RAM with at least 32 GB free memory
    - Storage with at least 128 GB free space
    - Minimum base software: `apt update && apt install build-essential iptables git procps jq docker.io -y`

## Instructions

The following steps and commands must be executed in the order that they are presented, top to bottom. All `cd` commands are done relative to your development environment's root folder. Assume every new step starts at the root folder.

_Note 1_: "root folder" means the base directory where you are storing all the project dependencies for this installation.

_Note 2_: 1) "Internet Access" and 2) "Isolated Network" will be noted when the instructions differ between 1) a system that can pull and execute remote dependencies from the internet, and 2) a system that is isolated and cannot reach outside networks or remote repositories.

_Note 3:_  For all "Isolated Network" installs, `wget`, `git clone` and `zarf package create` commands are assumed to have been done and stored on a removable media device. These commands are under the bash comments `download` and `create`.

_Note 4:_ For instances where you do not want to download a tagged version of a LeapfrogAI release (e.g., leapfrogai-ctransformers-backend:0.2.0), you can perform the following generic instructions prior to any of the `zarf package create` commands:

```bash
docker build -t "ghcr.io/defenseunicorns/leapfrogai/<NAME_OF_PACKAGE>:<DESIRED_TAG>" .
# do a find and replace on anything matching the official tag
# example in VSCode for: in the git directory press alt+shift+f, and search 0.2.0 and "replace all" with your DESIRED_TAG
zarf package create zarf-package-<NAME_OF_PACKAGE>-*.tar.zst
```

### 1. Switch to Sudo

```bash
sudo su # login as required
```

### 2. Install Tools

For each of these commands, be in the `tools/` directory.

```bash
cd tools
```

#### Zarf

_Internet Access:_

```bash
brew install zarf
```

_Isolated Network:_

```bash
# download
wget https://github.com/defenseunicorns/zarf/releases/download/v0.31.0/zarf_v0.31.0_Linux_amd64

# install
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
# download
wget https://dl.k8s.io/release/v1.28.3/bin/linux/amd64/kubectl

# install
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# check
kubectl version
```

### 3. Create Zarf Packages

#### K3d

```bash
# download
git clone https://github.com/defenseunicorns/zarf-package-k3d-airgap.git
cd zarf-package-k3d-airgap

# install
zarf package create --confirm

zarf tools download-init

cd metallb
zarf package create --confirm
```

#### DUBBD
```bash
# download
git clone https://github.com/defenseunicorns/uds-package-dubbd.git

# create
cd uds-package-dubbd/k3d/
docker login registry1.dso.mil # account creation is required
zarf package create --confirm
```

#### LeapfrogAI

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-api.git
cd leapfrogai-api/

# create
zarf package create --confirm
```

#### Whisper Model

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-whisper.git
cd leapfrogai-backend-whisper # into leapfrogai-backend-whisper folder

# create
zarf package create --confirm
```

#### CTransformers

```bash
# download
git clone https://github.com/defenseunicorns/leapfrogai-backend-ctransformers.git
cd leapfrogai-backend-ctransformers

# create
zarf package create --confirm
```

#### Leapfrog Transcribe

```bash
# download
git clone https://github.com/defenseunicorns/doug-translate.git
cd doug-translate

# create
zarf package create --confirm
```

### 4. Install Zarf Packages

For each of these commands, be in the `zarf-packages/` directory.

```bash
cd zarf-packages
```

#### Setup the K3d Cluster

```bash
cd zarf-package-k3d-airgap/temp
zarf package deploy --set enable_traefik=false --set enable_service_lb=true --set enable_metrics_server=false --set enable_gpus=false ../zarf-package-k3d-airgap-amd64-5.5.2.tar.zst

cd ../
zarf init --components git-server --confirm

cd metallb
zarf package deploy --confirm zarf-package-metallb-amd64-v0.13.10.tar.zst
```

#### Deploy DUBBD

```bash
# deploy
cd uds-package-dubbd/k3d/
zarf package deploy --confirm zarf-package-k3d-local-*.tar.zst
```

#### Deploy LeapfrogAI

```bash
cd leapfrogai-api/

# install
zarf package deploy zarf-package-leapfrogai-api-*.zst
# press "y" for prompt on deployment confirmation
# press "y" for prompt to create and expose new gateway for load balancer access
```

#### Deploy Whisper Model

```bash
# install
cd leapfrogai-backend-whisper # into leapfrogai-backend-whisper folder
zarf package deploy zarf-package-whisper-*.tar.zst --confirm
```

#### Deploy CTransformers

```bash
# install
cd leapfrogai-backend-ctransformers
zarf package deploy zarf-package-ctransformers-*.tar.zst --confirm
```

#### Deploy Leapfrog Transcribe

```bash
# install
cd doug-translate
zarf package deploy zarf-package-doug-translate-*.tar.zst
# press "y" for prompt on deployment confirmation
# for "LEAPFROGAI_BASE_URL" prompt, press enter
# for "DOMAIN" prompt type "localhost:8083"
# for "SUMMARIZATION_MODEL" prompt, press enter
```

#### Deploy Leapfrog UI

```bash
# install
cd leapfrog-ui
zarf package deploy zarf-package-leapfrog-ui-*.tar.zst --set
# press "y" for prompt on deployment confirmation
# for "LEAPFROGAI_BASE_URL" prompt, press enter
# for "DOMAIN" prompt type "localhost:8083"
# for "SUMMARIZATION_MODEL" prompt, press enter
```

#### 5. Setup Access

```bash
k3d cluster edit zarf-k3d --port-add "443:30535@loadbalancer"

# If the load balancer does not restart
k3d cluster start zarf-k3d
```

#### 6. Test Access

Go to https://localhost:8083 to hit the Leapfrog Transcribe frontend web application, and run the Transcription and Summarization workflow with your own audio or video files.

#### 6. Extended API Testing [Optional]

First, you need to port-forward the API from within the cluster to an `<API PORT>` of choice using `zarf tools monitor`. After that, you can run the following example cURL request:

```
curl --insecure -L -X "POST" -H "Accept: application/json" -H "Authorization: Bearer stuff" -d
'{"model":"ctransformers","prompt":"Give me the name of the US president that served in
2018.","temperature":1.0,"max_tokens":1024}'
http://localhost:<API PORT>/openai/v1/completions
```

## Troubleshooting

#### After performing a restart or restarting the docker service, the cluster cannot be connected with.
```bash 
k3d cluster list
# verify that the cluster has `LOADBALANCER` set to true
k3d cluster stop <cluster-name>
k3d cluster start <cluster-name>
```

## Disclaimers

Transcription & Summarization
- Summarization cannot gracefully (slow or clashing request handling) handle concurrent users, so 1 user at a time is recommended if no replica sets are created
- Transcription can handle concurrent users, but it may slow down all users if the CPU RAM is being maxed out
- Transcription and summarization workflow for a dense, 3-hour audio takes anywhere between 15-30 minutes depending on CPU and RAM
- Workflow accuracy worsens for audio longer than 30 minutes on whisper-base
    - Transcription accuracy is around 97% for audio between 5-20 minutes, and goes down to 70-95% for audio shorter or longer than 5-20 minutes
    - See the following for some transcription benchmarking details: https://github.com/defenseunicorns/whisper-benchmarking
    - Summarization has not been benchmarked, and the batching/concatenation method is experimental
