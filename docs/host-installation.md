# Generate and host install files

prerequisites:

- [ ] RH id for pull secret
- [ ] SSH public key

### 1. Download and extract the openshift install

download okd/openshift installer from release page and extract it

- [OKD Releases Page](https://github.com/openshift/okd/releases)
- [OCP Releases Page](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/)

```bash
mkdir ocp-install
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.14/openshift-install-linux.tar.gz
tar -xvf openshift-install-linux.tar.gz -C ocp-install
```

### 2. Check the version of the installer and downlaod coreos

```bash
./openshift-install coreos print-stream-json
```

the output will show json file like below:

```json
          "metal": {
            "release": "48.84.202109241901-0",
            "formats": {
              "4k.raw.gz": {
                "disk": {
                  "location": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-metal4k.x86_64.raw.gz",
                  "signature": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-metal4k.x86_64.raw.gz.sig",
                  "sha256": "9cf43a94eded275b2fb54bbb164ce63b18d7e696ca6c15f8cea4bb09941a0e57",
                  "uncompressed-sha256": "475d1123a2da31882b445c51ec5d5896c84de417feca5fe0ac1f93d8ea4e2c84"
                }
              },
              "iso": {
                "disk": {
                  "location": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live.x86_64.iso",
                  "signature": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live.x86_64.iso.sig",
                  "sha256": "dc0c7af647137c9b16210c977896dc756b72b5e568774daf199964ca020935ae"
                }
              },
              "pxe": {
                "kernel": {
                  "location": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live-kernel-x86_64",
                  "signature": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live-kernel-x86_64.sig",
                  "sha256": "d13269e6c60119397210418781b7057673c4018692d28a868e248a0b550ea247"
                },
                "initramfs": {
                  "location": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live-initramfs.x86_64.img",
                  "signature": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live-initramfs.x86_64.img.sig",
                  "sha256": "5b6f1b728a5e656ef128a54f9534f7d3d63953ef46e726a808aacf1ce9ad2742"
                },
                "rootfs": {
                  "location": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live-rootfs.x86_64.img",
                  "signature": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-live-rootfs.x86_64.img.sig",
                  "sha256": "bd005071f295762a613ff42c26ce453be7c19d35594b68c94fab6797b4d834d4"
                }
              },
              "raw.gz": {
                "disk": {
                  "location": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz",
                  "signature": "https://rhcos-redirector.apps.art.xq1c.p1.openshiftapps.com/art/storage/releases/rhcos-4.8/48.84.202109241901-0/x86_64/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz.sig",
                  "sha256": "6e67556bebee87b6d85d776a8b728ef727fb2e55c77a183f7979e8b89211b4db",
                  "uncompressed-sha256": "1b104a26b338f6aa4ba16a0bb3887e7bdc05c8735902785578208fb793fc504d"
                }
              }
            }
          }
```

We need to download 2 file from here iso and raw.gz, iso file to be used as initial disk and raw.gz file to be used as source during cluster deployment.
Mount iso file on vm node and raw.gz file will be hosted on Apache Web Server.

### 3. Copy/create the install-config.yaml to the install directory

example of install-config.yaml

```bash
./openshift-install create install-config.yaml
```

standard install-config.yaml file will look like this

```yaml
apiVersion: v1
baseDomain: ocp.lan
compute:
  - hyperthreading: Enabled
    name: worker
    replicas: 0 
    # Replicas Must be set to 0 for User Provisioned Installation as worker nodes will be manually deployed.
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: your.cluster.name.here # Cluster name
networking:
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
    - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{"auths": ...}'
sshKey: "ssh-ed25519 AAAA..."
```

> Note: In here you must provide some spesific detail regarding your cluster, such as cluster name, your pull secret, and sshkey.

### 4. Generate Kubernetes manifest and ignition files

```bash
./openshift-install create manifests --dir ~/ocp-install
```

> Note: A warning is shown about making the control plane nodes schedulable. It is up to you if you want to run workloads on the Control Plane nodes. If you dont want to you can disable this with: `sed -i 's/mastersSchedulable: true/mastersSchedulable: false/' ~/ocp-install/manifests/cluster-scheduler-02-config.yml`. Make any other custom changes you like to the core Kubernetes manifest files.

Generate the Ignition config and Kubernetes auth files

```bash
./openshift-install create ignition-configs --dir ~/ocp-install/
```

### 5. Create a hosting directory to serve the configuration files for the OpenShift booting process and copy all the file on installation directory

```bash
mkdir /var/www/html/ocp4
cp -R ~/ocp-install/* /var/www/html/ocp4
```

Move the Core OS raw.gz file to the web server directory (later you need to type this path multiple times so it is a good idea to shorten the name)

```bash
mv ~/rhcos-X.X.X-x86_64-metal.x86_64.raw.gz /var/www/html/ocp4/rhcos
```

Change ownership and permissions of the web server directory

```bash
chcon -R -t httpd_sys_content_t /var/www/html/ocp4/
chown -R apache: /var/www/html/ocp4/
chmod 755 /var/www/html/ocp4/
```

Confirm you can see all files added to the /var/www/html/ocp4/ dir through Apache

```bash
curl localhost:8080/ocp4/
```
