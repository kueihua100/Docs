# My notes to controlsd 
## Where to start

### Source
From:  
  managed_processes[] = {  
   ...  
   "controlsd": "selfdrive.controls.controlsd",  
   ...  
  }  
  selfdrive.controls.controlsd will be import at manager.py::prepare_managed_process()  
  ===> controlsd.py can be found under openpilot/selfdrive/controls/  
    [note] defaultd is used to send can messages when controlsd is off to make car test easier

### special libraries:
    a) pyzmq: This is the Python bindings for ZMQ.
    b) pycapnp: This is a python wrapping of the C++ implementation of the Cap’n Proto library.
       Cap’n Proto is an insanely fast data interchange format and capability-based RPC system.
       http://capnproto.github.io/pycapnp/
    
## Code flow
#### controlsd_thread()
    controlsd.py::controlsd_thread()
    -> create/register zmq socket for comunicating msgs.
    -> get_car()
