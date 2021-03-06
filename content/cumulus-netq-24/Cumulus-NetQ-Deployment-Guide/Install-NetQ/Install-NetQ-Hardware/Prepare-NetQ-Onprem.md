---
title: Prepare for NetQ On-premises Installation
author: Cumulus Networks
weight: 70
aliases:
 - /display/NETQ/Install+NetQ
 - /pages/viewpage.action?pageId=12320951
pageID: 12320951
toc: 5
---
This topic describes the preparation steps needed before installing the NetQ components *on your premises*.  Refer to {{<link title="Prepare for NetQ Cloud Installation">}} for preparations for cloud deployments.

There are three key steps in the preparation for on-premises installation:

1. Decide whether you want to install the NetQ Platform on:
    - a virtual machine (VM) on hardware that you provide, or
    - the Cumulus NetQ Appliance.

2. Review the VM requirements if you have chosen that option.

3. Obtain the NetQ Platform image and setup the VM or appliance.

## Prepare Your KVM VM and Obtain the NetQ Platform

The first preparation step is to verify your VM meets the following *minimum* hardware and software requirements to ensure the VM can operate correctly.

### Virtual Machine Requirements

The NetQ Platform requires a VM with the following system resources allocated:

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>Resource</th>
<th>Minimum Requirement</th>
</tr>
</thead>
<tbody>
<tr class="even">
<td>Processor</td>
<td>Eight (8) virtual CPUs</td>
</tr>
<tr class="odd">
<td>Memory</td>
<td>64 GB RAM</td>
</tr>
<tr class="even">
<td>Local disk storage</td>
<td>256 GB SSD <br>(<strong>Note</strong>: This <em>must</em> be an SSD; use of other storage options can lead to system instability and are not supported.)</td>
</tr>
<tr class="odd">
<td>Network interface speed</td>
<td>1 Gb NIC</td>
</tr>
<tr class="even">
<td>Hypervisor</td>
<td><ul><li>VMware ESXi™ 6.5 or later (OVA image) for servers running Cumulus Linux, CentOS, Ubuntu and RedHat operating systems</li><li>KVM/QCOW (QEMU Copy on Write) image for servers running CentOS, Ubuntu and RedHat operating systems</li></ul></td>
</tr>
</tbody>
</table>

### Required Open Ports

You must also open the following ports on your NetQ Platform (or platforms if you are planning to deploy a server cluster).

For external connections:

| Port  |  Protocol | Component Access |
| ------: |  :-----: | ----- |
| 8443 |  TCP | Admin UI |
| 443 | TCP | NetQ UI |
| 31980 | TCP | NetQ Agent communication |
| 32708 | TCP | API Gateway |
| 22 | TCP | SSH |

For internal cluster communication:

| Port  |  Protocol | Component Access |
| ------: |  :-----: | ----- |
| 8080 | TCP | Admin API |
| 5000 | TCP | Docker registry |
| 8472 | UDP | Flannel port for VXLAN |
| 6443 | TCP | Kubernetes API server |
| 10250 | TCP | kubelet health probe |
| 2379 | TCP | etcd |
| 2380 | TCP | etcd |
| 7072 | TCP | Kafka JMX monitoring |
| 9092 | TCP | Kafka client |
| 7071 | TCP | Cassandra JMX monitoring |
| 7000 | TCP | Cassandra cluster communication |
| 9042 | TCP | Cassandra client |
| 7073 | TCP | Zookeeper JMX |
| 2888 | TCP | Zookeeper cluster communication |
| 3888 | TCP | Zookeeper cluster communication |
| 2181 | TCP | Zookeeper client |

{{%notice tip%}}
Port 32666 is no longer used for the NetQ UI.
{{%/notice%}}

The second preparation step is to follow the instructions below, based on whether you intend to deploy a single-server platform or a three-server cluster.

### KVM Single-Server Deployment

Two steps are needed, one to download the NetQ Platform and one to configure the VM.

#### Download the KVM NetQ Platform Image

**IMPORTANT**: Confirm that your server hardware meets the requirements identified in {{<link url="#virtual-machine-requirements" text="Virtual Machine Requirements">}}.

1. On the {{<exlink url="https://cumulusnetworks.com/downloads/" text="Cumulus Downloads">}} page, select *NetQ* from the **Product** list.

