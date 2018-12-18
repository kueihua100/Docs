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

## How openpilo steer to a position?
    a) The CAN packet doesn’t specify a position you want the wheel to turn to, instead it defines 
       how much torque to put on the wheel. In order to command the wheel to go to a position, 
       we need to use some controls to close the loop. The steering angle is available on the CAN bus as well.
    b) A desired angle, current angle, and torque command. Since torques are small, we only use a PI loop.

#### latcontrol.py
    class will call into PIControl: openpilot/selfdrive/controls/lib/pid.py
![pi_ctrl](/ai/res/pi_ctrl.gif)

## References
a) https://medium.com/@comma_ai/how-does-openpilot-work-c7076d4407b3  
b) https://medium.com/@comma_ai/openpilot-port-guide-for-toyota-models-e5467f4b5fe6   
c) https://medium.com/@energee/add-support-for-your-car-to-comma-ai-openpilot-3d2da8c12647  
