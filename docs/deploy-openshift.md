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

### 4. Remove Bootstrap Node

Remove all references to the ocp-bootstrap host from the /etc/haproxy/haproxy.cfg file

```bash
# Two entries
vim /etc/haproxy/haproxy.cfg
# Restart HAProxy - If you are still watching HAProxy stats console you will see that the ocp-boostrap host has been removed from the backends.
systemctl reload haproxy
```

The ocp-bootstrap host can now be safely shutdown and deleted from the VMware ESXi Console, the host is no longer required.

### 5. Wait for installation to complete

Collect the OpenShift Console address and kubeadmin credentials from the output of the install-complete event

```bash
./openshift-install --dir ~/ocp-install wait-for install-complete
```

### 6. Join Worker Nodes

Setup 'oc' and 'kubectl' clients on the ocp-svc machine

```bash
export KUBECONFIG=~/ocp-install/auth/kubeconfig
# Test auth by viewing cluster nodes
oc get nodes
```

View and approve pending CSRs
> Note: Once you approve the first set of CSRs additional 'kubelet-serving' CSRs will be created. These must be approved too. If you do not see pending requests wait until you do

```bash
# View CSRs
oc get csr
# Approve all pending CSRs
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
# Wait for kubelet-serving CSRs and approve them too with the same command
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```

Watch and wait for the Worker Nodes to join the cluster and enter a 'Ready' status

```bash
watch -n5 oc get nodes
```
