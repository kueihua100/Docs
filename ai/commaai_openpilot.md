# Notes to Commaai's openpilot

## Before starting
    1. openpilot, autoware, apollo... all need the docker environment.
    2. But you can do some handmade procedures to run the code outside docker.

## Run openpilot outside the docker
From [the discussion](https://github.com/commaai/openpilot/issues/204)

I needed to change the makefile for OP version 0.3.8.2, but didn't face this issue in latest version (0.4.2). 
So, in this version you don't need to change anything in makefile. To run OP outside of docker you have to 
follow the the following procedure -
1. First install all the libraries in your PC.
    * apt-get update && apt-get install -y build-essential clang vim screen wget bzip2 git libglib2.0-0 python-pip capnproto libcapnp-dev libzmq5-dev libffi-dev libusb-1.0-0
    * pip2 install numpy==1.11.2 scipy==0.18.1 matplotlib
    * pip2 install -r requirements_openpilot.txt # this will install the libraries mentioned in the requirements text file
2. Create a separate .sh file and include the following lines in it,
    * export PYTHONPATH="$PWD":$PYTHONPATH
    * rm selfdrive/can/libdbc.so # if you don't need to modify the .cc files or the makefile inside 'can' folder, then you can ignore this line
    * /bin/sh -c 'cd selfdrive/test/tests/plant && OPTEST=1 ./test_longitudinal.py'
3. Then execute this new .sh file instead of run_dockers_test.sh file. You may need to use sudo to execute this new sh file.
This is the basic procedure. You may see some module or path issues whole running the python file, but that are solvable.
