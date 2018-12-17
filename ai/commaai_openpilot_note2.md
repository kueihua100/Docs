# Note 2 to Commaai's openpilot
#### Determine CAN Fingerprint 
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
