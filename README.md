# Motion Planning and Decision Making for Autonomous Vehicles

In this project, I have implemented two of the main components of a traditional hierarchical planner: The Behavior Planner and the Motion Planner. Both work in unison to be able to:

1. Avoid static objects (cars, bicycles and trucks) parked on the side of the road (but still invading the lane). The vehicle must avoid crashing with these vehicles by executing either a “nudge” or a “lane change” maneuver.

2. Handle any type of intersection (3-way, 4-way intersections and roundabouts) by STOPPING in all of them (by default)

3. Track the centerline on the traveling lane.

I have also edited variable ```P_STOP_LINE_BUFFER``` in ```planning_params.h``` to 2.15m ensure the car stops well below the required position. This was changed from 0.5m because 0.5m is very short time that does not account for other delays and causes the vehicle to move past the stop line.

## Dependency - Udacity VM Workspace
The state code in this repository is aligned to run on the Udacity VM workspace. Refer to the classroom page **Ubuntu VM Workspace - Overview** to learn how to access the VM workspace and its restrictions and best practices. 

However, to set up your local machine with the necessary tools, you must have either *Windows Subsystem for Linux* (WSL) or *Ubuntu 20.04* or *18.04 LTS*. Below is the list of tools installed in the Udacity VM workspace that you should install on your local machine.