2. Click *2.4* from the **Version** list, and then select
    *2.4.1* from the submenu.

3. Select *KVM* from the **HyperVisor/Platform** list.

    {{< figure src="/images/netq/netq-24-download-options-241.png" width="500" >}}

4. Scroll down to view the image, and click **Download**.

    {{< figure src="/images/netq/netq-24-vm-dwnld-kvm-241.png" width="200" >}}

#### Configure the KVM VM

1. Open your hypervisor and set up your VM.  

    You can use this example for reference or use your own hypervisor instructions.

    <details><summary>KVM Example Configuration</summary>

    This example shows the VM setup process for a system with Libvirt and KVM/QEMU installed.

    1. Confirm that the SHA256 checksum matches the one posted on the Cumulus Downloads website to ensure the image download has not been corrupted.

    ```
    $ sha256sum ./Downloads/cumulus-netq-server-2.4.1-ts-amd64-qemu.qcow2
    $ 6fff5f2ac62930799b4e8cc7811abb6840b247e2c9e76ea9ccba03f991f42424  ./Downloads/cumulus-netq-server-2.4.1-ts-amd64-qemu.qcow2
    ```

    2. Copy the QCOW2 image to a directory where you want to run it.

        {{%notice tip%}} 
Copy, instead of moving, the original QCOW2 image that was downloaded to avoid re-downloading it again later should you need to perform this process again.
        {{%/notice%}}

    ```
    $ sudo mkdir /vms
    $ sudo cp ./Downloads/cumulus-netq-server-2.4.1-ts-amd64-qemu.qcow2 /vms/ts.qcow2
    ```

    3. Create the VM.

        For a Direct VM, where the VM uses a MACVLAN interface to sit on the host interface for its connectivity:

    ```
    $ virt-install --name=netq_ts --vcpus=8 --memory=65536 --os-type=linux --os-variant=debian7 --disk path=/vms/ts.qcow2,format=qcow2,bus=virtio,cache=none --network=type=direct,source=eth0,model=virtio --import --noautoconsole
    ```

        {{%notice note%}}
Replace the disk path value with the location where the QCOW2 image is to reside. Replace network model value (eth0 in the above example) with the name of the interface where the VM is connected to the external network.
        {{%/notice%}}

        Or, for a Bridged VM, where the VM attaches to a bridge which has already been setup to allow for external access:

    ```
    $ virt-install --name=netq_ts --vcpus=8 --memory=65536 --os-type=linux --os-variant=debian7 \ --disk path=/vms/ts.qcow2,format=qcow2,bus=virtio,cache=none --network=bridge=br0,model=virtio --import --noautoconsole
    ```

        {{%notice note%}}
Replace network bridge value (br0 in the above example) with the name of the (pre-existing) bridge interface where the VM is connected to the external network.
        {{%/notice%}}

    4.  Watch the boot process in another terminal window.

    ```
    $ virsh console netq_ts
    ```

    5.  From the Console of the VM, check to see which IP address Eth0 has obtained via DHCP, or alternatively set a static IP address by viewing the */etc/netplan/01-ethernet.yaml* Netplan configuration file:

    ```
    # This file describes the network interfaces available on your system
    # For more information, see netplan(5).
    network:
        version: 2
        renderer: networkd
        ethernets:
            eno0:
                dhcp4: no
                addresses: [192.168.1.222/24]
                gateway4: 192.168.1.1
                nameservers:
                    addresses: [8.8.8.8,8.8.4.4]
    ```

        This example show that the IP address is a static address. If this is desired, exit the file without changes. If you wanted the IP address to be determined by DHCP, edit the file as follows:

        ```
        network:
            version: 2
            renderer: networkd
            ethernets:
                eno0:
                    dhcp4: yes
        ```

        Apply the settings.

        ```
        $ sudo netplan apply
        ```

    </details>

4. Verify the platform is ready for installation. Fix any errors indicated before installing the NetQ software.

    ```
    cumulus@<hostname>:~$ sudo opta-check
    ```
    
5. Run the Bootstrap CLI on the platform *for the interface you defined above* (eth0 or eth1 for example). This example uses the eth0 interface.

    ```
    cumulus@<hostname>:~$ netq bootstrap master interface eth0 tarball /mnt/installables/netq-bootstrap-2.4.1.tgz
    ```

    Allow about five minutes for this to complete,  and only then continue to the next step.

    {{%notice tip%}}
