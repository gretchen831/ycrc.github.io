# Singularity

[Singularity](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0177459) is a linux container technology that is well suited to use in shared-user environments such as the [clusters](/clusters-at-yale/clusters) we maintain at Yale. It is similar to [Docker](https://docs.docker.com/); You can bring with you a stack of software, libraries, and a Linux operating system that is independent of the host computer you run the container on. This can be very useful if you want to share your software environment with other researchers or yourself across several computers. Because Singularity containers run as the user that started them and mount home directories by default, you can usually see the data you're interested in working on that is stored on a host computer without any extra work.

Below we will outline some common use cases covering the creation and use of containers. There is also excellent documentation available on the full and official [user guide for Singularity](https://www.sylabs.io/guides/2.6/user-guide/index.html). We are happy to help, just [email us](mailto:hpc@yale.edu?subject=Singularity Help) with your questions.

!!!warning
    On the Yale clusters, **Singularity is not installed on login nodes.** You will need to run it from compute nodes.

## Singularity Containers

Images are the file(s) you use to run your container. Singularity images are single files that usually end in `.simg` or `.img` and are read-only by default, meaning changes you make to the environment inside the container are not persistent.

## Use a Pre-existing Container

If someone has already built a container that suits your needs, you can use it directly. You can either copy the singularity image file directly with `scp`, `rsync`, or [globus](/clusters-at-yale/data/transfer/#large-transfers-globus). You can also use [Docker Hub](https://hub.docker.com/explore/) or [Singularity Hub](https://singularityhub.github.io/containers/registry/singularity-hub-registry/) images hosted on their respective registries. Container images can take up a lot of disk space (dozens of gigabytes), so you may want to change the default location singularity uses to cache these files. To do this before getting started, you should add something like the example below to to your `~/.bashrc` file:

```
# set SINGULARITY_CACHEDIR if you want to pull files (which can get big) somewhere other than $HOME/.singularity
# e.g. 
export SINGULARITY_CACHEDIR=~/project/.singularity
```

Here are some examples of getting containers already built by someone else with singularity:

```
# from Docker Hub (https://hub.docker.com/)
singularity build ubuntu-18.10.simg docker://ubuntu:18.10
singularity build tensorflow-10.0-py3.simg docker://tensorflow/tensorflow:1.10.0-py3

# from Singularity Hub (https://singularity-hub.org/)
singularity build bioconvert-latest.simg shub://biokit/bioconvert:latest
```

## Build Your Own Container

You can define a container image to be exactly how you want/need it to be, including applications, libraries, and files of your choosing with a recipe file. Singularity recipe files are similar to Docker's `Dockerfile`, but use different syntax. To build a container from a recipe file, you need administrative privileges on a Linux machine where [Singularity is installed](https://www.sylabs.io/guides/2.6/user-guide/installation.html). If you don't have such an environment available, [get in touch with us](mailto:hpc@yale.edu?subject=Singularity Help) for help getting set up. For full recipes and more documentation please see [the singularity site](https://www.sylabs.io/guides/2.6/user-guide/container_recipes.html).

### Header

Every container recipe must begin with a header that defines what image to start with, or bootstrap from. This can be an official Linux distribution or someone else's container that gets you nearly what you want.

To start from Ubuntu Bionic Beaver (18.04 LTS):

```
Bootstrap: docker
From: ubuntu:18.04
```

Or an Nvidia developer image

```
Bootstrap: docker
From: nvidia/cuda:9.2-cudnn7-devel-ubuntu18.04
```

The rest of the sections all begin with `%` and the section name. You will see section contents indented by convention, but this is not required.

### %labels

The labels section allows you to define metadata for your container:

```
%labels
    Name
    Maintainer "YCRC Support Team" <hpc@yale.edu>Version v99.9
    Architecture x86_64
    URL https://research.computing.yale.edu/</hpc@yale.edu>
```

You can examine container metadata with the `singularity inspect` command.

### %files

If you'd like to copy any files from the system you are building on, you do so in the %files section. Each line in the files section is a pair of source and destination paths, where the source is on your host system, and destination is a path in the container.

```
%files
    sample_data.tar /opt/sample_data/
    example_script.sh /opt/sample_data/
```

### %post

The post section is where you can run updates, installs, etc in your container to customize it.

```
%post
    echo "Customizing Ubuntu"
    apt-get update
    apt-get -y install software-properties-common build-essential cmake
    add-apt-repository universe
    apt-get update
    apt-get -y libboost-all-dev libgl1-mesa-dev libglu1-mesa-dev
    cd /tmp
    git clone https://github.com/gitdudette/myapp && cd myapp
    # ... etc etc
```

### %environment

The environment section allows you to define environment variables for your container. These variables are available when you run the built container, not during its build.

```
%environment
    export PATH=/opt/my_app/bin:$PATH
    export LD_LIBRARY_PATH=/opt/my_app/lib:$LD_LIBRARY_PATH
```

### Building

To finally build your container after saving your recipe file as `my_app.def`, for example, you would run

```
singularity build my_app.simg my_app.def
```

## Use a Container Image

Once you have a container image, you can run it as a part of a batch job, or interactively.

### Interactively

To get a shell in a container so you can interactively work in its environment:

```
singularity shell --shell /bin/bash containername.simg
```

### In a Job Script

You can also run applications from your container non-interactively as you would in a batch job. If I wanted to run a script called `my_script.py` using my container's python:

```
singularity exec containername.simg python my_script.py
```

## Environment Variables

If you are unsure if you are running inside or outside your container, you can run:

```
echo $SINGULARITY_NAME
```

If you get back text, you are in your container.

If you'd like to pass environment variables into your container, you can do so by defining them prefixed with `SINGULARITYENV_` . For Example:

```
export SINGULARITYENV_BLASTDB=/home/me/db/blast
singularity exec my_blast_image.simg env | grep BLAST
```

Should return `BLASTDB=/home/me/db/blast`, which means you set the `BLASTDB` environment variable in the container properly.

## Additional Notes

### MPI

MPI support for Singularity is relatively straight-forward. The only thing to watch is to make sure that you are using the same version of MPI inside your container as you are on the cluster.

### GPUs

You can use gpu-accelerated code inside your container, which will need most everything also installed in your container (e.g. CUDA, cuDNN). In order for your applications to have access to the right drivers on the host machine, use the `--nv` flag. For example:

```
singularity exec --nv tensorflow-10.0-py3.simg python ./my-tf-model.py
```

### Home Directories

Sometimes the maintainer of a docker container you are trying to use installed software into a special user's home directory. If you need access to someone's home directory that exists in the container and not on the host, you should add the `--contain` option. Unfortunately, you will also then have to explicitly tell Singularity about the paths that you want to use from inside the container with the [`--bind`](https://www.sylabs.io/guides/2.6/user-guide/bind_paths_and_mounts.html) option.

```
singularity shell --shell /bin/bash --contain --bind /gpfs/ysm/project/be59:/home/be59/project bioconvert-latest.simg
```