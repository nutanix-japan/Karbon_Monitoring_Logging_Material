.. title:: Nutanix Karbon Monitoring and Logging

.. toctree:: :maxdepth: 2 :caption: Monitoring in Kubernetes :name:
\_monitoring_kubernetes :hidden:

promgraf/promgraf explore/explore connect/connect visualise/visualise
cleanup/cleanup

.. toctree:: :maxdepth: 2 :caption: Logging in Kubernetes :name:
\_logging_kubernetes :hidden:

logging/logging

.. toctree:: :maxdepth: 2 :caption: Appendix :name: \_appendix :hidden:

appendix/linuxmint_tools_vm appendix/create_kube appendix/helm
appendix/glossary

.. \_getting_started:

++++++++++++++ Introduction ++++++++++++++

Welcome to the Kubernetes Monitoring and Logging Bootcamp using Nutanix
Karbon Kubernetes clusters. This bootcamp accompanies an instructor-led
session that introduces Nutanix and Kubernetes technologies and many
common management tasks.

You will explore Prism Element and become familiar with its features and
navigation. You will use Prism to perform basic cluster administration
tasks, including storage and networking.

At the end of the bootcamp, attendees should understand the Core
concepts and technologies that make up the Nutanix Enterprise Cloud
stack and Karbon offering and should be well prepared for a hosted or
onsite proof-of-concept (POC) engagement.

.. note::

    Estimated time to complete this lab is 90 minutes

What's New +++++++++++

-   Workshop updated for the following software versions:
    -   AOS & PC 5.11.2.x
-   Optional Lab Updates:

Lab Agenda +++++++++++

Nutanix Karbon deployed kubernetes clusters come with Prometheus
pre-installed and ready to use. However, Grafana is not available by
default. We will go through the following steps to enable and visualize
the status of the kubernetes cluster.

-   Connect to Karbon deployed kubernetes cluster
-   Explore the `ntnx-system` namespace for monitoring and logging
    resources
-   Explore available Prometheus metrics
-   Deploy Grafana in `ntnx-system` namespace
-   Configure prometheus as a data source in Grafana
-   Configure a custom dashboard
-   Import a pre-configured dashboard

Initial Setup ++++++++++++++

-   Take note of the *Passwords* being used.
-   Log into your virtual desktops (connection info below)

Environment Details ++++++++++++++++++++

Nutanix Workshops are intended to be run in the Nutanix Hosted POC
environment. Your cluster will be provisioned with all necessary images,
networks, and VMs required to complete the exercises.

Networking ..........

Hosted POC clusters follow a standard naming convention:

-   **Cluster Name** - POC *XYZ*
-   **Subnet** - 10.**21**. *XYZ* .0
-   **Cluster IP** - 10.**21**. *XYZ* .37

If provisioned from the marketing pool:

-   **Cluster Name** - MKT *XYZ*
-   **Subnet** - 10.**20**. *XYZ* .0
-   **Cluster IP** - 10.**20**. *XYZ* .37

For example:

-   **Cluster Name** - POC055
-   **Subnet** - 10.21.55.0
-   **Cluster IP** - 10.21.55.37

Throughout the Workshop there are multiple instances where you will need
to substitute *XYZ* with the correct octet for your subnet, for example:

.. list-table:: :widths: 25 75 :header-rows: 1

-   -   IP Address
    -   Description

-   -   10.21. *XYZ* .37
    -   Nutanix Cluster Virtual IP

-   -   10.21. *XYZ* .39
    -   **PC** VM IP, Prism Central

-   -   10.21. *XYZ* .40
    -   **DC** VM IP, NTNXLAB.local Domain Controller

Each cluster is configured with 2 VLANs which can be used for VMs:

.. list-table:: :widths: 25 25 10 40 :header-rows: 1

-   -   Network Name
    -   Address
    -   VLAN
    -   DHCP Scope

-   -   Primary
    -   10.21. *XYZ* .1/25
    -   0
    -   10.21. *XYZ* .50-10.21. *XYZ* .124

-   -   Secondary
    -   10.21. *XYZ* .129/25
    -   *XYZ1*
    -   10.21. *XYZ* .132-10.21. *XYZ* .253

Credentials ...........

.. note::

The *`<Cluster Password>`{=html}* is unique to each cluster and will be
provided by the leader of the Workshop.

.. list-table:: :widths: 25 35 40 :header-rows: 1

-   -   Credential
    -   Username
    -   Password

-   -   Prism Element
    -   admin
    -   *`<Cluster Password>`{=html}*

-   -   Prism Central
    -   admin
    -   *`<Cluster Password>`{=html}*

-   -   Controller VM
    -   nutanix
    -   *`<Cluster Password>`{=html}*

-   -   Prism Central VM
    -   nutanix
    -   *`<Cluster Password>`{=html}*

Each cluster has a dedicated domain controller VM, **DC**, responsible
for providing AD services for the **NTNXLAB.local** domain. The domain
is populated with the following Users and Groups:

.. list-table:: :widths: 25 35 40 :header-rows: 1

-   -   Group
    -   Username(s)
    -   Password

-   -   Administrators
    -   Administrator
    -   nutanix/4u

-   -   SSP Admins
    -   adminuser01-adminuser25
    -   nutanix/4u

-   -   SSP Developers
    -   devuser01-devuser25
    -   nutanix/4u

-   -   SSP Consumers
    -   consumer01-consumer25
    -   nutanix/4u

-   -   SSP Operators
    -   operator01-operator25
    -   nutanix/4u

-   -   SSP Custom
    -   custom01-custom25
    -   nutanix/4u

-   -   Bootcamp Users
    -   user01-user25
    -   nutanix/4u

Access Instructions +++++++++++++++++++

The Nutanix Hosted POC environment can be accessed a number of different
ways:

Lab Access User Credentials ...........................

PHX Based Clusters: **Username:** PHX-POCxxx-User01 (up to
PHX-POCxxx-User20), **Password:** *`<Provided by Instructor>`{=html}*

RTP Based Clusters: **Username:** RTP-POCxxx-User01 (up to
RTP-POCxxx-User20), **Password:** *`<Provided by Instructor>`{=html}*

Frame VDI .........

Login to: https://frame.nutanix.com/x/labs

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Parallels VDI .................

PHX Based Clusters Login to: https://xld-uswest1.nutanix.com

RTP Based Clusters Login to: https://xld-useast1.nutanix.com

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Employee Pulse Secure VPN ..........................

Download the client:

PHX Based Clusters Login to: https://xld-uswest1.nutanix.com

RTP Based Clusters Login to: https://xld-useast1.nutanix.com

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Install the client.

In Pulse Secure Client, **Add** a connection:

For PHX:

-   **Type** - Policy Secure (UAC) or Connection Server
-   **Name** - X-Labs - PHX
-   **Server URL** - xlv-uswest1.nutanix.com

For RTP:

-   **Type** - Policy Secure (UAC) or Connection Server
-   **Name** - X-Labs - RTP
-   **Server URL** - xlv-useast1.nutanix.com

Nutanix Version Info ++++++++++++++++++++

-   **AHV Version** - AHV 20170830.337
-   **AOS Version** - 5.11.2.3
-   **PC Version** - 5.11.2.1
-   **Karbon Version** - 2.1.1