If this step fails for any reason, you can run `netq bootstrap reset` and then try again.
    {{%/notice%}}

You are now ready to install the Cumulus NetQ software.  Refer to {{<link title="Install NetQ Using the Admin UI">}} (recommended) or {{<link title="Install NetQ Using the CLI">}}.

### KVM Three-Server Cluster

To prepare a three-server cluster is similar to preparing a single server configuration. For the master server, follow the instructions for the single server, then continue here:

1. Copy the file you downloaded for the single server to the other two servers.

2. On each worker node, open your hypervisor and setup the VM in the same manner as for the single server.

    {{%notice note%}}
Make a note of the private IP addresses you assign to the master and two worker nodes. They are needed for the installation steps.
    {{%/notice%}}

3. Verify the server is ready for installation. Fix any errors indicated before installing the NetQ software.

    ```
    cumulus@<hostname>:~$ sudo opta-check
    ```
    
4. Run the Bootstrap CLI on each worker node for the interface you defined above (eth0 or eth1 for example). This example uses the eth0 interface.

    ```
    cumulus@<hostname>:~$ netq bootstrap worker interface eth0 tarball /mnt/installables/netq-bootstrap-2.4.1.tgz
    ```

    Allow about five minutes for this to complete,  and only then continue to the next step.

    {{%notice tip%}}
If this step fails for any reason, run `netq bootstrap reset` and then try again.
    {{%/notice%}}

You are now ready to install the Cumulus NetQ software.  Refer to {{<link title="Install NetQ Using the Admin UI">}} (recommended) or {{<link title="Install NetQ Using the CLI">}}.

## Prepare Your VMware VM and Obtain NetQ Platform

The first preparation step is to verify your VM meets the following *minimum* hardware and software requirements to ensure the VM can operate correctly.

### Virtual Machine Requirements

The NetQ Platform requires a VM with the following system resources allocated:

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>Resource</th>
<th>Minimum Requirement</th>
</tr>
</thead>
<tbody>
<tr class="even">
<td>Processor</td>
<td>Eight (8) virtual CPUs</td>
</tr>
<tr class="odd">
<td>Memory</td>
<td>64 GB RAM</td>
</tr>
<tr class="even">
<td>Local disk storage</td>
<td>256 GB SSD <br>(<strong>Note</strong>: This <em>must</em> be an SSD; use of other storage options can lead to system instability and are not supported.)</td>
</tr>
<tr class="odd">
<td>Network interface speed</td>
<td>1 Gb NIC</td>
</tr>
<tr class="even">
<td>Hypervisor</td>
<td><ul><li>VMware ESXi™ 6.5 or later (OVA image) for servers running Cumulus Linux, CentOS, Ubuntu and RedHat operating systems</li><li>KVM/QCOW (QEMU Copy on Write) image for servers running CentOS, Ubuntu and RedHat operating systems</li></ul></td>
</tr>
</tbody>
</table>

### Required Open Ports

You must also open the following ports on your NetQ Platform (or platforms if you are planning to deploy a server cluster).

For external connections:

| Port  |  Protocol | Component Access |
| ------: |  :-----: | ----- |
| 8443 |  TCP | Admin UI |
| 443 | TCP | NetQ UI |
| 31980 | TCP | NetQ Agent communication |
| 32708 | TCP | API Gateway |
| 22 | TCP | SSH |

For internal cluster communication:

| Port  |  Protocol | Component Access |
| ------: |  :-----: | ----- |
| 8080 | TCP | Admin API |
| 5000 | TCP | Docker registry |
| 8472 | UDP | Flannel port for VXLAN |
| 6443 | TCP | Kubernetes API server |
| 10250 | TCP | kubelet health probe |
| 2379 | TCP | etcd |
| 2380 | TCP | etcd |
| 7072 | TCP | Kafka JMX monitoring |
| 9092 | TCP | Kafka client |
| 7071 | TCP | Cassandra JMX monitoring |
| 7000 | TCP | Cassandra cluster communication |
| 9042 | TCP | Cassandra client |
| 7073 | TCP | Zookeeper JMX |
| 2888 | TCP | Zookeeper cluster communication |
| 3888 | TCP | Zookeeper cluster communication |
| 2181 | TCP | Zookeeper client |

