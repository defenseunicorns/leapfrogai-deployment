# LeapfrogAI Deployment Guide

> [!WARNING]  
> The instructions in this repository require knowledge of Docker, Unix-based CLIs, GPU computing, and Kubernetes.

This repository contains documentation for deploying LeapfrogAI. LeapfrogAI an AI-as-a-service (AaaS) platform that brings the various capabilities and modalities of AI models to any environment, from cloud-based deployments to ingress/egress-limited servers. LeapfrogAI allows teams to deploy APIs that shadow OpenAI's API specification, enabling end-users and developers to build and use tools for almost any model and code library on the market All of this is done on your local machine, environment, etc., allowing the user to keep their information and sensitive data secure in their own environment.

Visit the main [LeapfrogAI repository](https://github.com/defenseunicorns/leapfrogai) for more details.

## Quickstart

Don't know Kubernetes that well, or just want to get something working locally as quickly as possible? Then just go to the [Tadpole repository](https://github.com/defenseunicorns/tadpole) and skip directly to a super-simplified Docker deployment using `docker compose`.

> [!NOTE]  
> Tadpole is only for local testing and development, and not meant for production. It also only presents a limited set of LeapfrogAI's capabilities.

Our goal for LeapfrogAI is to also simplify the Kubernetes-based deployment of LeapfrogAI into a single command; however, this repository's instructions will provide the user with as much customizability as possible during installation.

## Table of Contents

- [Instructions](#instructions)
- [Tested Environments](#tested-environments)
  - [Operating Systems](#operating-systems)
  - [Hardware](#hardware)
  - [Architectures](#architectures)
- [Issues and Features](#issues-and-features)

## Instructions

The instructions are located in the [INSTALL.md](./INSTALL.md).

> [!IMPORTANT]  
> The GPU and CPU-only installation are very similar, so if you are installing with GPU support then take note of drop-downs that show a difference in deployment configuration.

## Tested Environments

Below are some of the operating systems, architectures, and system specifications that we have tested our deployment instructions on.

In the future, our goal is to add instructions and test compatibility on other environments, e.g., ROCm, Metal, etc.

> [!NOTE]  
> The the speed and quality of LeapfrogAI and its hosted AI models depends heavily on the presence of a strong GPU to offload model layers to.

### Operating Systems

- Ubuntu LTS (jammy)
  - 22.04.2
  - 22.04.3
  - 22.04.4
  - 22.04.5
- MacOS 13.0.1 (Ventura)

### Hardware

- 64 CPU cores (`Unknown Compute via Virtual Machine`) and ~250 GB RAM, no GPU
- 32 CPU cores (`AMD Ryzen Threadripper PRO 5955WX`) and ~250 GB RAM, 2x `NVIDIA RTX A4000` (16Gb vRAM each)
- 64 CPU cores (`Intel Xeon Platinum 8358 CPU`) and ~200Gb RAM, 1x `NVIDIA RTX A10` (16Gb vRAM each)
- 10 CPU cores (`Apple M1 Pro`) and ~32 GB of free RAM, 1x `Apple M1 Pro`
- 32 CPU cores (`13th Gen Intel Core i9-13900KF`) and ~190GB RAM, 1x `NVIDIA RTX 4090` (24Gb vRAM each)
- 2x 128 CPU cores (`AMD EPYC 9004`) and ~1.4Tb RAM, 8x `NVIDIA H100` (80Gb vRAM each)
- 32 CPU cores (`13th Gen Intel Core i9-13900HX`) and ~64Gb RAM, 1x `NVIDIA RTX 4070` (8Gb vRAM each)

### Architectures

- `linux/amd64`
- `linux/arm64`

## Issues and Features

If you have issues with installation or if you believe something is missing from the installation guides then please submit an issue using the default template.
