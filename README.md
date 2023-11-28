# LeapfrogAI Deployment Guide

This repository contains documentation for deploying LeapfrogAI, an AI-as-a-service (AaaS) platform that brings the capabilities of AI models to egress-limited environments. LeapfrogAI allows teams to deploy APIs that mirror OpenAI's spec, enabling them to use tools built around OpenAI's models in their own environment without the need to release sensitive data to SaaS tools.

## Table Contents

1. [Environments](#environments) 
    - [Tested Operating System](#tested-operating-system)
    - [Tested Hardware](#tested-hardware)
2. [Preparations](#preparations)
    - [Stack](#stack)
    - [Tools](#tools)
3. [Installation](#installation)

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

## Preparations

### Stack

Base
1. [K3d Air-gap Zarf Package](https://github.com/defenseunicorns/zarf-package-k3d-airgap)
2. [UDS' DUBBD K3d Software Factory Package](https://github.com/defenseunicorns/uds-package-dubbd/tree/main/k3d)
3. [LeapfrogAI Python API](https://github.com/defenseunicorns/leapfrogai-api)
4. [LeapfrogAI CTransformers Backend, wrapping The Bloke's Quantized Synthia-7b (Q4, K, M GGUF)](https://github.com/defenseunicorns/leapfrogai-backend-ctransformers)

Transcription
1. [LeapfrogAI Whisper Backend, faster-whisper and whisper-base](https://github.com/defenseunicorns/leapfrogai-backend-whisper)
2. [Leapfrog Transcribe](https://github.com/defenseunicorns/doug-translate)

### Tools

1. jq
2. docker
3. k3d
4. kubectl
5. zarf