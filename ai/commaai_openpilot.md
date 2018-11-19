# Notes to Commaai's openpilot

## Before starting
    1. openpilot, autoware, apollo... all need the docker environment.
    2. But you can do some handmade procedures to run the code outside docker.

## Ubuntu 16.04+ install Docker CE
1. Add Docker CE AP image source:
    *  sudo apt-get update
    *  sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    *  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    *  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

2. Install Docker CE
    *  sudo apt-get update
    *  sudo apt-get install docker-ce
    *  sudo usermod -aG docker $USER  
       [note] Only root and users from docker can access docker's unix socket, so add yourself into docker group.
       
3. Start Docker CE (maybe not needed)
    *  sudo systemctl enable docker
    *  sudo systemctl start docker

4. Commands to test docker environment
    *  docker run hello-world
    *  docker run -it ubuntu bash  
       [note] type 'exit' to logout docker shell 
    
## Run openpilot outside the docker
From [the discussion](https://github.com/commaai/openpilot/issues/204)

I needed to change the makefile for OP version 0.3.8.2, but didn't face this issue in latest version (0.4.2). 
So, in this version you don't need to change anything in makefile. To run OP outside of docker you have to 
follow the the following procedure -
1. First install all the libraries in your PC.
    * apt-get update && apt-get install -y build-essential clang vim screen wget bzip2 git libglib2.0-0 python-pip capnproto libcapnp-dev libzmq5-dev libffi-dev libusb-1.0-0
    * pip2 install numpy==1.11.2 scipy==0.18.1 matplotlib
    * pip2 install -r requirements_openpilot.txt  
      [note] this will install the libraries mentioned in the requirements text file
2. Create a separate .sh file and include the following lines in it,
    * export PYTHONPATH="$PWD":$PYTHONPATH
    * rm selfdrive/can/libdbc.so  
      [note] # if you don't need to modify the .cc files or the makefile inside 'can' folder, then you can ignore this line
    * /bin/sh -c 'cd selfdrive/test/tests/plant && OPTEST=1 ./test_longitudinal.py'
3. Then execute this new .sh file instead of run_dockers_test.sh file. You may need to use sudo to execute this new sh file.
This is the basic procedure. You may see some module or path issues whole running the python file, but that are solvable.

Above discussion is similiar to the one from docker's Dockerfile.openpilot:   

    ------------------------------------------
    FROM ubuntu:16.04
    ENV PYTHONUNBUFFERED 1

    RUN apt-get update && apt-get install -y build-essential clang vim screen wget bzip2 git libglib2.0-
    0 python-pip capnproto libcapnp-dev libzmq5-dev libffi-dev libusb-1.0-0
    
    RUN pip install numpy==1.11.2 scipy==0.18.1 matplotlib==2.1.2

    COPY requirements_openpilot.txt /tmp/
    RUN pip install -r /tmp/requirements_openpilot.txt

    ENV PYTHONPATH /tmp/openpilot:$PYTHONPATH

    COPY ./common /tmp/openpilot/common
    COPY ./cereal /tmp/openpilot/cereal
    COPY ./opendbc /tmp/openpilot/opendbc
    COPY ./selfdrive /tmp/openpilot/selfdrive
    COPY ./phonelibs /tmp/openpilot/phonelibs
    COPY ./pyextra /tmp/openpilot/pyextra

    RUN mkdir -p /tmp/openpilot/selfdrive/test/out
    RUN make -C /tmp/openpilot/selfdrive/controls/lib/longitudinal_mpc clean
    RUN make -C /tmp/openpilot/selfdrive/controls/lib/lateral_mpc clean
    ------------------------------------------
    
