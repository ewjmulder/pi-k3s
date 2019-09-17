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
* Flash latest Rasbian Lite to 5 micro SD cards and insert
* Power up and connect network cable with some DHCP server / router
* Check which IP the pi's got on router management page or use port scanner: https://angryip.org
* SSH into the PI's and to for each of them:
  * Fix IP's manually or with DHCP fixed IP by MAC address

