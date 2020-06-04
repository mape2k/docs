---
title: Production Ready Automation Guide
author: Cumulus Networks
weight: 10
product: Cumulus Networks Guides
version: "1.0"
draft: true
---
The Production Ready Automation (PRA) package from Cumulus Networks provides several examples of a fully operationalized, automated data center and includes:

- A standard datacenter reference topology for all examples
- A full {{<exlink url="https://www.vagrantup.com/" text="Vagrant" >}} and {{<exlink url="https://libvirt.org/" text="libvirt" >}} simulation of the Cumulus Networks reference topology, {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle/cldemo2/" text="cldemo2" >}}, that provides the foundational physical infrastructure and bootstrap configuration to support and demonstrate Cumulus Linux features and technologies
- A variety of golden standard, production grade, EVPN-VXLAN architecture reference configurations
- Best practice {{<exlink url="https://www.ansible.com/" text="Ansible" >}} automation and infrastructure as code (IaC)
- Working examples of Continuous Integration and Continuous Deployment (CI/CD) using {{<exlink url="https://gitlab.com/" text="GitLab" >}}
- CI/CD testing powered by NetQ Cloud

You can use this Production Ready Automation package as a learning resource and as a starting template to implement these features, technologies and operational workflows in your Cumulus Linux network environments.

## Defining Production Ready

Cumulus Networks has purposefully defined these published configurations, automations and integrations as "Production Ready" to reflect the fact that they are actively deployed in the production networks of dozens of Cumulus customers. All of the components contained within the Production Ready Automation are used as the starting point for all paid {{<exlink url="https://cumulusnetworks.com/products/consulting/" text="Cumulus Services" >}} engagements. Any modifications or learnings from those customer engagements are committed back into the PRA repositories. The PRA repositories always reflect the current best practices for datacenter automation and configuration.

## The Value of Production Ready Automation

Cumulus created the Production Ready Automation to assist customers anywhere they are on the Cumulus Linux automation and deployment journey. Any or all of the PRA content can be used directly against a new or existing datacenter deployment or can be used as a best practice implementation.

We see a vast majority of our customers deploying EVPN-VXLAN networks along with common features including DNS, SNMP, Syslog, NetQ and NTP. This list will grow over time as we identify more common deployments and best practices.

Not only do the PRA collections include full Ansible configurations for these features, but it also includes {{<exlink url="https://docs.cumulusnetworks.com/cumulus-linux/Installation-Management/Zero-Touch-Provisioning-ZTP/" text="ZTP scripts" >}}, high-fidelity simulation with Vagrant and Cumulus Vx and a fully automated network testing and validation suite.

All of this is stored in a Git repository allowing for a modern infrastructure as code definition.

## Infrastructure as Code

The Production Ready Automation package contains examples of best practice Ansible automation and infrastructure as code. A completely stock Ansible core installation is used without any vendor specific or third-party plugins. Examples of Ansible best practices using roles, highly granular templates, and structured variables represent how you can store your network configurations as a highly scalable version of infrastructure as code. It is that base code that is rendered by the automation engine to produce the final configurations that exist on the network devices.

Adopting infrastructure as code allows for offline backups, simple collaboration and easy revision control. The configuration and automation stored within Git are the definitive source of truth for the infrastructure. 

## Production Ready Automation Components

A number of specific tooling choices were made as part of building PRA. These choices came from years of experience building and automating customer networks as well as our experience in the field speaking with customers running both homogeneous Cumulus Linux networks as well as mixed vendor networks.

### Automation with Ansible

Ansible was selected as the automation tool of choice due to its growing adoption by the broader networking community. Ansible as an organization has also put significant weight behind embracing network automation. Ansible's agentless nature allows it to work well with legacy networking devices as well as modern networking systems like Cumulus Linux.

Although Cumulus Linux does not rely on any networking specific Ansible modules and instead uses the off the shelf Linux server modules, Ansible has been the easiest for network engineers to learn and fits well with a network engineer's (push) model of configuration management.

We have seen a growing adoption of {{<exlink url="https://www.saltstack.com/" text="Saltstack" >}} due to its ability to have both an agentless push-based model as well as an on-box agent based pull model, we still see a vast majority of our customers adopting and satisfied with Ansible.

### Network Simulation with Vagrant, KVM and Libvirt

A number of different virtualization options, both free and commercial, exist in the marketplace today. We have found that only KVM and Libvirt meet the needs of multiple VMs with a large number of interfaces (up-to 128 per-VM) as well as the ability to transparently cluster simulations across servers. This allows us to simulate a network of any size, regardless of the power of any single server.

The use of Vagrant allows us to define the VM settings, such as memory, CPU and binary image, as well as the complex network connectivity of the VMs as code. This allows us to easily share the lab definitions in the form of a Vagrantfile as well as to version control with Git.

### GitLab Source Code Control

There are two major, public, source code repositories: GitHub and GitLab. We selected GitLab based on their CI/CD framework. GitLab relies on a "GitLab Runner", an agent that runs on a server that will run your CI tests. Our network simulation must run in VMs and not inside of containers, due to the need for a specific VM image (Cumulus VX) and multiple interconnected interfaces. Most "build" tools like {{<exlink url="https://travis-ci.com/" text="Travis-CI" >}} rely on stand alone Docker containers to test software. Our needs are more complex than a third-party build tool can support.

We run the GitLab Runner on servers pre-installed with KVM, libvirt and Vagrant and are dedicated to network simulations. When changes are made to our GitLab source code repository GitLab will notify the Runner agent and trigger the creation of a network simulation and run the associated network tests. The other advantage to using the GitLab Runner is that it operates on a pull model, where the Runner will reach outbound from the server and check with GitLab to see if there are any jobs to run. This is in contrast to {{<exlink url="https://www.jenkins.io/" text="Jenkins" >}} which is triggered based on inbound webhooks. This creates challenges in enterprise environments where inbound HTTP requests may not be allowed due to NAT or firewall policies. There are no technical limitations in using Jenkins as a build server.

### Reference Topology

The Cumulus Networks reference topology provides a common and consistent preconfigured spine and leaf base network topology, which serves as the basis for all supported Cumulus Networks demos and golden standards. This reference topology is intended to be a blank slate with minimal configuration to prepare the simulation to receive additional deployment and provisioning that demonstrates a feature or represents a fully operational production network.

The Cumulus Networks reference topology provides a complete two-tier spine and leaf topology. It also includes a complete out-of-band management. The devices include:

- Four Cumulus Linux 3.7 spines
- Four Cumulus Linux 3.7 leafs
- Eight Ubuntu 18.04 servers
- Two Cumulus Linux 3.7 border leafs
- Two Cumulus Linux 3.7 *fw* devices that provide a placeholder for *policy* devices
- One Ubuntu 18.04 out-of-band management server (oob-mgmt-server)
- One Cumulus Linux 3.7 out-of-band management switch (oob-mgmt-switch)
- One Cumulus NetQ Cloud virtual appliance (netq-ts)

{{<img src="/images/guides/cldemo2-diagram.png" >}}

When you start the reference topology simulation environment, all interfaces (except for the out-of-band management network) are unconfigured and administratively down. This is the default configuration of Cumulus Linux. The golden standard configurations and demos provide interface and routing protocol configurations that are applied to this simulation topology.

The Cumulus Networks reference topology is included with every officially-supported Cumulus Linux demo. To see a full example of the Production Ready Automation, use one of the {{<link text="EVPN-VXLAN golden standard demos" title="#golden standard demos" >}}.

Although your datacenter topology may be different, all of the concepts and principles described in the PRA automations stay the same. More details on how to customize PRA are described in the {{<link title="Customize Production Ready Automation" text="Customizing Production Ready Automation">}} section.

<!-- For information on how to build a package like this, or contribute your own demo or environment for the base Cumulus Networks reference topology, refer to the Contributors Guide.-->

## Golden Standard Demos

Cumulus Networks currently provides three officially-supported demo solutions to overlay and provision the reference topology. These demos are all EVPN-VXLAN environments and each performs tenant routing in a different style.

The golden standard demos and the underlying base reference topology are officially hosted on GitLab in the Golden Turtle folder of the {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle" text="Cumulus Consulting GitLab group">}}.

{{%notice note%}}

The name *Golden Turtle* comes from the idea of a *golden reference* and the rocket turtle, which is the Cumulus Networks mascot. Golden reference + rocket turtle = Golden Turtle.

{{%/notice%}}

- {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle/dc_configs_vxlan_evpnl2only" text="EVPN Layer 2 Only">}} is an EVPN-VXLAN environment with only a layer 2 extension.
- {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle/dc_configs_vxlan_evpncent" text="EVPN Centralized Routing">}} is an EVPN-VXLAN environment with a layer 2 extension between tenants with inter-tenant routing on a centralized (fw) device.
- {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle/dc_configs_vxlan_evpnsym" text="EVPN Symmetric Mode">}} is an EVPN-VXLAN environment with a layer 2 extension, layer 3 VXLAN routing, and VRFs for multi-tenancy.

{{%notice tip%}}

To take a *quick* look at a Cumulus Networks golden standard demo, use our simulation platform, Cumulus in the Cloud. The simulation platform has no system requirements or dependencies. Visit {{<exlink url="https://cumulusnetworks.com/products/cumulus-in-the-cloud/" text="Cumulus in the Cloud">}} to get a full blank slate Cumulus Networks reference topology. You can deploy any of the golden standard demos right from the UI with one click.

{{%/notice%}}

For more detailed information about IP addressing and included features, refer to the {{<exlink url="https://gitlab.com/cumulus-consulting/goldenturtle" text="README">}} page of the demo.

## Continuous Integration and Continuous Deployment

As network operations become more programmatic and automated, and in combination with a robust simulation platform, CI/CD and DevOps style workflows are supplanting legacy workflows. You can test configuration changes automatically in a simulated environment to allow for more rapid and robust change management workflows. The Production Ready Automation package provides an example CI/CD pipeline implemented on GitLab with the CI network testing and validation powered by {{<exlink url="https://docs.cumulusnetworks.com/cumulus-netq/" text="Cumulus NetQ">}}.
