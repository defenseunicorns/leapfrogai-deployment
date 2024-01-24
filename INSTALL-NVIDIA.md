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

- nvidia-driver[^1] (>=525.60)
- nvidia-container-toolkit[^2] (>=1.14)

### Required Tools

- nvidia-cuda-toolkit[^3] (>=12.2)

[^1]: must be pre-installed on the system before being air-gapped, see [details here](https://linuxconfig.org/how-to-install-the-nvidia-drivers-on-ubuntu-22-04)
[^2]: must be pre-installed on the system before being air-gapped, see [details here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation)
[^3]: only required on the system building the Zarf packages and not necessarily on the system to be deployed to, see [details here](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)

## Assumptions

- Your NVIDIA GPU has the most up-to-date drivers installed
- Your NVIDIA GPU drivers can handle CUDA >= 12.2

## Important Notes

1. "N/A" will be used to denote that nothing changes from the CPU-only instructions, and those instructions should be referred to for that step.

2. If there is a variation from the CPU-only instructions, then the command(s) and steps that are different will be described.

3. By default, each backend is configured to request 1 GPU device

   - At the moment, you cannot time-slice nor setup multi-instance GPU using these instructions
   - Over scheduling GPU resources will cause your backend pods to crash
   - To prevent crashing, install backends as CPU-only if you have filled up all your GPU devices already
   - See the [README.md](./README.md) for more details on future improvement to these instructions

## Instructions

### Switch to Sudo

N/A

### Install Tools

#### Zarf

N/A

#### Kubectl

N/A

#### NVIDIA

1. [Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation)
2. [CUDA Toolkit]()

### Deploy Kubernetes Cluster

> [!IMPORTANT]
> The following steps are split into _download_, _create_, and _deploy_. The only different between "_Internet Access_" and "_Isolated Network_" is that you perform _download_ and _create_ outside of the isolated network's environment, and then perform _deploy_ in the isolated network environment.

#### Bootstrap K3d

```bash
# largest difference is setting `enable_gpus` to `true`
zarf package deploy --set enable_traefik=false --set enable_service_lb=true --set enable_metrics_server=false --set enable_gpus=true ../zarf-package-*.tar.zst
```

#### (OPTIONAL) GPU Support Test

This helps confirm that the cluster's pods have access to GPU resources.

```bash
# download
git clone https://github.com/justinthelaw/gpu-support-test
cd gpu-support-test

# create
zarf package create --confirm

# install
zarf package deploy zarf-package-*.tar.zst
# press "y" for prompt on deployment confirmation
# enter the number of GPU(s) that are expected to be available when prompted RESOURCES_GPU and LIMITS_GPU

# check
zarf tools kubectl logs -n leapfrogai deployment/gpu-support-test
# the logs should show that GPU(s) are accessible
```

#### UDS DUBBD

N/A

### Deploy LeapfrogAI

#### LeapfrogAI API

N/A

#### (OPTIONAL) Whisper Model

Prior to `zarf package create --confirm`, you will need to perform a docker build:

```bash
docker build -f Dockerfile.gpu -t ghcr.io/defenseunicorns/leapfrogai/whisper:0.0.1 .
```

The package deployment command also changes to this:

```bash
# install
zarf package deploy zarf-package-*.tar.zst --set GPU_ENABLED=true --confirm
```

#### (OPTIONAL) LLaMA CPP Python

Prior to `zarf package create --confirm`, you will need to perform a docker build:

```bash
docker build -f Dockerfile.gpu -t ghcr.io/defenseunicorns/leapfrogai/llamacpp:0.0.1 .
```

The package deployment command also changes to this:

```bash
# install
zarf package deploy zarf-package-*.tar.zst --set GPU_ENABLED=true --confirm
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
