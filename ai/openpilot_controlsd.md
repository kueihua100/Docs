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
    
## Code flow
