+++
title = "What is Singularity / Apptainer"
slug = "sing-01-intro"
weight = 1
+++

Singularity (recently changed its name to Apptainer):

- is an open-source project developed within the research community
- creates a custom virtual Linux environment (a **container**) that is different from the host Linux system
  - e.g., on a CentOS/Rocky Linux machine you can create a virtual Ubuntu system where you can install your
    own packaged software from Ubuntu repositories
  - in a sense, gives you control of your software environment without being `root` on the host system (with a
    catch: creating containers from scratch usually requires root)
- quickly became a way to package and deploy scientific software to different systems
- is different from Docker, as it does not require `root` access on the host system to run it
  - specifically designed for running containers on multi-user HPC clusters
- on a Linux host is very lightweight compared to a full virtual machine (**VM**)
- on Mac or Windows can be deployed inside a VM (still requires a Linux host layer &nbsp;➜&nbsp; a VM)
- from the technical standpoint, uses:
  - Linux control groups (<u>cgroups</u>) to limit / control / isolate resource usage such as CPU and memory
    access, disk I/O
  - <u>kernel namespaces</u> to virtualize and isolate OS resources, so that processes inside the container
    see only a specific, virtualized set of resources
  - <u>overlay filesystems</u> to enable the appearance of writing to otherwise read-only filesystems,
    e.g. system directories

## Why use a container

Idea: package and distribute the software environment along with the application.

Why:
1. avoid compiling complex software chains from scratch
1. install software in the environment where it might not be available (e.g. Chapel on Windows)
1. use a familiar software environment everywhere where you can run Singularity, e.g. across different HPC centres
1. popular, but somewhat dubious reason: code/results reproducibility (use the same dependencies as the authors)

An **image** is a bundle of files including an operating system, software and potentially data and other
application-related files. Singularity uses the Singularity Image Format (SIF), and images are provided as
single `.sif` files.

A **container is a virtual environment that is based on an image. You can start multiple container instances
from an image.

```sh
module load singularity
singularity --version   # singularity version 3.7.4 on the training cluster
```

```sh
module load apptainer
apptainer --version   # apptainer version 1.0.2 on production clusters
```
