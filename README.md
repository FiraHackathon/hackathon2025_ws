This project corresponds to the workspace of the FIRA 2025 hackathon.
It allows installing everything required to run the challenges and demos of the hackathon.
It is based on the [TIRREX workspace](https://github.com/Tirrex-Roboterrium/tirrex_workspace)
developed for the [_roboterrium_ platform of the TIRREX project](https://tirrex.fr/plateforme/roboterrium/).


## Contents

* [Installation](#installation)
* [Installation(for INRAE developers)](#installation-only-for-inrae-developers)
* [Usage](#usage)
* [Architecture of the workspace](#architecture-of-the-workspace)
* [FAQ](#faq)

Other documents available:

* [Scenario of the challenge](/doc/challenge.md)
* [Configuration of the simulation](/doc/challenge_configuration.md)
* [Configuration of the robot](/doc/robot_configuration.md)
* [Configuration of the devices](/doc/devices_configuration.md)
* [Description of the robot control node](/doc/robot_control.md)
* [Description of the provided data files](/doc/plots_surveying.md)


## Installation

### Create workspace

You need to install `vcstool`. You can install it using pip:
```
pip3 install vcstool
vcs --version
```

If the `vcs` command is not found, you can follow [these
instructions](#the-vcs-command-is-not-found).
Alternatively, if you use Ubuntu 20 or 22, you can install `vcs` using `apt`:
```
sudo apt install python3-vcstool
```
and for Ubuntu 24, the package is now named `vcstool`:
```
sudo apt install vcstool
```

Clone this project and go to the root:
```
git clone https://github.com/FiraHackathon/hackathon2025_ws.git
cd hackathon2025_ws
```

All the following commands must be run from the root of this project.
Execute this script to clone all the ROS packages and download the gazebo models:
```
./scripts/update
```

### Build the docker image and compile

The recommended method for compiling the workspace and running the programs is to use docker.
If you do not want to use docker, and you are using Ubuntu 22.04 and ROS Humble, you can skip these
steps and directly use `colcon` and `ros2` commands.
However, the following instructions assume that docker is being used.

You first need to install a recent version of docker compose by [following the instruction on the
docker documentation](https://docs.docker.com/compose/install/linux/).
If it is already installed, you can check that its version is greater or equal to `2.20` using the
command:
```
docker compose version
```

By default, running docker is only available for root user.
You have to add your user to the `docker` group (and create it if it does not exist) to execute
docker commands with your own user.
```
sudo groupadd docker
sudo usermod -aG docker $USER
```
Bear in mind that by executing these commands, you are giving your user privileges equivalent to
those of root using docker commands.
For more details, see [Docker Daemon Attack
Surface](https://docs.docker.com/engine/security/#docker-daemon-attack-surface)

Once your user is added to the `docker` group, you need to reboot (or log out / log in and restart
the docker daemon).

After that, you can build the image (the first time) and compile the workspace:
```
docker compose run --rm compile
```
This command will:
* pull the latest image containing a ROS environment with all the workspace dependencies installed
* build a local image based on the previous one but including a copy of your local
  user in order to execute every command using the same user as the host system
* run the `compile` service that execute a `colcon build` command to compile everything

If you modify some packages of the workspace, you need to re-execute this command regularly.


## Installation (only for INRAE developers)

If you are an INRAE developer with access to the projects on gitlab.irstea.fr, you can use
alternative repositories with ssh URLs.
To do that, you have to add `REPOS_FILE` in the `.env` file before creating the workspace and
building the docker image.
Execute the following commands:
```bash
git clone git@github.com:FiraHackathon/hackathon2025_ws.git
cd hackathon2025_ws
echo 'REPOS_FILE=repositories.private' >> .env
./scripts/update_ws
docker compose run --rm --build compile
```

## Usage

You can run the hackathon demo by starting the services in `demos/hackathon`
```
cd demos/hackathon
docker compose up
```

More details in the [README of the demo](/demos/hackathon/README.md).

### Run other commands in docker

It is also possible to open a shell on the docker using the `bash` service:
```
docker compose run --rm bash
```

The option `--rm` allows to automatically delete the container when the command finishes.

### Updating

You can update the ROS packages using:
```
./scripts/pull
```
or simply
```
vcs pull -nw6
```

If you want to load repositories, switch to the correct branches and update gazebo models, you
can re-run the installation script
```
./scripts/update
```


## Architecture of the workspace

Here is a brief presentation of the main directories:

* `build` contains the files generated by the compilation step
* `docker` contains the configuration files for docker
* `gazebo` contains the downloaded gazebo models that are too large to be versioned
* `install` contains the shared files and executables of the ROS packages
* `log` contains the logged information of the compilation
* `scripts` contains some useful tools to prepare the workspace
* `src` contains all the cloned ROS packages

### How the docker image works

The docker image corresponds to an Ubuntu 22.04 image with a complete installation of ROS2 Humble
and all the dependencies of the ROS2 packages of this workspace.
The image is built from the [Dockerfile](docker/Dockerfile) and can be edited to add other Ubuntu or
pip packages.

When a docker service is started, this workspace is mounted as a volume inside the docker
container and the ROS commands are run using your own Linux user.
This makes using the tools in docker similar to using them directly.
This means that, if you have ROS Humble on your host system, you should be able to directly run ros2
commands without using docker commands.

### Organization of the ROS packages

In the `src` directory, the packages are organized in several sub-folders:

* `romea_core` contains all packages that are independent of ROS
* `romea_ros2` contains all ROS2 packages (in several sub-folders)
* `third_party` contains packages that are not written by our team

For more details about these packages, read their README.


## FAQ

### The `vcs` command is not found

If you have installed this tools using `pip` or `pip3` outside a virtual env, then the executable
are located in the `~/.local/bin` directory.
You can make them available by adding this path into your `PATH` environment variable:
```bash
export PATH="$PATH:$HOME/.local/bin"
```


### The service `compile` does not exist

If you are in a subdirectory that contains a `compose.yaml` file, the `docker compose` command will
load this file instead of the `compose.yaml` at the root of the workspace.
The `compile` service is only defined in the one at the root, so you need to move to the root before
executing `docker compose run --rm compile`.

### How to use NVIDIA GPU?

You need to install the [NVIDIA container
toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

After that, edit the file `compose.yaml` of your demo to replace the service `x11_base` by
`x11_gpu`.
For example, the beginning of `demos/hackathon/compose.yaml` should look like to this:
```yaml
x-yaml-anchors:
  base: &base
    extends:
      file: ../../docker/common.yaml
      service: x11_gpu
    ...
```
These services are defined in the file `docker/common.yaml` of the workspace and provide all the
required docker options.
The `x11_gpu` service add special options to use the NVIDIA GPUs.

### Gazebo is slow

Gazebo comprises a physics engine (`gzserver` program) and a rendering engine (`gzclient` program).
You can check the rendering performance by looking at the _FPS_ (frame per seconds) value at the
bottom of the gazebo window, and the physics performance by looking at the _real time factor_.
The rendering engine is configured to run at 60 FPS and a real time factor of 1.0, but you may
experience slow down if your computer is not powerful enough.
By default, the docker does not use NVIDIA GPU, but it can significantly improve gazebo performance
to use it.
If you have one, you can refer to [How to use NVIDIA GPU?](#how-to-use-nvidia-gpu).
If you do not have one, you can increase FPS by disabling shadows by following instructions of
[Gazebo is dark](#gazebo-is-dark).
You can also improve the _real time factor_ by increasing `max_step_size` in the `Physics` component
of gazebo, but it will also degrade its behavior.

### Gazebo is dark

When using Gazebo with no GPU, you may experience a graphical bug that seems to cause rendering to
be darker than it should be.
This is due to the fact that shadow calculation is buggy with Intel graphics chipsets.
You can disable shadow rendering by clicking on `Scene`, then unchecking `shadows`.
If it is already unchecked, check it and uncheck it again (this is another bug).