- [CARLA simulator 0.9.9.4](https://github.com/carla-simulator/carla/releases/tag/0.9.9). <br/>
    You can find more details at [CARLA Quick Start Installation](https://carla.readthedocs.io/en/latest/start_quickstart/). The deb installation is the easiest way to get the latest release in Linux.
    ```bash
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1AF1527DE64CB8D9
    sudo add-apt-repository "deb [arch=amd64] http://dist.carla.org/carla $(lsb_release -sc) main"
    sudo apt-get update # Update the Debian package index
    sudo apt-get install carla-simulator=0.9.10-2 
    ```

    The installation directory must be **/opt/carla-simulator/** on your Linux machine. To verify, open a terminal an launch CARLA as:
    ```bash
    cd /opt/carla-simulator
    ./CarlaUE4.sh
    ```
    The Carla Simulator should launch in a few seconds. You can close it after verification. 


- [NICE DCV Server](https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-installing-linux-prereq.html). <br/>
    This includes the Nvidia drivers along with CUDA libraries for the underlying Tesla T4 GPU.

    ```bash
    Sat Oct 14 15:31:45 2023       
    +---------------------------------------------------------------------------------------+
    | NVIDIA-SMI 535.104.12             Driver Version: 535.104.12   CUDA Version: 12.2     |
    |-----------------------------------------+----------------------+----------------------+
    | GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
    |                                         |                      |               MIG M. |
    |=========================================+======================+======================|
    |   0  Tesla T4                       On  | 00000000:00:1E.0 Off |                    0 |
    | N/A   31C    P0              27W /  70W |   2093MiB / 15360MiB |     27%      Default |
    |                                         |                      |                  N/A |
    +-----------------------------------------+----------------------+----------------------+
                                                                                            
    +---------------------------------------------------------------------------------------+
    | Processes:                                                                            |
    |  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
    |        ID   ID                                                             Usage      |
    |=======================================================================================|
    |    0   N/A  N/A      1055      G   /usr/lib/xorg/Xorg                           67MiB |
    |    0   N/A  N/A      1521      G   /usr/lib/xorg/Xorg                           89MiB |
    |    0   N/A  N/A      1669      G   /usr/bin/gnome-shell                         23MiB |
    |    0   N/A  N/A      1948    C+G   /usr/lib/x86_64-linux-gnu/dcv/dcvagent      398MiB |
    |    0   N/A  N/A      3320      G   ...sion,SpareRendererForSitePerProcess       30MiB |
    |    0   N/A  N/A      4489    C+G   ...aries/Linux/CarlaUE4-Linux-Shipping     1348MiB |
    +---------------------------------------------------------------------------------------+
    ```


    ```bash
    dcv version
    # Output
    NICE DCV 2023.0 (r15487)
    Copyright (C) 2010-2023 NICE s.r.l.
    ```

- C++ 
    ```bash
    gcc --version
    # Output
    gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0
    ```
- Git
- [OpenCV](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html)
- [CMake](https://askubuntu.com/questions/161104/how-do-i-install-make) and Make
- [VSCode](https://code.visualstudio.com/download)
- [Eigen Library for C++](https://eigen.tuxfamily.org/index.php?title=Main_Page)
- [Point Cloud Library](https://pointclouds.org/downloads/)
- Python3 and Pip v20.xx or above. 
    ```bash
    python3 --version
    # Output
    Python 3.8.10
    ```
- ROS

- Project specific dependencies
    ```bash
    # Required for building project
    sudo apt-get install -y libgoogle-glog-dev libgtest-dev
    # Required for running project. 
    # Install carla python package
    sudo python3 /usr/lib/python3/dist-packages/easy_install.py /opt/carla-simulator/PythonAPI/carla/dist/carla-0.9.9-py3.7-linux-x86_64.egg
    # Install python requirements
    pip install numpy pygame websocket-client
    ```


# Instructions
The sections ahead will guide you through the steps to build and run the project. 


## Step 1. Log into VM Workspace

Open the VM workspace and log into the VM to practice the current project. 
Once you log into the VM, open a Terminal window. 

<br/><br/>

## Step 2. Clone this Repository

Fork the repository to your Github account and clone the repository to the workspace using the commands below. 

```bash
git clone https://github.com/udacity/nd013-c5-planning-starter.git
```

Change to the project directory.
```bash
cd nd013-c5-planning-starter/project
```

<br/><br/>

## Step 3. Review the starter files
You will find the following files in the project directory.

```bash
.
├── cserver_dir
├── install-ubuntu.sh
├── manual_control.py
├── run_carla.sh
├── run_main.sh
├── simulatorAPI.py
└── starter_files
```

<br/><br/>

## Step 4. Start the Carla Server
Start the Carla server by executing the following shell script. 
```bash
./run_carla.sh
```
This file will, in turn, execute the **/opt/carla-simulator/CarlaUE4.sh** script. 


<br/><br/>

## Step 5. Install Dependencies
Open another Terminal tab, and change to the **nd013-c5-planning-starter/project**  directory. Execute the following shell script to install the project-specific dependencies. 
```bash
./install-ubuntu.sh
```
This file will install utilities such as, `libuv1-dev`, `libssl-dev`, `libz-dev`, `uWebSockets`. 

You may also need to install `websocket` and `pygame` if you encounter compile error by executing these two lines:
```
pip install websocket-client
pip install pygame
```

### Update Notes
In the previous version of the project starter code, we had **libcarla-install/** and **rpclib/** directories inside the starter files. But, those directories are no longer needed in the current version of the starter code because the current **CMakeLists.txt** file has corresponding `includes` and `libs` added at `/opt/carla-source`.

To give some old context, when we had **rpclib/** directory inside the starter files, we used to compile the **rpclib** library using the following commands. 
```bash
cd starter_files
rm -rf rpclib
git clone https://github.com/rpclib/rpclib.git
```
This library is a **msgpack-rpc** library written using modern C++. The goal of building this library was to provide a simple RPC solution. However, all of the above-mentioned steps are **no longer needed** in the current version of the project strarter code. 

<br/><br/>

## Step 6. Build and Execute the Project

You can execute the project using the commands below. 

```bash
# Build the project
# Run the following commands from the starter_files/ directory
cmake .
# The command below compiles your c++ code. Run it after each time you edit the CPP or Header files
make
```

```bash
# Run the project
cd ..
# Run the following commands from the nd013-c5-planning-starter/project directory
./run_main.sh
```
If the execution fails silently, you can use **ctrl + C** to stop, and try again. 

Another possible error you may get is `bind failed. Error: Address already in use`. In such a case, you can kill the process occupying the required port using the commands below.

```bash
ps -aux | grep carla
# Use the IDs displayed in the output of the last command. 
kill id     
```

<br/><br/>