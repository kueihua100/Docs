# Note 2 to Commaai's openpilot
## CAN Bus (*a)
![CAN_BUS](/ai/res/can_bus.png)
CAN is a simple protocol. It’s a bus, where any device can send a message to all others. 
A message contains an identifier, 11-bits long in standard CAN, 29-bits long in extended, 
and a message, which can be up to 8 bytes long.

#### parser.py:
    Class to parse CAN messages: openpilot/selfdrive/can/xxxx

#### dbc.py:
    implementation of DBC: openpilot/common/dbc.py

## Get a starting: (*a)
    a) openpilot/selfdrive/manager.py: is responsible for starting and stopping processes.
       It has two states, car stopped and car started, and runs different processes depending on 
       what state it’s in. See service_list.yaml for a list.

    b) boardd: communicates with the car (process that communicates with the outside world)
    c) sensord: publishes the IMU and GPS (process that communicates with the outside world)
    d) visiond: talks to the cameras, runs the model, saves the videos (process that communicates with the outside world)
    e) plannerd: decides where to drive the car
    f) controlsd: actually drives the car. It is started when the car starts. 
                  The controlsd_thread function would probably be a good place to start 
                  reading the code, it’s the main 100hz control loop.
    g) radard: processes the radar data
    h) loggerd: Logger and uploader of car data.
    i) uploader: communicates through file system with loggerd
    j) logmessaged: central logging service, can log to cloud
    k) logcatd: fetches logcat info from android
    l) proclogd: fetches process information
    m) tombstoned: reports native crashes
    
    [note for versiond]: From below 2 links:    
    https://github.com/commaai/openpilot/issues/63#event-978371221  
    https://github.com/commaai/openpilot/issues/96  
      a) Apart from visiond, openpilot is straightforward to use in other hardware. 
         You would only need to write your own manager.py that starts a different 
         visiond daemon and a different boardd daemon.
      b) The visiond expects that specific camera, and cannot get good performance with a different one.

#### manager.py:
    a) update NEOS
    b) update/install apks
    c) build cereal code
    d) prepare processes in managed_processes[]
    e) run manager_thread(): run process in persistent_processes[] and car_started_processes[]

## Determine CAN Fingerprint (*b/c)
    a) CAN fingerprint is CAN msgs from Powertrain CAN bus. 
    b) The assumption is that every car model can be uniquely identified by the set of 
       CAN messages on the Powertrain CAN Bus.
    c) Using openpilot/selfdrive/debug/get_fingerprint.py:
       This script listens to all the CAN messages published on the Powertrain CAN Bus and prints out a list.
       Like below:
       number of messages: 65
       fingerprint 36: 8, 37: 8, 170: 8, 180: 8, 186: 4, 426: 6, 452: 8, 464: 8, 466: 8, 467: 8, 547: 8, 
       548: 8, 552: 4, 608: 8, 610: 5, 705: 8, 800: 8, 849: 4, 896: 8, 897: 8, 900: 6, 902: 6, 905: 8, 
       ...
    d) Failing to collect all of the messages will result in openpilot unreliably detecting the fingerprint 
       of your car when you turn on your vehicle.

## Car specific code (*b/c)
Except for the fingerprint and the dbc file, car-specific code is contained in the path:  
openpilot/selfdrive/car

