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
        - run the following: ansible-playbook -i lab site.yml

