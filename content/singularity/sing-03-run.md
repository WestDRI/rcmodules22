+++
title = "Running containers"
slug = "sing-03-run"
weight = 3
+++

We already saw some of these commands:

- `singularity shell ubuntu.sif` launches the container and opens an interactive shell inside it
- `singularity exec ubuntu.sif command` launches the container and runs a command inside it
- `singularity run ubuntu.sif` launches the container and executes the default runscript

Singularity matches users between the container and the host. For example, if you run a container that needs
to be root, you also need to be root outside the container.

### Mounting external directories

By default, Singularity containers are read-only, so you cannot write into its directories. However, from
inside the container you can organize read-write access to your directories on the host filesystem. The
command

```sh
singularity shell -B /home,/project,/scratch ubuntu.sif
```

will bind-mount /home,... inside the container so that these directories can be accessed for both read and
write, subject to your account's permissions, and then will run a shell.

You can also add the `-e` flag to remove all Compute Canada environment variables from your container, to
start in a new clean environment:

```sh
singularity shell -e -B /home,/project,/scratch ubuntu.sif
```

You can mount host directories to specific paths inside the container, e.g.

```sh
singularity shell -B /project/def-sponsor00/${USER}:/project,$SCRATCH:/scratch ubuntu.sif
```

Note that usually Singularity mounts some of the host directories by default. The reason is that it needs some
space to store temporary files that get generated along the way, access some host's system files, and also
provide space in `/home` to store your data. You can disable specific mounts, e.g.

```sh
singularity shell --no-mount home ubuntu.sif
```

will start the container without a home directory. Alternatively, you can disable mounting `/home` with the
`--no-home` flag. And you can disable multiple mounts with something like `--no-mount tmp,sys,dev`.

You don't have to pass the same bind flags every time -- instead you can put them into a variable (that can be
stored in your `~/.bashrc` file):

```sh
export SINGULARITY_BIND="/home,/project,/scratch"
singularity shell ubuntu.sif
```

You can have more granular control (e.g. specifying read only) with the `--mount` flag -- for details see the
official
[Bind Paths and Mounts documentation](https://sylabs.io/guides/latest/user-guide/bind_paths_and_mounts.html).

### Running a single command

```sh
singularity exec ubuntu.sif ls /
singularity exec ubuntu.sif ls /; whoami
singularity exec ubuntu.sif cat /etc/os-release
```

### Running a default script

We've already done this!

### Starting a shell

We've already done this!

### Running container instances

You can also run backgrounded processes within your container. You can start/terminate these with
`instance start`/`instance stop`. All these processes will terminate once your job ends.

```sh
singularity instance start ubuntu.sif test01   # test01 is the name of the session
singularity shell ubuntu.sif instance://test01
nohup find / -type d >dump.txt
exit
singularity exec instance://test01 ls -lh dump.txt
singularity instance list
singularity instance stop test01
```