{{%notice tip%}}
Port 32666 is no longer used for the NetQ UI.
{{%/notice%}}

The second preparation step is to follow the instructions below, based on whether you intend to deploy a single-server platform or a three-server cluster.

### VMware Single-Server Arrangement

Two steps are needed, one to download the NetQ Platform and one to configure the VM.

#### Download the VMware NetQ Platform Image

**IMPORTANT**: Confirm that your server hardware meets the requirements identified in {{<link url="#virtual-machine-requirements" text="Virtual Machine Requirements">}}.

1. On the {{<exlink url="https://cumulusnetworks.com/downloads/" text="Cumulus Downloads">}} page, select *NetQ* from the **Product** list.

2. Click *2.4* from the **Version** list, and then select
    *2.4.1* from the submenu.

3. Select *VMware* from the **HyperVisor/Platform** list.

    {{< figure src="/images/netq/netq-24-download-options-241.png" width="500" >}}

4. Scroll down to view the image, and click **Download**.

    {{< figure src="/images/netq/netq-24-vm-dwnld-vmware-241.png" width="200" >}}

#### Configure the VMware VM

1. Open your hypervisor and set up your VM.

    You can use this examples for reference or use your own hypervisor instructions.

    <details><summary>VMware Example Configuration</summary>

    This example shows the VM setup process using an OVA file with VMware ESXi.

      1. Enter the address of the hardware in your browser.

      2. Log in to VMware using credentials with root access.  

          {{< figure src="/images/netq/vmw-main-page.png" width="700" >}}

      3. Click **Storage** in the Navigator to verify you have an SSD installed.  

          {{< figure src="/images/netq/vmw-verify-storage.png" width="700" >}}

      4. Click **Create/Register VM** at the top of the right pane.

          {{< figure src="/images/netq/vmw-menu-create-register.png" width="700" >}}

      5. Select **Deploy a virtual machine from an OVF or OVA file**, and
          click **Next**.  

          {{< figure src="/images/netq/vmw-deploy-vm-from-ova.png" width="700" >}}

      6. Provide a name for the VM, for example *Cumulus NetQ*.

      7. Drag and drop the NetQ Platform image file you downloaded in Step 2 above.

      8. Click **Next**.

          {{< figure src="/images/netq/vmw-name-the-vm.png" width="700" >}}

      9. Select the storage type and data store for the image to use, then
          click **Next**. In this example, only one is available.

          {{< figure src="/images/netq/vmw-select-storage.png" width="700" >}}

      10. Accept the default deployment options or modify them according to
          your network needs. Click **Next** when you are finished.

          {{< figure src="/images/netq/vmw-default-deploy-options.png" width="700" >}}

      11. Review the configuration summary. Click **Back** to change any of
          the settings, or click **Finish** to continue with the creation of
          the VM.

          {{< figure src="/images/netq/vmw-review-before-create.png" width="700" >}}

          The progress of the request is shown in the Recent Tasks window at the bottom of the application. This may take some time, so continue with your other work until the upload finishes.

      12. Once completed, view the full details of the VM and hardware.

          {{< figure src="/images/netq/vmw-deploy-results.png" width="700" >}}

     </details>

2. Verify the platform is ready for installation. Fix any errors indicated before installing the NetQ software.

    ```
    cumulus@<hostname>:~$ sudo opta-check
    ```

3. Run the Bootstrap CLI on the platform for the interface you defined above (eth0 or eth1 for example). This example uses the eth0 interface.

    ```
    cumulus@<hostname>:~$ netq bootstrap master interface eth0 tarball /mnt/installables/netq-bootstrap-2.4.1.tgz
    ```

    Allow about five minutes for this to complete,  and only then continue to the next step.

    {{%notice tip%}}
If this step fails for any reason, you can run `netq bootstrap reset` and then try again.
    {{%/notice%}}

You are now ready to install the Cumulus NetQ software.  Refer to {{<link title="Install NetQ Using the Admin UI">}} (recommended) or {{<link title="Install NetQ Using the CLI">}}.

### VMware Three-Server Cluster

To prepare a three-server cluster is similar to preparing a single server configuration. For the master server, follow the instructions for the single server, then continue here:

1. Copy the file you downloaded for the single server to the other two servers.

2. On each worker node, open your hypervisor and setup the VM in the same manner as for the single server.

    {{%notice note%}}