#### carstate.py: 
    this class reads CAN messages from the Powertrain CAN Bus, parses them using the dbc file 
    and converts them in a common car state format: see CarState structure in openpilot/cereal/car.capnp    
    [note]: car.capnp is a car abstraction layer taht is defined to be able to talk to many different cars
    
    vEgo: 
      this is the vehicle speed in m/s. While driving the car, make sure the speedometer speed roughly 
      matches the printed value.
      
    gearShifter: 
      with the car turned on, move the gear shifter and make sure the printed gear enumeration matches
      the actual gear lever position (park, neutral, drive etc...).
      
    leftBlinkers, rightBlinker: 
      booleans that indicate the blinker’s state.
      
    steeringAngle: 
      from a neutral/straight position, rotate the steering wheel 360 degrees to the left. 
      You should read 360.
      
    gas: 
      you should read 0 when the gas pedal is released, while you should read 1 when the gas pedal 
      if fully depressed. Note that you can test this with the car in ON ignition mode to avoid revving up 
      the engine when pressing the gas pedal.
      
    gasPressed: 
      boolean that determines if the gas pedal is pressed or released.
      
    brakePressed: 
      boolean that determines if the brake pedal is pressed or released.
      
    steeringTorque: 
      numerical value that represents how much torque the driver is putting on the steering wheel.
      This value is noisy as it’s measured by a torque transducer. Make sure the value is roughly 
      linearly correlated to the effort that you put in turning the steering wheel. 
      It should be positive when trying to steer left.
      
    steeringPressed: 
      a boolean that indicates if the driver is putting any torque on the steering wheel.
      
    doorOpen and seatbeltUnbuckled: 
      booleans that indicates if any of the doors are open and if the driver’s seat belt 
      is unlatched, respectively.
      
    genericToggle: 
      this boolean is used to facilitate testing for the later steps. By default, it’s arbitrarily 
      linked with the auto high-beam toggle state. You have to have the headlights on for this to work.

#### carcontroller.py: 
    class that receives control data from the controlsd thread (such as abstracted actuators commands, 
    see CarController in openpilot/cereal/car.capnp) and packs them into CAN messages using the dbc file. 
    CAN messages are sent both on Powertrain CAN Bus and Radar CAN Bus.
       
#### interface.py: 
    class that contains car specific physical parameters, tuning parameters and methods 
    to execute car state and controller updates.
       
#### radar_interface.py: 
    very similar to carstate.py, this class reads CAN messages from the Radar CAN Bus, parses them 
    and converts them in a common radar state format: see RadarState structure in openpilot/cereal/car.capnp. 
    It’s very unlikely that your unsupported car model will require changes to this file, so you can ignore it.
    [note] ignore for adding an un-support car.

#### values.py and <car_maker>can.py: 
    these 2 files are a collection of a few functions and classes mainly used by carcontroller.py. 
    In particular,values.py includes a dictionary of static messages that carcontroller needs to send to 
    properly simulate the disconnected ECUs (FRC and/or DSU). 

## How openpilot steer to a position? (*a)
    a) The CAN packet doesn’t specify a position you want the wheel to turn to, instead it defines 
       how much torque to put on the wheel. In order to command the wheel to go to a position, 
       we need to use some controls to close the loop. The steering angle is available on the CAN bus as well.
    b) A desired angle, current angle, and torque command. Since torques are small, we only use a PI loop.
    openpilot/selfdrive/controls/lib/latcontrol.py will call into PIControl: 
    openpilot/selfdrive/controls/lib/pid.py
    
![pi_ctrl](/ai/res/pi_ctrl.gif)

#### A Vision System (lateral)
visiond runs the driving neural network. This part of openpilot is closed source.
But its API is open as below:

    struct ModelData {
        frameId @0 :UInt32;

        path @1 :PathData;
        leftLane @2 :PathData;
        rightLane @3 :PathData;
        lead @4 :LeadData;
        ...
    }
    
    a) openpilot/selfdrive/controls/lib/pathplanner.py: versiond outputs a best guess at the path, 
       where it thinks the left and right lanes are, and where it thinks the lead car is. 
       It merges these paths together into the path the car should follow.

    b) openpilot/selfdrive/controls/lib/latcontrol.py: a point is picked along this path as the target point 
       for the car to aim for. Then it follows the arc to that point.

#### Adaptive Cruise Control (longitudinal)
    Openpilot separates the gas/brakes from the steering contorl. 
    Longitudinal control is still done in an old school way, without a neural network (versiond).

    b) openpilot/selfdrive/controls/radard.py: is used to parse the messages from the car radar. 
       It does a rudimentary fusion with the vision system, and outputs the positions of up to 
       two “lead cars” at 20hz.
       
## References
a) https://medium.com/@comma_ai/how-does-openpilot-work-c7076d4407b3  
b) https://medium.com/@comma_ai/openpilot-port-guide-for-toyota-models-e5467f4b5fe6   
c) https://medium.com/@energee/add-support-for-your-car-to-comma-ai-openpilot-3d2da8c12647  
