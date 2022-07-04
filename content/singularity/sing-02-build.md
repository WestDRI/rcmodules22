+++
title = "Creating container images"
slug = "sing-02-build"
weight = 2
+++

<!-- https://docs.sylabs.io/guides/3.0/user-guide/build_a_container.html -->

The `singularity build` command is a versatile tool that lets you:

1. download and assemble existing containers from external hubs like the
   [Container Library](https://cloud.sylabs.io) (`library://`), [Docker Hub](https://hub.docker.com)
   (`docker://`), and [Singularity Hub](https://singularityhub.github.io) (`shub://`)
1. convert containers between different formats supported by Singularity
1. create a container from scratch using a Singularity definition file and customized it to your needs
1. create a container from a sandbox

<!-- You can create an image from: -->
<!-- - a recipe/definition file (similar to Dockerfile for Docker images) -->
<!-- - from Singularity Hub -->
<!-- - from Docker Hub -->
<!-- - few other options -->

```sh
singularity build --help
```

In many cases, you would download an existing Docker/Singularity container image and build/run a container
from it, without having to modify the image. Or maybe, the image is already provided by your lab, a research
collaborator, or the Alliance. For example, right now we are testing a set of Singularity images with GPU
development tools for different GPU architectures, such as NVIDIA's CUDA and AMD's ROCm, as it is often
difficult to compile these from source -- and we'll make these images available to all users.

In other cases you might want to modify the image or build your own container image from scratch. To create a
container image, you need a machine that:

1. runs Linux,
1. has Singularity installed,
1. has Internet access, and
1. ideally where you have `root` (or `sudo`) permissions, otherwise:
  - all permissions inside the image will be messed up, and you won't have `root` or `sudo` access
    &nbsp;➜&nbsp; you won't be able to install or upgrade packages (this requires `root`)
  - if installing packages is not important to you, i.e. you are planning to use the container as is for
    production purposes, you can create an image on an HPC cluster; for more details see
    [Creating images on Compute Canada clusters](https://bit.ly/3xg29gK).
  - for developing/modifying the container you will need `root`

{{<note>}} In Singularity the user always remains the same inside and outside of the container. In other
words, if you enter a container without root privileges, you won't be able to obtain root privileges within
the container. {{</note>}}





### Example: run a pre-packaged container

Run the "Lolcow" container by Singularity developers:

```sh
mkdir tmp && cd tmp
module load singularity
salloc --time=1:0:0 --mem-per-cpu=3600
singularity pull hello-world.sif shub://vsoch/hello-world   # store it into the name hello-world.sif
singularity run hello-world.sif   # runs its default script
```

Where is this script? What did running the container actually do to result in the displayed output?

```sh
singularity inspect -r hello-world.sif          # it runs the script /rawr.sh
singularity exec hello-world.sif cat /rawr.sh   # here is what's inside
```

We can also run an image on the fly without putting it into the current directory:

```sh
rm hello-world.sif                          # clear old image
singularity run shub://vsoch/hello-world    # use the cached image	
```

We also run a container directly off Docker Hub. However, it will first convert a Docker image into a
Singularity image, and then run this Singularity image:

```sh
singularity run docker://godlovedc/lolcow   # random cow message
```

Let's try something more basic:

```sh
singularity run docker://ubuntu   # press Ctrl-D to exit
```

What happened here in the last example? Well, there was no default script, so it just presented the container
shell to type commands.

### Singularity’s image cache

In the last two examples we did not store SIF images in the current directory. Where were they stored?

```sh
singularity cache list
singularity cache list -v
singularity cache clean          # clean all; will ask to confirm
singularity cache clean --help   # more granular control
```

By default, Singularity cache is stored in `$HOME/.singularity/cache`.






### Building a development (writable) container within a sandbox as root

{{<note>}}
You need to be root in this section.
{{</note>}}

1. Pull a existing Docker image from Docker Hub.
1. Create a modifiable sandbox directory into which you can install packages.
1. Add packages.
1. Convert the sandbox into a regular SIF image.

{{<note>}}
By default, Singularity containers are read-only, i.e. while you can write into bind-mounted directories,
normally you cannot modify files inside the container.
{{</note>}}

To build a writable container into which you can install packages, you need `root` access. The `--sandbox`
flag below builds a sandbox to which changes can be made, and the `--writable` flag launches a read-write
container. In these examples `ubuntu.dir` is a directory on the host filesystem.

Old, two-step method:

```sh
centos
cd tmp
wget https://raw.githubusercontent.com/moby/moby/master/contrib/download-frozen-image-v2.sh
sh download-frozen-image-v2.sh build ubuntu:latest   # download an image from Docker Hub into build/
cd build && tar cvf ../ubuntu.tar * && cd ..
sudo /opt/software/singularity-3.7/bin/singularity build --sandbox ubuntu.dir docker-archive://ubuntu.tar
/bin/rm -rf build ubuntu.tar download-frozen-image-v2.sh
sudo /opt/software/singularity-3.7/bin/singularity shell --writable ubuntu.dir
apt-get update            # hit return twice
apt-get -y install wget   # will fail here if run as regular user
exit
```

> The warning above when running `... singularity shell --writable ...`
> ```txt
> WARNING: Skipping mount /etc/localtime [binds]: /etc/localtime doesn't exist in container
> ```
> occurs when you try to mount a file/directory into the container without that file/directory already
> inside the container. You can simply ignore this message.

New, single-step method:

```sh
centos
cd tmp
sudo /opt/software/singularity-3.7/bin/singularity build --sandbox ubuntu.dir docker://ubuntu
sudo du -skh ubuntu.dir   # 81M
sudo /opt/software/singularity-3.7/bin/singularity shell --writable ubuntu.dir
apt-get update            # hit return twice
apt-get -y install wget   # will fail here if run as regular user
exit
sudo du -skh ubuntu.dir   # 118M
```

To convert the sandbox to a regular non-writable container, use

```sh
sudo /opt/software/singularity-3.7/bin/singularity build ubuntu.sif ubuntu.dir
sudo rm -rf ubuntu.dir
singularity shell ubuntu.sif   # can now start it as non-root
```










### Building a development container from a Singularity definition file

{{<note>}}
You need to be root in this section.
{{</note>}}

Create a new file `test.def`:

```txt
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom Singularity image!")'
```

We will bootstrap our image from a minimal Ubuntu 20.04 Linux Docker image as then run it as a regular user:

```sh
centos
cd tmp
sudo /opt/software/singularity-3.7/bin/singularity build test.sif test.def
ls -l test.sif   # 62M
module load singularity
singularity run test.sif   # Hello World! Hello from our custom Singularity image!
```

> On the training cluster, if you try to do the same as non-root, you will receive an error:
> ```sh
> $ singularity build test.sif test.def
> FATAL:   You must be the root user, however you can use --remote or --fakeroot to build from a Singularity recipe file
> $ singularity build --fakeroot test.sif test.def
> FATAL:   could not use fakeroot: no mapping entry found in /etc/subuid for razoumov
> ```

> <font style="color:blue" size=+3>Discussion</font>
>
> How do we install packages into this new container?

<!-- Answer: we would need to replace `test.sif` with `--sandbox test.dir` to make it a sandbox: -->
<!-- ```sh -->
<!-- sudo /opt/software/singularity-3.7/bin/singularity build --sandbox test.dir test.def -->
<!-- sudo /opt/software/singularity-3.7/bin/singularity shell --writable ubuntu.dir -->
<!-- apt-get update            # hit return twice -->
<!-- apt-get -y install wget   # will fail here if run as regular user -->
<!-- ``` -->




<!-- $ singularity build --remote test.sif test.def -->
<!-- FATAL:   Unable to submit build job: no authentication token, log in with `singularity remote login` -->








### Example: converting a Docker image to a Singularity container as a regular user

You can pull a Docker image from [Docker Hub](https://hub.docker.com) and convert it to a Singularity
image. For this you typically do not need `sudo` access. Please build containers only on compute nodes, as
this process is CPU-intensive.

The following commands require online access and will work only on Cedar cluster where compute nodes can
access Internet:

```sh
cd ~/scratch
salloc --cpus-per-task=4 --time=0:30:0 --mem-per-cpu=3600 --account=...
module load singularity
singularity pull topologytoolkit.sif docker://topologytoolkit/ttk:latest
```

On other clusters -- such as Béluga, Narval or Graham -- compute nodes do not have Internet access, so you
will have to use the two-step approach there:

```sh
cd ~/scratch
wget https://raw.githubusercontent.com/moby/moby/master/contrib/download-frozen-image-v2.sh
sh download-frozen-image-v2.sh build ttk:latest   # download an image
                                                  # from Docker Hub into build/
cd build && tar cvf ../ttk.tar * && cd ..
salloc --cpus-per-task=4 --time=0:30:0 --mem-per-cpu=3600 --account=...
module load singularity
singularity build topologytoolkit.sif docker-archive://ttk.tar   # build the Singularity image;
                                                         # wait for `Build complete`
/bin/rm -rf build topologytoolkit.tar
```
