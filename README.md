# pi-k3s
Setup and management of a raspberry pi kubernetes cluster

## Hardware
Pico cluster R-Pi 5E: https://www.picocluster.com/products/pico-5-raspberry-pi?variant=42238421772
5 x Samsung EVO 32 GB micro SD cards
Assembled according to instructions. No custom mods (yet).
To be added: Blinkt!
Maybe to be added: custom LCD display with status

## Software
K3S - Lightweight Kubernetes: https://k3s.io/

## Installation steps:
Used these steps as guideline: https://blog.alexellis.io/test-drive-k3s-on-raspberry-pi/
* Assemble hardware
* Flash latest Rasbian Lite to 5 micro SD cards, add empty file `ssh` to the root of the boot partition and insert
* Power up and connect network cable with some DHCP server / router
* Check which IP the pi's got on router management page or use port scanner: https://angryip.org
* SSH into the PI's and to for each of them:
  * `sudo raspi-config`
    * 1 Change User Password (optional)
    * 2 Network Options - N1 Hostname: choose hostname applicable to master/node (pi-k3s-master or pi-k3s-node[1-4])
    * 4 Localisation Options - Change Timezone: set to your own timezone
    * 7 Advanced Options - A3 Memory Split: Set to 16 MB
  * Fix IP in config (or with DHCP fixed IP by MAC address)
    * `sudo nano /etc/dhcpcd.conf`
    * Append these lines at the end
    ```
    interface eth0

    static ip_address=x.x.x.x/24 (e.g. 10.42.0.101/24 or 192.168.0.151)
    static routers=x.x.x.x (e.g. 10.42.0.1 or 192.168.0.1)
    static domain_name_servers=x.x.x.x (e.g. 10.42.0.1 or 192.168.0.1)
    ```
    * reboot
  * Copy your local SSH key onto the PI for no-password ssh login: `ssh-copy-id pi@x.x.x.x`
