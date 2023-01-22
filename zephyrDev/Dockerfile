# syntax=docker/dockerfile:1
# 
#
# 1.) RUN ON WINDOWS mounting a seperate directory (OneDrive\Documents\git\zephyr_fs) for storage of source
# docker run --name zephyr_v1_4 -v C:\Users\maxbe\OneDrive\Documents\git\zephyr_fs:/home/zephyr_dev/zephyrproject/zephyr_fs --rm -i -t mbezold/zephyr_ubuntu:v2.0 bash
#
#
# 2.) RUN On Ubuntu passing USB dev directtory and mounts for X11 to allow GUI and flashing of hardware. 
#docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb -v /tmp/.X11-unix:/tmp/.X11-unix -v /dev/bus/usb:/dev/bus/usb -e DISPLAY=unix$DISPLAY --name x11_demo99 mbezold/zephyr_ubuntu:v2.0 /bin/bash
#
#
#sudo docker run -it --name zephyr_ubuntu_jtag5 --privileged -v /dev/bus/usb:/dev/bus/usb # mbezold/zephyr_ubuntu:v2.0 /bin/bash
#

#


FROM ubuntu:latest

ENV PYTHONUNBUFFERED 1
# prevents debpackages from running interactively 
# https://techoverflow.net/2019/05/18/how-to-fix-configuring-tzdata-interactive-input-when-building-docker-images/
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
apt-get upgrade -y && \
apt-get install --no-install-recommends -y git ninja-build 

# -y option causes apt-get to autmatically say yes when prompted - important for headless dockerfile create process
RUN apt-get install --no-install-recommends -y  gperf ccache dfu-util device-tree-compiler  
RUN apt-get install --no-install-recommends -y  python3-dev python3-pip python3-setuptools python3-wheel 
RUN apt-get install --no-install-recommends -y  xz-utils file make gcc gcc-multilib g++-multilib libsdl2-dev
RUN apt-get install --no-install-recommends -y cmake 
RUN apt-get install --no-install-recommends -y wget
RUN apt-get install --no-install-recommends -y python3.8-venv
RUN apt-get install --no-install-recommends -y less

# Download, inspect and execute the Kitware archive script to add the Kitware APT repository to your sources list. A detailed explanation of kitware-archive.sh can be found here kitware third-party apt repository:
RUN wget https://apt.kitware.com/kitware-archive.sh 
RUN bash kitware-archive.sh

# add a user (without a password) for non-root installation and later operations 
RUN adduser --disabled-password --gecos '' zephyr_dev

USER zephyr_dev

WORKDIR /home/zephyr_dev

# setup virtual python environment in the local users account
RUN mkdir zephyrproject
WORKDIR /home/zephyr_dev/zephyrproject
RUN python3 -m venv ./.venv
WORKDIR /home/zephyr_dev/zephyrproject/.venv/

# Need  to run as bash - cshell is used by default in docker file
RUN bash bin/activate

# Install west:
RUN pip install west 

# Set path to include West's path
ENV PATH /home/zephyr_dev/.local/bin:$PATH

# Get the Zephyr source code:
RUN west init ~/zephyrproject 
RUN cd ~/zephyrproject 
WORKDIR /home/zephyr_dev/zephyrproject
RUN west update


# Export a Zephyr CMake package. This allows CMake to automatically load boilerplate code required for building Zephyr applications.
RUN west zephyr-export

# Zephyr’s scripts/requirements.txt file declares additional Python dependencies. Install them with pip.
WORKDIR /home/zephyr_dev/zephyrproject
RUN pip install -r zephyr/scripts/requirements.txt


### SDK INSTALLER ####
# Download the latest SDK installer
WORKDIR /home/zephyr_dev/zephyrproject
RUN wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.13.2/zephyr-sdk-0.13.2-linux-x86_64-setup.run

#Run the installer, installing the SDK in ~/zephyr-sdk-0.13.2:
RUN chmod +x zephyr-sdk-0.13.2-linux-x86_64-setup.run 
RUN bash ./zephyr-sdk-0.13.2-linux-x86_64-setup.run -- -d ~/zephyr-sdk-0.13.2

# switch to root in order to install a version of cmake that definitely will work with the build tools
# UNCLEAR why this needs tobe done.
USER root
RUN apt-get install -y --no-install-recommends cmake
USER zephyr_dev

# Set Path for west to be used by the user when calling West meta-tool
# NOTE - this only applies in the Docker build process, will need to set path in
# bashrc when building this container interatctively.
ENV PATH="/home/zephyr_dev/.local/bin:${PATH}"


# Test the Build chain by building a test example 
WORKDIR /home/zephyr_dev/zephyrproject/zephyr
RUN west build -p auto -b efm32pg_stk3401a samples/basic/blinky

USER root
WORKDIR /home/zephyr_dev/

# Install JLINK from SEGGER (JTAG debug driver)
# File source https://www.segger.com/downloads/jlink/JLink_Linux_x86_64.deb
#gedbi installs the dpkg's dependancies
RUN apt-get install -y --no-install-recommends gdebi
COPY JLink_Linux_V760g_x86_64.deb /home/zephyr_dev/
RUN yes | gdebi /home/zephyr_dev/JLink_Linux_V760g_x86_64.deb

# Install some other useful programs 
RUN apt-get install --no-install-recommends -y nano vim sudo

# Password for root - allows us to switch back and forth between root and regular user.
RUN echo 'root:Docker!' | chpasswd

# Switch back to regular user to setup environment variables in bashrc
USER zephyr_dev
WORKDIR /home/zephyr_dev/
RUN echo 'export PATH=/home/zephyr_dev/.local/bin:${PATH}' >> ~/.bashrc
RUN echo 'export PATH=/usr/bin:${PATH}' >> ~/.bashrc

# change password so we run "sudo" with this account
#RUN echo 'Docker!' | passwd --stdin zephyr_dev

# zephyr_dev part of sudoers group
USER root
RUN echo 'zephyr_dev:Docker!' | chpasswd
RUN usermod -a -G sudo zephyr_dev
USER zephyr_dev


#install open OCD dependancies and Eclipse dependances and make them accessible via udev
USER root
RUN apt-get install --no-install-recommends -y udev 
WORKDIR /home/zephyr_dev/
#RUN cp ./zephyr-sdk-0.13.2/sysroots/x86_64-pokysdk-linux/usr/share/openocd/contrib/60-openocd.rules /etc/udev/rules.d
#RUN udevadm control --reload
USER zephyr_dev

# Install Eclipse depedenancies (including SSH)
USER root
WORKDIR /home/zephyr_dev/
RUN apt-get update && \
apt-get upgrade -y && \ 
apt-get install --no-install-recommends -y openjdk-11-jre openjdk-11-jdk ssh x11vnc xvfb xauth xfce4 xfce4-goodies tightvncserver
RUN apt-get install --no-install-recommends -y  xfonts-base
COPY eclipse-inst-jre-linux64.tar.gz /home/zephyr_dev
RUN tar -xvzf eclipse-inst-jre-linux64.tar.gz


# Required for tightvnc usage
USER zephyr_dev
WORKDIR /home/zephyr_dev/
RUN echo 'export USER=zephyr_dev' >> ~/.bashrc



# Presumabley exposing port for GDB to do things
EXPOSE 4444

#EXPOSE port for SSH
EXPOSE 22

USER root
# restart SSH service
RUN service ssh restart


# Will need to be done interactively. 
WORKDIR /home/zephyr_dev/eclipse-installer
#RUN BASH ./eclipse-inst

# Restart ssh with new settings
EXPOSE 59000-59010
EXPOSE 6000-6010

USER zephyr_dev

