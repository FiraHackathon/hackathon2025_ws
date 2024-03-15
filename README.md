## Installation

### Create workspace

You need to install `vcstool`. You can install it using pip:
```
pip install vcstool
```

Clone this project and go to the root:
```
git clone https://github.com/Tirrex-Roboterrium/tirrex_workspace.git
cd tirrex_workspace
```

### Build the docker image

The recommended method for compiling the workspace and running the programs is to use docker.
If you do not want to use docker and you are using Ubuntu 22.04 and ROS2 Humble, you can skip this
step and directly use `colcon` and `ros2` commands.
However, the following instructions assume that docker is being used.

To build the `tirrex` docker image, your first need to start a `ssh-agent` (if not already running):
```
source <(ssh-agent)
```

After that, you can 
```
docker compose build compile
```

### Compiling

#### Create docker image and compile

A docker image is available to install ROS2 and all the dependencies of the packages.
You first need to install a recent version of docker compose by [following the instruction on the
docker documentation](https://docs.docker.com/compose/install/linux/).
Then you can build the image and compile the workspace using the following command (from the root of
this project):
```
docker compose run --rm compile
```

## Installation (only for INRAE developers)

If you are an INRAE developer with an access to the projects on gitlab.irstea.fr, you can use
alternative repositories with ssh URLs.
To do that, you have to add `REPOS_FILE` in the `.env` file before creating the workspace and
building the docker image.
Execute the following commands after replacing `id_ed25519` by the correct name of your ssh key:
```bash
echo 'REPOS_FILE=repositories.private' >> .env
source <(ssh-agent)
ssh-add ~/.ssh/id_ed25519
./scripts/create_ws
docker compose run --rm --build compile
```

## Running

You can run a simple simulation test that spawn a robot in Gazebo and can be controlled using a
joypad.
You just have to start one of the docker services corresponding to the robot name with the suffix
`_test`.
The currently robot available are `adap2e`, `campero`, `ceol`, `cinteo`, `hunter`, `husky`,
`scout_mini` and `robufast`.
For example, you can test `adap2e` using the command:
```
docker compose up adap2e_test
```

Alternatively, you can start a docker service using `run` to avoid prefixes on output lines:
```
docker compose run --rm adap2e_test
```

All the available services are defined in the file [`docker-compose.yml`](docker-compose.yml).
You can add your own services if you want to easily execute specific commands in the docker
environment.

### Run other commands in docker

It is possible to open a shell in the docker.
It can be useful to execute some ROS command like `ros2 topic list` or `ros2 launch ...`.
To start the shell, use the command:
```
docker compose run --rm bash
```

It is also possible to directly specify a command in the line.
For exemple, you can manually start `adap2e_test` using:
```
docker compose run --rm bash ros2 launch adap2e_bringup adap2e_test.launch.py
```

The option `--rm` allows to automatically delete the container when the command finishes.


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
The image is built from the [Dockerfile](docker/Dockerfile) and can be edited to add other ubuntu or
pip packages.

When a docker service is started, this workspace is mounted as a volume inside the docker
container and the ROS commands are run using your own Linux user.
This makes using the tools in docker similar to using them directly.
This means that, if you have ROS Humble on your host system, you should be able to direclty run ros2
commands without using docker commands.

### Organization of the ROS packages

In the `src` directory, the packages are organized in several sub-folders:

* `romea_core` contains all packages that are independant of ROS
* `romea_ros2` contains all ROS2 packages (in several sub-folders)
* `third_party` contains packages that are not written by our team

For more details about this packages, read their README.

## Updating

All the ROS packages may evolve during the tirrex.
You can update them by using the command
```
vcs pull -nw6
```

Some projects may fail to update.
This may be due to local modifications.
You have to manually handle this projects.
However, it is possible to stash your local changes on every project before updating by running
```
vcs custom -nw6 --args pull --rebase --autostash 
```

If you want to update projects and re-download the gazebo models, you can re-run the installation
script
```
./scripts/create_ws
```
