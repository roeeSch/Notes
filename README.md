# General Notes



## ROS

#### running ROS on 2 machines

In order to run see\read\display topics between two machines run the following commands:

on master machine  (where roscore runs) run:

1. `$export ROS_IP=ip_of_master`

on client machine (not where roscore runs) run:

1. `$export ROS_MASTER_URI=http://ip_addr_of_master:11311`
2. `$export ROS_IP=ip_of_client`

**<u>note:</u>** without the export of ROS_IP, only meta data of messages is available.



## Docker



```$docker images``` - list images

An **image** is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.

A container is launched by running an image.

``$docker ps `` (list of running containers)

A container is a runtime instance of an image--what the image becomes in memory when executed (that is, an image with state, or a user process)

``$docker --version`` - version of installed docker
``$docker version`` - more details

`docker container ls --all`  - the `all` flag lists also containers that have exited.



#### Cheat sheet

```shell
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

The hierarchy of docker application:

A container, which [this page](https://docs.docker.com/get-started/part2/) covers. Above this level is a service, which defines how containers behave in production, covered in [Part 3](https://docs.docker.com/get-started/part3/). Finally, at the top level is the stack, defining the interactions of all the services, covered in [Part 5](https://docs.docker.com/get-started/part5/).

- Stack
- Services
- Container (Define a container with `Dockerfile`)



#### Create hector-slam docker:

1. Create a Dockerfile (based on [link](https://github.com/osrf/docker_images/blob/460ddc4707530c2179788e2100d5c624cf2af3d7/ros/kinetic/ubuntu/xenial/desktop-full/Dockerfile)):

```bash
# This is an auto generated Dockerfile for ros:desktop-full
# generated from docker_images/create_ros_image.Dockerfile.em
FROM osrf/ros:kinetic-desktop-xenial

# install ros packages
RUN apt-get update && apt-get install -y \
    ros-kinetic-desktop-full=1.3.2-0* \
    && rm -rf /var/lib/apt/lists/*
```

2. run `$docker build --tag=ros-desktop-full .` 

These interactive shells will exit after running any scripted commands, unless they are run in an interactive terminal - so for this example to not exit, you need to docker run -it alpine /bin/sh.

3. run `$docker run -it ros-desktop-full /bin/sh ` (interactive mode) Now you are in the container.

   In a different shell you could see the running container by typing `docker container ls -a`.

After some progress (init catkin workspace, install hector_slam.... and so on) don't exit container (all installations will be deleted!) a commit is needed in order to save our work:

4. commit changes


   obtain container ID: `docker container ls`:

```bash
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS               NAMES
fd275287a981        ros-desktop-full:latest   "/ros_entrypoint.sh …"   17 minutes ago      Up 17 minutes                           vibrant_thompson

```

​	then, commit the changes:

   `$docker commit fd275287a981 roees/ros-kinetic:latest`

​	and push to docker-hub:

   `$docker pull roees/ros-kinetic:latest`



​	now when running `docker images`:

   ```bash
REPOSITORY          TAG                      IMAGE ID            CREATED             SIZE
roees/ros-kinetic         latest            0bbccef193a9        6 minutes ago       3.41GB
ros-desktop-full    latest                   15f9986beb1f        25 minutes ago      3.37GB
osrf/ros            kinetic-desktop-xenial   71161c7adeb0        4 days ago          2.32GB
alpine              latest                   961769676411        11 days ago         5.58MB

   ```

5. return to previous commit:

   `$docker run -it roees/ros-kinetic:latest`



#### To use this specific docker for running hector-slam:

obtain an up to date commit:

`docker pull roees/ros-kinetic:latest` 

<u>Background:</u> Master side generates the /scan messages (either from simulation or bag record). The docker uses the /scan messages with hector slam to produce a map which is published under the /map topic. This docker implements the client side.



run docker:

`docker run roees/ros-kinetic:latest`

Setup client environment variables:

```bash
root@a4e62f7648f0:/home/tmp/catkin_ws#export ROS_MASTER_URI=http://10.59.128.24:11311
root@a4e62f7648f0:/home/tmp/catkin_ws#export ROS_IP=172.17.0.2
```

**<u>note:</u>** Also needed in the master side (export ROS_IP). See section **running ROS on 2 machines** for more information.

Then source setup:

```bash
root@a4e62f7648f0:/home/tmp/catkin_ws#source devel/setup.bash
```

run the launch file:

```bash
/home/tmp/catkin_ws# roslaunch hector_slam_launch tutorial_roee.launch base_frame:="base_link" odom_frame:="base_link"
```

**<u>note:</u>** if your main frame is `base_frame` then omit the arguments in the last command.