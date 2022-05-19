# Deploy OpenShift

### 1. Create VM for the node

Instance Name | vCPU | Memory | Disk | OS |
--------------|------|--------|------|----|
 Helper/svc | 4 Core | 6GB | 150GB | RHEL8 |
 Bootstrap | 6 Core | 10 GB | 50GB | RHCOS |
 Master00 | 4 Core | 8 GB | 50 GB | RHCOS |
 Master01 | 4 Core | 8 GB | 50 GB | RHCOS |
 Master02 | 4 Core | 8 GB | 50 GB | RHCOS |
 Worker00 | 4 Core | 8 GB | 50 GB | RHCOS |
 Worker01 | 4 Core | 8 GB | 50 GB | RHCOS |

Bootstrap, Master, and Worker use the iso image downloaded during file-setup.md for first boot process.
NIC for Bootstrap, Master, and Worker node will be attach to internal network.

### 2. Power on the ocp-bootstrap host and ocp-cp-# hosts and select 'Tab' to enter boot configuration. Enter the following configuration

Turn on Bootstrap node and run these commands:
> Make sure the hostname is not set to 'localhost' and IP is assgin to each node correctly.

```bash
# Bootstrap Node - ocp-bootstrap
coreos.inst.install_dev=vda coreos.inst.image_url=http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz coreos.inst.insecure=yes coreos.inst.ignition_url=http://192.168.22.1:8080/ocp4/bootstrap.ign

# Or if you waited for it boot, use the following command then just reboot after it finishes and make sure you remove the attached .iso
sudo coreos-installer install /dev/vda -u http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz -I http://192.168.22.1:8080/ocp4/bootstrap.ign --insecure --insecure-ignition
```

Turn on master node and run these commands:

```bash
# Each of the Control Plane Nodes - ocp-cp-\#
coreos.inst.install_dev=vda coreos.inst.image_url=http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz coreos.inst.insecure=yes coreos.inst.ignition_url=http://192.168.22.1:8080/ocp4/master.ign

# Or if you waited for it boot, use the following command then just reboot after it finishes and make sure you remove the attached .iso
sudo coreos-installer install /dev/vda -u http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz -I http://192.168.22.1:8080/ocp4/master.ign --insecure --insecure-ignition
```

Turn on worker node and run these commands:

```bash
# Each of the Control Plane Nodes - ocp-cp-\#
coreos.inst.install_dev=vda coreos.inst.image_url=http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz coreos.inst.insecure=yes coreos.inst.ignition_url=http://192.168.22.1:8080/ocp4/worker.ign

# Or if you waited for it boot, use the following command then just reboot after it finishes and make sure you remove the attached .iso
sudo coreos-installer install /dev/vda -u http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz -I http://192.168.22.1:8080/ocp4/worker.ign --insecure --insecure-ignition
```

### 3. Monitor the Bootstrap Process

You can monitor the bootstrap process from the ocp-svc host at different log levels (debug, error, info).

```bash
./openshift-install --dir ~/ocp-install wait-for bootstrap-complete --log-level=debug
```

