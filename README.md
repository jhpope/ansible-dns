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

### Ansible
Make sure the control node can connect via ssh to the managed nodes. Utilize ```ssh-agent```

Install Python on nodes
```sudo apt install python3```

### GlusterFS
GlusterFS is a networked filesystem used in this instance to create a replicated volume. This solves the common issue of keeping the configuration and white/black lists in sync when running multiple Pi-hole instances. An NFS or other network share could be used but this would create a single point of failure.

I manually create a 4GB xfs partition on the SD cards with [gparted](https://gparted.org). The partition is mounted and used for the GlusterFS brick. The partitions could probably be configured via Ansible but since I'm using the SD card which also houses the OS, it can't be modified while active.

A minimum of 3 hosts are used in order to avoid a [split brain](https://gluster.readthedocs.io/en/latest/Administrator%20Guide/Split%20brain%20and%20ways%20to%20deal%20with%20it/)

### Docker swarm
Docker swarm's routing mesh is being circumvented via MACVLAN routing. This allows clients to communicate directly with the Pi-hole instances resulting in proper client metrics. You will need to configure the `ip-range` in ```\roles\docker\tasks\subtasks\macvlan_config.yml```. Running the playbook as is will work but I've found Docker's IPAM does not seem to distribute IPs to replicated containers correctly. The network config is deployed per node and defines the IP address the service using the network will obtain. Use the playbook as an example, you will probably need to create the network configs manually.

[Portainer](https://www.portainer.io) is deployed for swarm management. Here we can use the routing mesh. Access Portainer on port ```9000``` of any node in the swarm. You can use Portainer to configure a MACVLAN network or manually via the Docker CLI

Inspect and configure the Pi-hole stack file, ```\roles\docker\tasks\subtasks\files\pihole-stack.yml```. For example, the ```WEBPASSWORD``` isn't set, you should obviously set this.
> TODO: Use ansible-vault or docker secrets
