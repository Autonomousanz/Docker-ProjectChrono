## Customized Docker image_pychrono_projectchrono-

Version 7.0.0 python 3.10
This is customized docker image for projectchrono and pychrono users 

Date: 09/11/2022
Author: Sanskruti Jadhav
| HOW TO DOCKER
1. INSTALLATION OF DOCKER ON UBUNTU OS
Prerequisites:
OS requirements: To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:
Ubuntu Jammy 22.04 (LTS)
Ubuntu Impish 21.10
Ubuntu Focal 20.04 (LTS)
Ubuntu Bionic 18.04 (LTS)
Uninstall old versions
$ sudo apt-get remove docker docker-engine docker.io containerd runc
It’s OK if apt-get reports that none of these packages are installed.
The contents of /var/lib/docker/, including images, containers, volumes, and networks, are preserved.
If you do not need to save your existing data, and want to start with a clean installation, refer to the below site:
uninstall-docker-engine
This installation guide is the same as the main docker hub installation below:
https://docs.docker.com/engine/install/ubuntu/#uninstall-docker-engine
A. Installation Method: Setup the repository
Although there are many other ways to install docker engine we will use the recommended repository method
i. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
2
$ sudo apt-get update
$ sudo apt-get install \ ca-certificates \ curl \ gnupg \ lsb-release
ii. Add Docker’s official GPG key:
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
iii. Use the following command to set up the repository:
echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
B. Install Docker Engine
3
i. Update the apt package index, and install the latest version of Docker Engine, containerd, and Docker Compose, or go to the next step to install a specific version:
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
Note: ** Receiving a GPG error when running apt-get update?
Your default umask may not be set correctly, causing the public key file for the repo to not be detected. Run the following command and then try to update your repo again: sudo chmod a+r /etc/apt/keyrings/docker.gpg.
ii. To install a specific version of Docker Engine, list the available versions in the repo, then select and install:
a. List the versions available in your repo:
$ apt-cache madison docker-ce
iii. Install a specific version using the version string from the second column, for example, I have replaced <VERSION_STRING> with 5:20.10.18~3-0~ubuntu-focal in below command
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
C. Verify Docker Engine is installed and Setup
Verify that Docker Engine is installed correctly by running the hello-world image.
$ sudo service docker start
$ sudo docker run hello-world
Thats It! now you have a running docker engine that can utilized for running docker images
4
2. RUNNING CUSTOMIZED PROJECT CHRONO IMAGE
Let's now call our project chrono image: docker : sanskrj/ubuntu: pychrono_projchrono
A. Features
Project chrono build with Irrlicht engine
Pychrono in anaconda
POV Ray
Cuda packages
MATLAB R_2022a linked with project chrono (meaning MATLAB can be called from project chrono c++ script)
B. Project Chrono Structure
the build directory and the chrono source codes are located in /opt folder,
This folder also has MATLAB install directory. Additionally anaconda3 is installed.
Now that you are in the terminal, you can do ls and you can see below output folders, the main build and folders are found in /opt folder.
It is important to have new features in /opt folder only, by default docker installs in /root folder (e.g. MATLAB, Anaconda etc is installed in /root)
we need to shift these directories to /opt because we need to access these later in singularity images form, they cannot access root folders of docker images.
This is a topic of later discussion, for now just focus on running a demo of project chrono
$ ls
C. PC Setting
Giving display for our docker to run project chrono irrlicht engine visualization is significant. Run below commands on terminal before running the docker image.
since docker can be run by sudoers only, we need to add access to root user allows the root user to access the running X server.
The current X server is indicated by the DISPLAY environment variable
$ xhost +si:localuser:root
$ echo $DISPLAY
return argument is :1/0(depending on your system)
Take note of this.
$ xauth list
5
The xauth command is usually used to edit and display the authorization information used in connecting to the X server.
This will give an MIT magic cookie name of my pc as well a hex key
in above command :1 is my echo $DISPLAY output additionally after the first part it is necessary to add . before hex key
$ xauth add sanskruti-Lenovo-Legion-5-15ARH05/unix:1 . 06b578f1f32c4921ad7ac0559db142d8
D. How to Run the Image
In below detail just replace Image -name as sanskrj/ubuntu:projchro_matlab2
$ sudo docker run -it --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix --net=host <image-name >
This will bring you to the terminal of the Ubuntu OS , it acts exactly as we type commands in terminal except you need not use sudo.
Now if you want you can visualize this as a normal virtual box OS as well which will display all of it in files and folders just as normal OS, this requires additional processes of using VNC+ docker
But for our purposes let's start with a simple command line interface of docker image.
6
7
$ apt-get update
As I have already build all the project chrono, you have all the executable demos present in this path
$ cd /opt/build_chrono/bin
$ ./demo_HMMV_VEH
This will output a HMMV vehicle in an environment and you can interact with using keyboard keys a,w,d,s for right turn, forward, left turn and stop respectively.
E. Modify Image
If you make changes in chrono/ src containing the source code in c++ of various modules of vehicle you will need to re-build the chrono as below
$ cd /opt/build_chrono
$ ccmake ../chrono
This will give a CMAKE GUI as below and you need to configure and generate pressing C and g respectively. It will automatically exit once generation is done .Now we need to make and make install,
$ make
$ make install
This will update all the executables (since they are already build they are mostly skipped but still they are shown in make output)
the code which you updated will now have the corresponding executable updated in the same /build_chrono/bin library with output of the run generated inside DEMO_OUTPUT folder
F. Commit to the changes in Docker Image
10
After having made a lot of changes, adding packages , new apps etc you will need to commit to changes or they will not persist next time you run the image and form a new image by committing to your docker hub a account. You will need your account on docker hub and this image will be stored in repository publicly available to all.
$ exit
$ sudo docker ps –a
$ sudo docker commit <containerID> {repository:tag}
e.g. docker commit c3f279d17e0a sanskrj/ubuntu:projectchrono
$sudo docker login
$ sudo docker push
Details for how to commit is present in docker :
https://docs.docker.com/engine/reference/commandline/commit/
3. RUNNING CUSTOMIZED PYCHRONO IMAGE
Let's now call our pychrono
docker: sanskrj/ubuntu: pychrono_projchrono
Once in the shell, activate chrono environment with python 3.10 also has irrlicht package installed in this environment.
The environment is build using the below webpage information:
https://api.projectchrono.org/pychrono_installation.html
$ conda activate chrono
$ cd /opt/anaconda3/pkgs/pychrono-7.0.0-py310_2116/lib/python3.10/site-packages/pychrono/demos/vehicle/
$ python demo_VEH_HMMV.py


