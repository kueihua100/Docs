# Note 2 to Commaai's openpilot
#### Determine CAN Fingerprint 
    a) CAN fingerprint is CAN msgs from Powertrain CAN bus. 
    b) The assumption is that every car model can be uniquely identified by the set of 
       CAN messages on the Powertrain CAN Bus.
    c) Using openpilot/selfdrive/debug/get_fingerprint.py:
       This script listens to all the CAN messages published on the Powertrain CAN Bus and prints out a list.
