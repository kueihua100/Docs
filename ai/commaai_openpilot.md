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
    
## Run openpilot inside the docker (at Ubuntu Linux)
```bash
# Requires working docker
./run_docker_tests.sh
```
    [note 1] From run_docker_test.sh:
        #!/bin/bash
        set -e
        docker build -t tmppilot -f Dockerfile.openpilot .
        docker run --rm \
          -v "$(pwd)"/selfdrive/test/tests/plant/out:/tmp/openpilot/selfdrive/test/tests/plant/out \
          tmppilot /bin/sh -c 'cd /tmp/openpilot/selfdrive/test/tests/plant && OPTEST=1 ./test_longitudinal.py'

    [note 2] 
        docker build -t tmppilot -f bashDock.openpilot .
        --------------------------
        Above cmd will build a image taged "tmppilot:latest" at the end.
        Using "docker images" to check the built image.
        --------------------------
        
        Below is the content of bashDock.openpilot:
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
        ------------------------------------------
        
    [note 3]
        docker run -it tmppilot:latest bash
        -----------------------
        Above cmd can enter bash shell of tmppilot image.
        Then do below cmds will do the same things at [note 1]
        # rm -rf /tmp/openpilot/selfdrive/test/tests/plant/out
        # make -C /tmp/openpilot/selfdrive/controls/lib/longitudinal_mpc clean
        # make -C /tmp/openpilot/selfdrive/controls/lib/longitudinal_mpc
        # make -C /tmp/openpilot/selfdrive/controls/lib/lateral_mpc clean
        # make -C /tmp/openpilot/selfdrive/controls/lib/lateral_mpc
        # /bin/sh -c 'cd /tmp/openpilot/selfdrive/test/tests/plant && OPTEST=1 ./test_longitudinal.py'

## Run openpilot inside the docker (at Win 10)
1.  Install Win10 Powershell
2. Install Docker for Win 10
3. Build tmppilot docker image with sharing a folder:  
    a) git clone openpilot source to d:\test\openloit (example)  
    b) open Powershell and enter d:\test\openloit  
    c) Run below cmd to build tmppilot docker image:  
        docker build -t tmppilot -f bashDock.openpilot .  
    d) Show available docker images:  
        docker images  
    e) Delete docker image:  
        docker rmi xxx_image_id  
    f) Delete docker container  
        docker container ls -a <=== list currently containers
        docker container rm xxx_container_id
4. Run tmppilot:  
    a) Enable "Share Drives" from Win docker's [Setting] menu.  
        Here select "D" and click "Apply" button.  
        
    b) Run tmppilot image with sharing d:\test\openloit  
    
        docker run -it --mount type=bind,source=d:\\/test\\/openloit,target=/data tmppilot:latest bash  
        # rm -rf /tmp/openpilot/selfdrive/test/tests/plant/out  
        # make -C /tmp/openpilot/selfdrive/controls/lib/longitudinal_mpc clean  
        # make -C /tmp/openpilot/selfdrive/controls/lib/longitudinal_mpc  
        # make -C /tmp/openpilot/selfdrive/controls/lib/lateral_mpc clean  
        # make -C /tmp/openpilot/selfdrive/controls/lib/lateral_mpc  
        # /bin/sh -c 'cd /tmp/openpilot/selfdrive/test/tests/plant && OPTEST=1 ./test_longitudinal.py'  

        [note] 
            Sometimes win10's docker environment is not stable during building images (download packages).
            You can use "docker export" to export a docker image that you already built and used in win10.
            docker export xxx_container_id > from_linux.tgz
            docker import from_linux.tgz RESPOSITORY:TAG  <== maybe need to setup environment varables taht set at linux.
            docker load -i from_linux.tgz

## Run bash shell from a stopped container
    $ docker container ls -a
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        ab242a13f03d        testpilot:latest    "bash"              5 days ago          Exited (0) 2 seconds
    
    $ docker container start ab242a13f03d
    $ docker container ls -a
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        ab242a13f03d        testpilot:latest    "bash"              5 days ago          Up 1 second
        
    $ docker exec -it ab242a13f03d bash
        root@ab242a13f03d:/
