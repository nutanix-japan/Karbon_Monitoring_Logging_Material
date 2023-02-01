---
title: Linux Tools VM
---

## Overview

This CentOS VM image will be staged with packages used to support
multiple lab exercises.

Deploy this VM on your assigned cluster if directed to do so as part of
**Lab Setup**.

!!!caution
        Only deploy the VM once, it does not need to be cleaned up as part of any lab completion    

## Deploying CentOS

1. In **Prism Central** > select **Menu**> **Compute and Storage** and **VMs**.

2. Click on **Create VM**

3. Fill out the following fields:

    -   **Name** - *Initials*-Linux-ToolsVM

    -   **Description** - (Optional) Description for your VM.

    -   **vCPU(s)** - 2

    -   **Number of Cores per vCPU** - 1

    -   **Memory** - 4 GiB

    -   Select **Attach Disk**

        -   **Type** - DISK
        -   **Operation** - Clone from Image
        -   **Image** - CentOS7.qcow2
        -   Select **Save**

4.   Select **Attach to Subnet**

    -   **VLAN Name** - Primary
    -   Select **Save**

5. Click **Next** and **Create VM** to create the VM.

6. In the list of VMs, select *Initials*-Linux-ToolsVM 

7. From the **Actions** menu, choose **Power On**.

## Installing Linux Tools

1. Login to the VM via ssh or Console session, using the following
credentials:

2.  Install the software needed by running the following commands:

    ```bash
    yum update -y
    yum install -y ntp ntpdate unzip stress nodejs python-pip s3cmd awscli
    yum install -y bind-utils nmap wget git
    ```

3. Enable and configure NTP by running the following commands:

    ``` bash
    systemctl start ntpd
    systemctl enable ntpd
    ntpdate -u -s 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
    systemctl restart ntpd
    ```

4. Disable the firewall and SELinux by running the following commands:

    ``` bash
    systemctl disable firewalld
    systemctl stop firewalld
    setenforce 0
    sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config
    ```
Now your Linux Tools VM is ready for you to use.
