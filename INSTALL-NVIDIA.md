# LeapfrogAI Deployment Guide: NVIDIA GPUs

> [!WARNING]  
> The GPU and CPU-only installation are very similar, so if you are installing with GPU support, bring the INSTALL-CPU.md as well, since all the instructions in this markdown only point out the steps and dependencies that vary from the CPU-only instructions.

## Table of Contents

- [Preparations](#preparations)
- [Assumptions](#assumptions)
- [Important Notes](#important-notes)
- [Instructions](#instructions)
- [Troubleshooting](#troubleshooting)

## Preparations

### Host Dependencies

- nvidia-driver

### Required Tools

- nvidia-container-toolkit (>=1.14.x)
- nvidia-cuda-toolkit (>=12.2.x)

## Assumptions

- Your NVIDIA GPU has the most up-to-date drivers installed
- Your NVIDIA GPU drivers can handle CUDA >= 12.2

## Important Notes

1. "N/A" will be used to denote that nothing changes from the CPU-only instructions, and those instructions should be referred to for that step.

2. If there is a variation from the CPU-only instructions, then the command(s) and steps that are different will be described.

## Instructions

### Switch to Sudo

N/A

### Install Tools

#### Zarf

N/A

#### Kubectl

_Internet Access:_

N/A

### Deploy Kubernetes Cluster

> [!IMPORTANT]
> The following steps are split into _download_, _create_, and _deploy_. The only different between "_Internet Access_" and "_Isolated Network_" is that you perform _download_ and _create_ outside of the isolated network's environment, and then perform _deploy_ in the isolated network environment.

#### Bootstrap K3d

```bash
# largest difference is setting `enable_gpus` to `true`
zarf package deploy --set enable_traefik=false --set enable_service_lb=true --set enable_metrics_server=false --set enable_gpus=true ../zarf-package-*.tar.zst
```

#### UDS DUBBD

N/A

### Deploy LeapfrogAI

#### LeapfrogAI API

N/A

#### (OPTIONAL) Whisper Model

BEFORE `zarf package create --confirm`, you will need to perform a docker build:

```bash
docker build -f Dockerfile.gpu -t ghcr.io/defenseunicorns/leapfrogai/whisper:0.0.1 .
```

The package deployment command also changes to this:

```bash
# install
zarf package deploy zarf-package-whisper-*.tar.zst --set GPU_ENABLED=true --confirm
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

N/A

### Setup Ingress/Egress

```bash
k3d cluster edit zarf-k3d --port-add "443:30535@loadbalancer"
k3d cluster edit zarf-k3d --port-add "8080:30535@loadbalancer"

# if the load balancer does not restart
k3d cluster start zarf-k3d
```

### Stopping and Clean-up

N/A

## Troubleshooting

Below are some occasional deployment issues that we have encountered that, and you may too.
