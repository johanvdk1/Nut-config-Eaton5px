Aim: all services are restored without intervention. Normally servers that were shutdown on a UPS will not experience power loss if the ups is not fully drained. So they will not use the BIOS "what to do after power loss" setting, and remain in standby. BIOS setting should be "restart" and not "do nothing" or "previous state"
Here the Proxmox server (also UPSD host) cuts the power to the Truenas server. Which then in turn wakes up the proxmox server when power is restored.

Configuration for Eaton5px
Connections: 
 - outlet unswitched: critical equipment and Proxmox pve. Keep internet and landline running.
 - outlet group 1: Truenas. Own nut client, shuts down 300 secs after ups reports ONBATT. Allows files to be saved by users on laptops.
 - outlet group 2: Shed group. Switch off to prolong battery runtime, just wait a few minutes in case of a short power interruption.
   roxmox will control (switch off) group 1 and go standby after 600 seconds. 

Wake-up sequence
  Power returns before 300 seconds:
  - cancel shutdown on Fileserver
  - cancel shutdown on Proxmox
  - restore power to Shed group

  Power returns after 300 secs but before 600 secs (servers are not experiencing power loss so BIOS boot on power failure will not be executed).
  - cancel shutdown on Proxmox
  - restore power to Shed group
  - Proxmox wakes Fileserver (wake on lan)
  - Container starts on Truenas Fileserver. This container only executes a number of wake on lan commands
  - File server wakes up backup server (wake on lan)
  

  Power returns after 600 sec, but before UPS Low Batt
  - File server restores itself from power loss through BIOS setting.
  - Container starts on Truenas Fileserver. This container only executes a number of wake on lan commands.
  - File server container wakes up Proxmox and File server backup 

  Power loss after "empty" battery (UPS kills outlets when 5% battery remaining, as set in ups)
  Proxmox and File server and Backup server restart as specified in BIOS (behaviour after power loss)
    
