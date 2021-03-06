Description
-----------
This repository contains a Python script that allows you to launch a docker 
container with X11 graphics support. 

Typical use case
----------------
A typical use case of this script is when you are connecting via ssh from your 
laptop to a remote computer (e.g. a DGX server) and you want to launch a docker 
container inside the remote computer with X11 support. A quick diagram:

Laptop => Remote computer (connected via ssh) => Docker container 

You want to launch a graphical application inside the Docker container and see the GUI in your laptop.
   
Requirements
------------
If you are launching this script on a server (e.g. DGX) you need to edit the 
configuration file of the SSH server -> ```/etc/ssh/sshd_config``` and
add the option:

``` X11UseLocalhost no ```

To edit ```/etc/ssh/sshd_config``` you need superuser access. After editing 
this file you need to run:

```bash
$ sudo service ssh reload
```

This will reload the SSH server configuration without disconnecting existing 
sessions. 

Install using pip
-----------------
```bash
$ python3 -m pip install dockerx --user
```

Install this package from source
--------------------------------
```bash
$ sudo apt install python3 python3-pip
$ python3 -m pip install docker argparse --user
$ git clone https://github.com/luiscarlosgph/dockerx.git
$ cd dockerx
$ python3 setup.py install --user
```

Launch containers from your terminal
------------------------------------
To launch a container and execute a specific command inside the container:
```bash
$ python3 -m dockerx.run --image <image name> --nvidia <0 or 1> --command <shell command>
```
For example:
```
$ python3 -m dockerx.run --image nvidia/cuda:11.0-base --nvidia 1 --command '/bin/bash -c "apt update && apt install -y x11-apps && xclock"'
```
This should display ```xclock``` in your local screen.

The idea behind the ```--command``` parameter is to use it for launching jobs inside the 
container that require X11 support. No console output will be shown when running a command 
with the ```--command``` option.

If ```--command``` is not specified, the default command executed inside the container is that 
defined by the CMD keyword in the Dockerfile of your image. If None is defined (as happens for 
many images such as ```ubuntu``` or ```nvidia/cuda:11.0-base```), the container will start, 
do nothing, and stop immediately. 

If you want to run a container forever so you can bash into it with ```docker exec -it <container id> /bin/bash```
and run GUIs inside the container, simply run:
```bash
$ python3 -m dockerx.run --image <image name> --nvidia <0 or 1> --command 'sleep infinity'
```

For example, to run just an ```ubuntu``` container:
```bash
$ python3 -m dockerx.run --image ubuntu --command 'sleep infinity'

To get a container terminal run:  docker exec -it b05bd722477e /bin/bash
To kill the container run:        docker kill b05bd722477e
To remove the container run:      docker rm b05bd722477e

$ docker exec -it b05bd722477e /bin/bash
root@b05bd722477e:/# apt update && apt install -y x11-apps
root@b05bd722477e:/# xclock
```
After running ```xclock``` above you should see a clock in your local screen.

To run an ```ubuntu``` container **with CUDA support**:

```bash
$ python3 -m dockerx.run --image nvidia/cuda:11.0-base --nvidia 1 --command 'sleep infinity'

To get a container terminal run:  docker exec -it 0b2b964b8b8f /bin/bash
To kill the container run:        docker kill 0b2b964b8b8f
To remove the container run:      docker rm 0b2b964b8b8f

$ docker exec -it 0b2b964b8b8f /bin/bash
root@0b2b964b8b8f:/# nvidia-smi
Thu Apr 15 23:42:59 2021
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.39       Driver Version: 460.39       CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  TITAN X (Pascal)    Off  | 00000000:01:00.0 Off |                  N/A |
| 23%   27C    P8     9W / 250W |     71MiB / 12195MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
root@0b2b964b8b8f:/# apt update && apt install -y x11-apps
root@0b2b964b8b8f:/# xclock
```

As in the example above, ```xclock``` should be now shown in your local display.
However, this container has CUDA support. GPU applications can now be executed
and displayed from within the container.

Launch containers from your Python code
---------------------------------------
Exemplary code snippet that shows different ways to launch containers using the 
Python module in this repo. 

```python
import dockerx

dl = dockerx.DockerLauncher()

# If no command is specified here, the CMD in your Dockerfile will be executed, if there is no CMD in your 
# Dockerfile either, then this container will be created and immediately destroyed
container_0 = dl.launch_container('ubuntu')
print(container_0.id)

# If a command is specified here, the CMD in your Dockerfile will be ignored and overridden by the command 
# specified here
container_1 = dl.launch_container('ubuntu', command='sleep infinity')
print(container_1.id)

# Launch a container with CUDA support (as a command is specified, the CMD in your Dockerfile will be ignored)
container_2 = dl.launch_container('nvidia/cuda:11.0-base', command='sleep infinity', nvidia_runtime=True)
print(container_2.id)
```

Run unit tests
--------------
```bash
$ python3 tests/test_docker_launcher.py
```

License
-------
The code in this repository is released under an [MIT license](https://github.com/luiscarlosgph/docker-with-graphics/blob/main/LICENSE).
