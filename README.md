pi-cluster setup

setup new server:
- add MAC<->IP mapping in local dhcp router
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
... run this on the server node rp4
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
