# syntax=docker/dockerfile:1
FROM ubuntu:latest
#RUN apt-get update && apt-get install -y screen
RUN apt-get update &&  apt-get install --no-install-recommends -y xterm wget bash nano sudo python3 git
#Visual Studio Code install Per: https://linuxize.com/post/how-to-install-visual-studio-code-on-ubuntu-20-04/
#RUN apt update && apt install -y software-properties-common apt-transport-https wget 
#RUN wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | apt-key add -
#RUN add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
#RUN apt install -y code



ENV DISPLAY=host.docker.internal:0.0

# Create new user in container - so we aren't doing everything as root
RUN adduser --disabled-password --gecos '' mesh_docker_dev
USER mesh_docker_dev
#Create the home directory for the user
WORKDIR /home/mesh_docker_dev

# Setting passwords for Root and User accoutn allows to switch between the two accounts
#RUN echo 'root:Docker!' | chpasswd
USER root
RUN echo 'root:Docker!' | chpasswd
RUN echo 'mesh_docker_dev:Docker!' | chpasswd

# add new user to sudoers group
RUN usermod -a -G sudo mesh_docker_dev

# PlatformIO installation from here: https://lindevs.com/install-platformio-core-on-ubuntu
RUN apt-get install --no-install-recommends -y python3-venv 
RUN wget -q https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py
RUN PLATFORMIO_CORE_DIR=/opt/platformio python3 get-platformio.py
RUN ln -s /opt/platformio/penv/bin/pio /usr/local/bin/pio
RUN usermod -a -G dialout mesh_docker_dev

USER mesh_docker_dev
# Create a vscode execution directory
RUN mkdir ~/vscode_dir

# Copy this dockefile used to build this container into home for traceability 
COPY Dockerfile ~/Dockerfile_to_build_this_container
 
#RUN make /app
#CMD python /app/app.py