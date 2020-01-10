# ansible-dns

Ansible Playbook to install, initialize, and deploy a high availability (HA) DNS solution utilizing Ansible, GlusterFS, Docker swarm, and [Pi-hole](https://pi-hole.net).

# Overview

Pi-hole blocks ads on your LAN but becomes a single point of failure that can bring your internet to a screeching halt if it fails for whatever reason. Having redundant Pi-hole servers will mitigate this concern.

Docker swarm is used to replicate the DNS servers.

GlusterFS is used to keep configuration in sync between servers.

## Summary

### Hosts
The swarm consists of 3 SBC hosts running [DietPi](https://dietpi.com)
- NanoPi NEO2 1G
- NanoPi NEO2 512M
- RaspberryPi 2 Model B

I partition the SD cards to have a 4GB xfs partition which is mounted and used for GlusterFS.

### Ansible
Make sure the control node can connect via ssh to the managed nodes. Utilize ```ssh-agent```

Install Python on nodes
```sudo apt install python3```

### GlusterFS
A minimum of 3 hosts are needed in order to avoid a split brain

### Docker swarm
[Portainer](https://www.portainer.io) is deployed for swarm management

Inspect and edit the ```\roles\docker]tasks\subtasks\files\pihole-stack.yml```