Make a note of the private IP addresses you assign to the master and two worker nodes. They are needed for the installation steps.
    {{%/notice%}}

3. Verify the platform is ready for installation. Fix any errors indicated before installing the NetQ software.

    ```
    cumulus@<hostname>:~$ sudo opta-check
    ```
    
4. Run the Bootstrap CLI on each worker node for the interface you defined above (eth0 or eth1 for example). This example uses the eth0 interface.

    ```
    cumulus@<hostname>:~$ netq bootstrap worker interface eth0 tarball /mnt/installables/netq-bootstrap-2.4.1.tgz
    ```

    Allow about five minutes for this to complete,  and only then continue to the next step.

    {{%notice tip%}}
If this step fails for any reason, you can run `netq bootstrap reset` and then try again.
    {{%/notice%}}

You are now ready to install the Cumulus NetQ software.  Refer to {{<link title="Install NetQ Using the Admin UI">}} (recommended) or {{<link title="Install NetQ Using the CLI">}}.

## Prepare Your Cumulus NetQ Appliance

Follow the preparation instructions below, based on whether you intend to deploy a single NetQ Appliance or three NetQ Appliances as a cluster.

### Single NetQ Appliance

To prepare your single NetQ Appliance:

Inside the box that was shipped to you, you'll find:

- Your Cumulus NetQ Appliance (a Supermicro 6019P-WTR server)
- Hardware accessories, such as power cables and rack mounting gear (note that network cables and optics ship separately)
- Information regarding your order

For more detail about hardware specifications (including LED layouts and FRUs like the power supply or fans, and accessories like included cables) or safety and environmental information, refer to the {{<exlink url="https://www.supermicro.com/manuals/superserver/1U/MNL-1943.pdf" text="user manual">}} and {{<exlink url="https://www.supermicro.com/QuickRefs/superserver/1U/QRG-1943.pdf" text="quick reference guide">}}.

#### Install the Appliance

After you unbox the appliance:

1. Mount the appliance in the rack.
2. Connect it to power following the procedures described in your appliance's user manual.
3. Connect the Ethernet cable to the 1G management port (eth0).
4. Power on the appliance.

   {{< figure src="/images/netq/netq-appliance-port-connections.png" width="700" caption="NetQ Appliance connections">}}

If your network runs DHCP, you can configure Cumulus NetQ over the network. If DHCP is not enabled, then you configure the appliance using the console cable provided.

#### Configure the Password, Hostname and IP Address

Change the password using the `passwd` command:

```
$ passwd 
Changing password for <user>.
(current) UNIX password: 
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```

By default, DHCP is used to acquire the hostname and IP address. However, you can manually specify the hostname with the following command:

```
sudo hostnamectl set-hostname <newHostNameHere>
```

You can also configure these items using the Ubuntu Netplan configuration tool. For example, to set your network interface *eth0* to a static IP address of *192.168.1.222* with gateway *192.168.1.1* and DNS server as *8.8.8.8* and *8.8.4.4*:

Edit the */etc/netplan/01-ethernet.yaml* Netplan configuration file:

    ```
    # This file describes the network interfaces available on your system
    # For more information, see netplan(5).
    network:
        version: 2
        renderer: networkd
        ethernets:
            eno0:
                dhcp4: no
                addresses: [192.168.1.222/24]
                gateway4: 192.168.1.1
                nameservers:
                    addresses: [8.8.8.8,8.8.4.4]
    ```

Apply the settings.

```
$ sudo netplan apply
```

{{%notice info%}}
If you have changed the IP address or hostname of the NetQ Appliance, you need to
re-register this address with the Kubernetes containers before you can
continue.

1. Reset all Kubernetes administrative settings. Run the command twice to make sure all directories and files have been reset.
    ```
    cumulus@netq-platform:~$ sudo kubeadm reset -f
    ```  

2. Remove the Kubernetes configuration.  
    ```
    cumulus@netq-platform:~$ sudo rm /home/cumulus/.kube/config
    ```

3. Reset the NetQ Platform install daemon.  
    ```
    cumulus@netq-platform:~$ sudo systemctl reset-failed
    ```  

4. Reset the Kubernetes service.  
    ```
    cumulus@netq-platform:~$ sudo systemctl restart cts-kubectl-config
    ```  
    **Note**: Allow 15 minutes for the prompt to return.
{{%/notice%}}

