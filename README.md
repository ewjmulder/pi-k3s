# pi-k3s
Setup and management of a raspberry pi kubernetes cluster

## Hardware
Pico cluster R-Pi 5E: https://www.picocluster.com/products/pico-5-raspberry-pi?variant=42238421772
5 x Samsung EVO 32 GB micro SD cards
Assembled according to instructions.
Mod: added Blinkt! (https://shop.pimoroni.com/products/blinkt)
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
* Install k3s on the master: (Used this page as guideline: https://blog.alexellis.io/test-drive-k3s-on-raspberry-pi/)
  * ssh into master and execute installation script: `curl -sfL https://get.k3s.io | sh -`
  * Check that is started successfully: `sudo systemctl status k3s`
  * Grab the join key of the master: `sudo cat /var/lib/rancher/k3s/server/node-token`
* Install k3s on each node:
  * ssh into each node and execute:
  * `export K3S_URL="https://x.x.x.x:6443"`
  * `export K3S_TOKEN="Kxxx::node:xxx"` (join key of master)
  * Run the installation script: `curl -sfL https://get.k3s.io | sh -` (it will know from the env vars that it's supposed to be a node for that server)
* Now you can see the nodes when executing on the master: `kubectl get node -o wide`
  * TODO: This actually requires sudo, is that ok? Or maybe have it like that on the master itself, but not on a machine connecting to the master 
* When done, it all works and you can play around with it on the master, but we want to run kubectl from our local machine and talk to the cluster over the network. Therefore we need the KUBECONFIG file. TODO: how to connect this...


* Optional mod: Install 'Blinkt!': (https://learn.pimoroni.com/tutorial/sandyj/getting-started-with-blinkt)
  * `curl https://get.pimoroni.com/blinkt | bash` (2 x 'y' during installation)
  * `sudo reboot -h now` to apply changes (actually seems not to be needed)
  
  Issue: If you give internet to the cluster by sharing your wifi over ethernet on a Linux system (Ubuntu / Mint / others?) you will get an IP range slash: both "Share with other computers" DHCP and k3s use the 10.42.0.xxx range. To solve this you could get k3s to use another range, but that can be tricky to get working (in my experience). Easier fix is to let the DHCP of the connection sharing use another range, which you can easily configure in the GUI.

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
