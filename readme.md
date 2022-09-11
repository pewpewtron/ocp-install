# OpenShift Installation on bare metal

This repo provides guide and template for network config to install OpenShift on bare metal.

![network-diagram](docs//images/netdiag.png)

Deployment this cluster requires 2 network one is use for internal ocp cluster and the other is used for external connection. both connection will be connected to Helper/SVC node. where it function as networking service for the cluster. also helper contain NFS and web server where the installation file will be hosted.

prerequisites

    * OpenShift 4.x
    * RHCOS 8
    * NetworkManager
    * DNS Server
    * DHCP Server
    * HAProxy

## How Installation works
### Ignition file

## Step for Installation

[SVC/Helper setups](docs/svc-setups.md)

[Host Installation files](docs/host-installation.md)

[Deploy Openshift](docs/deploy-openshift.md)