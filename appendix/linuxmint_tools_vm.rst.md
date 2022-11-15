.. _linuxmint_tools_vm:

---------------------
Linux Mint Tools VM
---------------------

Overview
+++++++++

This Linux Mint VM image will be staged with packages used to support this lab exercises.

Deploy this VM on your assigned cluster if directed to do so as part of **Lab Setup**.

.. raw:: html

  <strong><font color="red">Only deploy the VM once, it does not need to be cleaned up as part of any lab completion.</font></strong>

Deploying Linux Mint VM
++++++++++++++++++++++++

In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**, and click **Create VM**.

Fill out the following fields:

- **Name** - *Initials*-Linux-ToolsVM
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 2 GiB

- Select **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - LinuxMint
    - Select **Add**

- Select **Add New NIC**
    - **VLAN Name** - Primary
    - Select **Add**

Click **Save** to create the VM.

Power on the VM.

Installing Tools
++++++++++++++++

Login to the VM via ssh or Console session, using the following credentials:

- **Username** - nutanix
- **password** - (default password)

#. Update existing packages:

   .. code-block:: bash

    sudo apt-get update

#. Paste the command in clipboard to the shell in your LinuxMint VM to install kubectl

   .. code-block:: bash

    sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubectl
