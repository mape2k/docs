---
title: Quick Start
author: Cumulus Networks
weight: 20
product: Cumulus Networks Guides
version: "1.0"
draft: true
---
You can get started with the PRA in one two ways:

- Use {{<exlink url="https://cumulusnetworks.com/products/cumulus-in-the-cloud/" text="Cumulus in the Cloud">}} to try the PRA directly on Cumulus infrastructure.
- Use your own Vagrant, libvirt and KVM server. This requires a minimum of 15GB of ram with a preferred 24GB of RAM for the demo environment.

## Use Cumulus in the Cloud

To use Cumulus in the Cloud, you need to request a demo environment:

1. Visit {{<exlink url="https://cumulusnetworks.com/products/cumulus-in-the-cloud/" text="Cumulus in the Cloud">}} and click **Build a Simulation**.

{{%notice note%}}

You do not have to be a current Cumulus Networks customer to use Cumulus in the Cloud, but you do need to register for a `cumulusnetworks.com` account to start a simulation.

{{%/notice%}}

It takes just a few minutes to launch the simulation. You receive an email with a link to your lab when it is ready.

2. After the lab starts, the **Select a demo** menu on the left side navigation lists the PRA demos you can run.

{{<img src="/images/guides/citc-choose-demo.png">}}

Select your demo and click **Run Now** to fully provision the demo environment.

{{<img src="/images/guides/citc-run-demo.gif">}}

3. When the lab is provisioned, you can click directly on a device to access the console, login, and run commands.

{{<img src="/images/guides/citc-access-console.gif">}}

{{% notice tip %}}

The default Cumulus Linux username `cumulus` and password `CumulusLinux!` is used on all devices in the lab.

{{% /notice %}}

4. (Optional) If you want to see or modify any playbooks in the lab, access the oob-mgmt-server through the CITC **Advanced** button on the left navigation bar. This provides you with information on how to SSH to the oob-mgmt-server.

{{<img src="/images/guides/citc-oob-ssh.gif">}}

You can scroll down to the **Services** pane and see the `ssh oob-mgmt-server` service. The link launches an SSH session. If this link fails, you can ssh to `air.cumulusnetworks.com` using the port defined in `External Port`.

5. (Optional) From the `/home/cumulus` directory on the oob-mgmt-server, you see a folder name matching the repository of the demo you selected; for example, `dc_configs_vxlan_evpnsym`. This is directly cloned from the {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle" text="Golden Turtle" >}} repository.

## Use Vagrant and Libvirt

{{% notice tip %}}

The full {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle/cldemo2/" text="cldemo2" >}} topology requires a minimum of **2.4 GB** of RAM. Without NetQ it requires a minimum of **1.6 GB** of RAM.

For more information on the minimum requirements and tested software versions, refer to the {{<link title="Run Production Ready Automation" text="User Guide" >}}.

{{% /notice %}}

### Start a Golden Standard Demo Topology

The following procedure describes the easiest way to start a PRA demo using a bash script provided in the package. The bash script performs the following steps automatically:

- Checks if the Cumulus Networks reference topology submodule is present and attempts to download the reference topology if it is not present.
- Runs the `vagrant up` command for the out-of-band management network devices.
- Runs a series of `vagrant up` commands to start the rest of the network simulation.
- Runs the `vagrant scp` command to copy the network automation into the simulation.

{{< notice note>}}

The order in which devices start is critical to the proper functioning of the lab environment. If the oob-server and oob-switch are not online before the network, out of band management network DHCP fails.

To control which nodes start and in which order, and to save CPU and memory resources, you can run the simulation manually. Refer to {{<link text="Run the Production Ready Automation" title="Run Production Ready Automation" >}}.

{{< /notice >}}

To start a golden standard demo topology using a bash script:

1. From the {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle" text="Golden Turtle GitLab page">}}, select the demo topology that you want to use.

2. Run the `git clone` command for your selected Golden Turtle demo using the `--recurse-submodules` argument to also download the required base reference topology:

    ```
    user@host:~# git clone --recurse-submodules https://gitlab.com/cumulus-consulting/goldenturtle/dc_configs_vxlan_evpnsym.git
    Cloning into 'dc_configs_vxlan_evpnsym'...
    remote: Enumerating objects: 535, done.
    remote: Counting objects: 100% (535/535), done.
    remote: Compressing objects: 100% (300/300), done.
    remote: Total 535 (delta 198), reused 465 (delta 133), pack-reused 0
    Receiving objects: 100% (535/535), 66.78 KiB | 1.45 MiB/s, done.
    Resolving deltas: 100% (198/198), done.
    Submodule 'cldemo2' (https://gitlab.com/cumulus-consulting/goldenturtle/cldemo2.git) registered for path 'cldemo2'
    Cloning into '/root/dc_configs_vxlan_evpnsym/cldemo2'...
    remote: Enumerating objects: 215, done.
    remote: Counting objects: 100% (215/215), done.
    remote: Compressing objects: 100% (119/119), done.
    remote: Total 852 (delta 139), reused 156 (delta 96), pack-reused 637
    Receiving objects: 100% (852/852), 7.64 MiB | 8.21 MiB/s, done.
    Resolving deltas: 100% (549/549), done.
    Submodule path 'cldemo2': checked out '214c8a66e3fef8f6d5d2b1e13ca3942e4cfd120f'
    ```

3. Change to the directory created in the previous step. The directory name matches the name of the project from the git clone operation.

    ```
    user@host:~# cd dc_configs_vxlan_evpnsym/
    user@host:~/dc_configs_vxlan_evpnsym#
    ```

4. Run the `start-demo.sh` script. If you do not intend to use NetQ, add the `--no-netq` option.

{{%notice note%}}

An Ubuntu 18.04LTS box with additional CPU and memory resources for installing NetQ is included in the out-of-band management network of the base Cumulus Networks reference topology. NetQ is *not* a required element for any of the golden standard demos to function but is used in the topology to power CI/CD testing, and to preview and test the NetQ functionality. If you do not intend to use NetQ, Cumulus Networks recommends that you do not start it in simulation to save an additional 8GB of memory.

{{%/notice%}}

    ```
    user@host:~/dc_configs_vxlan_evpnsym# ./start-demo.sh --no-netq
    Starting OOB management devices
    Bringing machine 'oob-mgmt-server' up with 'libvirt' provider...
    Bringing machine 'oob-mgmt-switch' up with 'libvirt' provider...
    ==> oob-mgmt-server: Checking if box 'generic/ubuntu1804' version '2.0.6' is up to date...
    ==> oob-mgmt-switch: Checking if box 'CumulusCommunity/cumulus-vx' version '3.7.11' is up to date...
    ==> oob-mgmt-server: Creating image (snapshot of base box volume).
    ==> oob-mgmt-switch: Creating image (snapshot of base box volume).
    ==> oob-mgmt-server: Creating domain with the following settings...
    <output omitted for brevity>
    ```

     During this step, Vagrant starts and provisions the base simulation. It might take 15 or more minutes for this step to complete depending on the host machine and internet connection speed.

5. After the launch script succeeds, change to the `cldemo2/simulation` directory:

    ```
    user@host:~/dc_configs_vxlan_evpnsym# cd cldemo2/simulation/
    user@host:~/dc_configs_vxlan_evpnsym/cldemo2/simulation#
    ```

6. Run the `vagrant ssh oob-mgmt-server` command to enter the simulation onto the oob-mgmt-server. This is your jump box to reach the other devices in the network.

    ```
    user@host:~/dc_configs_vxlan_evpnsym/cldemo2/simulation# vagrant ssh oob-mgmt-server
                                                     _
          _______   x x x                           | |
     ._  <_______~ x X x   ___ _   _ _ __ ___  _   _| |_   _ ___
    (' \  ,' || `,        / __| | | | '_ ` _ \| | | | | | | / __|
     `._:^   ||   :>     | (__| |_| | | | | | | |_| | | |_| \__ \
         ^T~~~~~~T'       \___|\__,_|_| |_| |_|\__,_|_|\__,_|___/
         ~"     ~"

    ############################################################################
    #
    #         Out Of Band Management Server (oob-mgmt-server)
    #
    ############################################################################
    vagrant@oob-mgmt-server:~$
    ```

7. Change to the automation directory:

    ```
    vagrant@oob-mgmt-server:~$ cd automation/
    vagrant@oob-mgmt-server:~/automation$
    ```

8. Run the ansible playbook to configure your selected demo:

    {{%notice note%}}

The `-i` flag is used to specify the location of the Ansible inventory. This is required, unless the inventory is moved into a {{<exlink url="https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html" text="standard Ansible location">}}.

{{%/notice%}}

    ```
    vagrant@oob-mgmt-server:~/automation$ ansible-playbook playbooks/deploy.yml -i inventories/pod1 --diff

    PLAY [spine leaf border] ****************************************************************

    TASK [Gathering Facts] ****************************************************************
    Sunday 12 April 2020  23:35:25 +0000 (0:00:00.073)       0:00:00.073 ********** 
    ok: [spine03]
    ok: [spine04]
    ok: [spine02]
    ok: [spine01]
    ok: [leaf01]
    ok: [leaf02]
    ok: [leaf03]
    ok: [leaf04]
    ok: [border02]
    ok: [border01]
    <output omitted for brevity>
    ```

Check the `README.md` file on the selected demo repository for more information about the topology, such as IP addresses, and for a small guided tour of the specific technologies or architectures in the demo.

### Start a Blank Reference Topology

The Cumulus Networks reference topology is included as a submodule in all the Cumulus Networks golden standard demos, which eliminates the need to clone the base reference topology project. <!-- For more information about submodules see the contributor’s guide. TODO: add link to contributor's guide -->

You can start the reference topology by itself if you want to build configuration from scratch or intend to start from a completely blank slate network topology.

1. Clone the Cumulus Networks reference topology (`cldemo2`):

    ```
    user@host:~# git clone https://gitlab.com/cumulus-consulting/goldenturtle/cldemo2.git
    Cloning into 'cldemo2'...
    remote: Enumerating objects: 215, done.
    remote: Counting objects: 100% (215/215), done.
    remote: Compressing objects: 100% (119/119), done.
    remote: Total 852 (delta 139), reused 156 (delta 96), pack-reused 637
    Receiving objects: 100% (852/852), 7.64 MiB | 8.21 MiB/s, done.
    Resolving deltas: 100% (549/549), done.
    ```

2. Change to the `cldemo2` directory created by the previous step:

    ```
    user@host:~# cd cldemo2/
    user@host:~/cldemo2#
    ```  

3. Run the `start-blank-topology.sh` script to launch the simulation. If you do not intend to use NetQ, add the `--no-netq` option.

    ```
    user@host:~/cldemo2# ./start-blank-topology.sh --no-netq
    Starting OOB management devices
    Bringing machine 'oob-mgmt-switch' up with 'libvirt' provider...
    Bringing machine 'oob-mgmt-server' up with 'libvirt' provider...
    <output omitted for brevity>
    ```

    This step might take time depending on the host machine and internet connection speed.

4. After the launch script succeeds, change to the simulation directory:

    ```
    user@host:~/cldemo2# cd simulation/
    user@host:~/cldemo2/simulation#
    ```

5. Use the `vagrant ssh` command to ssh into the oob-mgmt-server. This is the jump box to access the rest of the simulation.

For more information about how to start developing and building onto this blank slate topology, refer to the `README.md` file for the Cumulus Networks {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle/cldemo2" text="reference topology GitLab project" >}}.

### Destroy and End a Simulation

To destroy *all* the systems in the simulation, run the `vagrant destroy -f` command from the `simulation` folder:

```
user@host:~/cldemo2/simulation# vagrant destroy -f
==> server08: Removing domain...
==> server07: Removing domain...
==> server06: Removing domain...
==> server05: Removing domain...
==> server04: Removing domain...
==> server03: Removing domain...
==> server02: Removing domain...
==> server01: Removing domain...
==> netq-ts: Removing domain...
==> leaf04: Removing domain...
==> leaf03: Removing domain...
==> leaf02: Removing domain...
==> leaf01: Removing domain...
==> fw2: Removing domain...
==> fw1: Removing domain...
==> spine04: Removing domain...
==> spine03: Removing domain...
==> spine02: Removing domain...
==> spine01: Removing domain...
==> border02: Removing domain...
==> border01: Removing domain...
==> oob-mgmt-switch: Removing domain...
==> oob-mgmt-server: Removing domain...
user@host:~/cldemo2/simulation#
```

To destroy an individual system, specify the system name in the `vagrant destroy` command:

```
user@host:~/cldemo2/simulation# vagrant destroy server08 -f
==> server08: Removing domain...
user@host:~/cldemo2/simulation#
```