#### Verify NetQ Software and Appliance Readiness

Now that the appliance is up and running, verify that the software is available and the appliance is ready for installation.

1. Verify that the needed packages are present and of the correct release, version 2.4.1 and update 26 or later.

    ```
    cumulus@<hostname>:~$ dpkg -l | grep netq
    ```

    For Ubuntu 18.04, you should see:
    
    ```
    ii  netq-agent   2.4.1-ub18.04u26~1581351889.c5ec3e5 amd64   Cumulus NetQ Telemetry Agent for Ubuntu
    ii  netq-apps    2.4.1-ub18.04u26~1581351889.c5ec3e5 amd64   Cumulus NetQ Fabric Validation Application for Ubuntu
    ```

    For Ubuntu 16.04, you should see:

    ```
    ii  netq-agent   2.4.1-ub16.04u26~1581350451.c5ec3e5 amd64   Cumulus NetQ Telemetry Agent for Ubuntu
    ii  netq-apps    2.4.1-ub16.04u26~1581350451.c5ec3e5 amd64   Cumulus NetQ Fabric Validation Application for Ubuntu
    ```

2. Verify the installation images are present and of the correct release, version 2.4.1.

    ```
    cumulus@<hostname>:~$ cd /mnt/installables/
    cumulus@<hostname>:/mnt/installables$ ls
    NetQ-2.4.1.tgz  netq-bootstrap-2.4.1.tgz
    ```

3. Run the following commands.

```
sudo systemctl disable apt-{daily,daily-upgrade}.{service,timer}
sudo systemctl stop apt-{daily,daily-upgrade}.{service,timer}
sudo systemctl disable motd-news.{service,timer}
sudo systemctl stop motd-news.{service,timer}
```

3. Verify the appliance is ready for installation. Fix any errors indicated before installing the NetQ software.

    ```
    cumulus@<hostname>:~$ sudo opta-check
    ```

4. Run the Bootstrap CLI on the appliance *for the interface you defined above* (eth0 or eth1 for example). This example uses the *eth0* interface.

    ```
    cumulus@<hostname>:~$ netq bootstrap master interface eth0 tarball /mnt/installables/netq-bootstrap-2.4.1.tgz
    ```

    Allow about five minutes for this to complete,  and only then continue to the next step.

    {{%notice tip%}}
If this step fails for any reason, you can run `netq bootstrap reset` and then try again.
    {{%/notice%}}

You are now ready to install the Cumulus NetQ software.  Refer to {{<link title="Install NetQ Using the Admin UI">}} (recommended) or {{<link title="Install NetQ Using the CLI">}}.

### Three-Appliance Cluster

To prepare a three-appliance cluster is similar to preparing a single server. For the master appliance, follow the instructions for a single appliance, then return here to configure the worker appliances.

1. Install the second NetQ Appliance using the same steps as a single NetQ Appliance.

2. Configure the IP address, hostname, and password using the same steps as a single NetQ Appliance.
    {{%notice note%}}
Make a note of the private IP addresses you assign to the master and two worker nodes. They are needed for the installation steps.
    {{%/notice%}}

3. Copy the *netq-bootstrap-2.4.1.tgz* and *NetQ-2.4.1.tgz* files,  downloaded for the single NetQ Appliance, to the */mnt/installables/* directory on the second NetQ Appliance and run the `systemctl` commands.

4. Verify that the needed files are present and of the correct release.

5. Verify the platform is ready for installation. Fix any errors indicated before installing the NetQ software.

    ```
    cumulus@<hostname>:~$ sudo opta-check
    ```

6. Run the Bootstrap CLI on the appliance *for the interface you defined above* (eth0 or eth1 for example). This example uses the *eth0* interface.

    ```
    cumulus@<hostname>:~$ netq bootstrap worker interface eth0 tarball /mnt/installables/netq-bootstrap-2.4.1.tgz
    ```

    Allow about five minutes for this to complete,  and only then continue to the next step.

    {{%notice tip%}}
If this step fails for any reason, you can run `netq bootstrap reset` and then try again.
    {{%/notice%}}

7. Repeat these steps for the third NetQ Appliance.

You are now ready to install the Cumulus NetQ software.  Refer to {{<link title="Install NetQ Using the Admin UI">}} (recommended) or {{<link title="Install NetQ Using the CLI">}}.
