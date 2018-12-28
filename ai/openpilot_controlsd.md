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
    c) cffi: C Foreign Function Interface for Python. Interact with almost any C code from Python.
       https://pypi.org/project/cffi/
       
## Code flow
#### controlsd_thread()
    controlsd.py::controlsd_thread()
    -> create/register zmq socket for comunicating msgs.
    -> car_helpers.py::get_car()
       a) get car interface from interface.py
       b) get car parameters from interface.py
    -> End if has no car interface
    -> Init Planner()
       a) zmq subscribe to live20
       b) zmq subscribe to model
       c) zmq subscribe to gpsPlannerPlan if env variable GPS_PLANNER_ACTIVE is set
       d) zmq publish to plan
       e) zmq publish to liveLongitudinalMpc
       f) Init PathPlanner()
       g) Init 2 of LongitudinalMpc()
          load c libraries: libmpc*.so
    -> Init LongControl()
    -> Init VehicleModel()
    -> Init LatControl()
    -> Init AlertManager()
    -> Init DriverStatus()
