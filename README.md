pi-cluster setup

setup new server:

Boot the pi from a previously installed media and set the bootloader order to reference the media type you're
using as the boot drive. This is done via rpi-eeprom-config:
   e.g. pi5 0xf146 (see https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#BOOT_ORDER)
   # sudo -E rpi-eeprom-config --edit
     BOOT_ORDER=0xf146

Make a note of the MAC address and configure your router's DHCP with a MAC<->IP lookup

Use rpi-imager to burn ubuntu 23.10 (or latest) onto your pi boot media: sd-card, usb drive or nvme
 - set a user with a password (ubuntu/password) - it doesn't work otherwise, can't ssh even though ssh key is apparently copied
 - set the additional options to only allow ssh access and paste your id_rsa.pub

(ansible) install updates (this also fixes the fan 100% issue)
$ sudo apt update
$ sudo apt upgrade
$ reboot now

(ansible) apt install sysbench



- boot system, ssh as user ubuntu and do mandatory password change
- copy ssh id to remote
   ssh-copy-id -i ~/.ssh/id_rsa ubuntu@new.system.ip.address
- check ssh access
   ssh ubuntu@new.system.ip.address
- add server to ssh config: ~/.ssh/config
Host rp5
    User ubuntu
    HostName new.system.ip.address
    IdentityFile ~/.ssh/id_rsa
    ForwardAgent no


ansible - use the playbooks to configure the PIs ready for kubernetes installation
        - run the following: 
            ansible-playbook -i lab site.yml

run this boot string update on all the PIs (TBD - add to ansible)
sudo sed -i \
'$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' \
/boot/firmware/cmdline.txt

reboot the PIs

following the k3s instructions from here:
https://anthonynsimon.com/blog/kubernetes-cluster-raspberry-pi/
... run this on the server node rp4 (kcontrolp)
curl -sfL https://get.k3s.io | sh -

then get the cluster token from here:
sudo cat /var/lib/rancher/k3s/server/node-token

On the remaining 3 worker nodes install k3s, pointing at the server node IP with the cluster token:
curl -sfL https://get.k3s.io | K3S_URL=https://$YOUR_SERVER_NODE_IP:6443 K3S_TOKEN=$YOUR_CLUSTER_TOKEN sh -


Bootstrap flux:
flux bootstrap github \
  --owner=thatmikewilliams \
  --repository=pi-cluster \
  --path=deployment/pi-cluster \
  --personal
