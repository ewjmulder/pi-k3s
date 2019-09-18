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
* SSH into the PI's and get them ready for k3s, and for each of them:
  * `sudo raspi-config`
    * 1 Change User Password
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
  * Enable container features: `sudo nano /boot/cmdline.txt` and append `cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory` to end of line
  * reboot: `sudo reboot -h now`
  * Copy your local SSH key onto the PI for no-password ssh login: `ssh-copy-id pi@x.x.x.x`
* Install k3s on the master:
  * ssh into master and execute installation script: `curl -sfL https://get.k3s.io | sh -`
  
  

Installing k3s on master: seems success, but system completely hangs afterwards
```
pi@pi-k3s-master:~ $ curl -sfL https://get.k3s.io | sh -
[INFO]  Finding latest release
[INFO]  Using v0.8.1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v0.8.1/sha256sum-arm.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v0.8.1/k3s-armhf
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
pi@pi-k3s-master:~ $ 
```
