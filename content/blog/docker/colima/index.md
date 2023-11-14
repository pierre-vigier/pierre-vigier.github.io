---
title: "[docker] Replacing docker desktop"
date: 2023-11-14T12:00:00+08:00
summary: Replacing docker desktop with colima
draft: false
tags: [ "docker", "lima", "colima" ]
---
Looking for an alternative for docker desktop, after their change of license led, me to [colima](https://github.com/abiosoft/colima). Colima is leveraging Lima to offer container management, and a viabale alternative to docker-desktop on MacOS.

Installing Lima and Colima is pretty straight forward and works directly with brew.

Once installed, and docker-desktop removed, using docker is as simple as starting colima with `colima start` and then using the normal docker-cli.

Further down the road, I did face some issues when playing with [kind](https://kind.sigs.k8s.io/) to test locally with Kubernetes. During those tests, i faced some issues, related to cgroupv2, that are present with Colima.

Colima is using Alpine as a lightweight VM to run containers on MacOS, Alpine is creating some issues in Colima for kind, unfortunately, Colima hides some of the configuration of Lima, and changing the image use for the OS is not present as a command line option.

Luckily, colima allows us to start it with option -e 

```
colima start -e 
```

This option allows for cusotmisation of the lima file before it's used to spawn a vm, and in this file we can configure more options, in this config, we can add the `images` section, which allows us to change the base os. At the same time, on Mac with Intel processor, it is possible to change the `vmType` option to `vz`, which uses the macOS virtualization framework, instead of qemu.


```
# ============================================================================================ #
# To abort, delete the contents of this file including the comments and save as an empty file
# ============================================================================================ #

# Number of CPUs to be allocated to the virtual machine.
# Default: 2
cpu: 2

# Size of the disk in GiB to be allocated to the virtual machine.
# NOTE: changing this has no effect after the virtual machine has been created.
# Default: 60
disk: 60

# Size of the memory in GiB to be allocated to the virtual machine.
# Default: 2
memory: 2

# Architecture of the virtual machine (x86_64, aarch64, host).
# Default: host
arch: x86_64

images:
  # Try to use a local image first.
  - location: "~/Downloads/hirsute-server-cloudimg-amd64.img"
    arch: "x86_64"
  # Download the file from the internet when the local file is missing.
  # Hint: run `limactl prune` to invalidate the "current" cache
  - location: "https://cloud-images.ubuntu.com/hirsute/current/hirsute-server-cloudimg-amd64.img"
    arch: "x86_64"



# Container runtime to be used (docker, containerd).
# Default: docker
runtime: docker

...

# Virtual Machine type (qemu, vz)
# NOTE: this is macOS 13 only. For Linux and macOS <13.0, qemu is always used.
#
# vz is macOS virtualization framework and requires macOS 13
#
# Default: qemu
vmType: vz
```

After those modification, we can check in the vm:

```
pierre.vigier» colima ssh
lima@colima:/Users/pierre.vigier$ uname -v
#10-Ubuntu SMP PREEMPT_DYNAMIC Fri Oct 13 13:49:38 UTC 2023
lima@colima:/Users/pierre.vigier$
```

Ubuntu is used as a base image instead of Alpine.
