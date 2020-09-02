.. _connect:

.. title:: Install Istio Service Mesh

-----------------------------
Install Istio Service Mesh
-----------------------------

In this exercise we will connect to an existing kubernetes cluster deployed using Nutanix Karbon and install Istio Service Mesh.

Overview
+++++++++

1. Create a Linux Mint VM (If one is not deployed already please use the instructions here :ref:`linuxmint_tools_vm` to deploy one
2. Connect to Linux Mint VM and install kubectl tool
3. Access you Karbon page and download KUBECONFIG file to Linux Mint VM
4. Download and install Istio Service Mesh

Connect to your LinuxMintVM
++++++++++++++++++++++++++++

#. Logon to your LinuxMint VM console as `nutanix` user (default password) and open terminal

   .. note::

       If you are using your PC/Mac you can also ssh/putty to your LinuxMint VM

       .. code-block:: bash

        ssh -l nutanix <LinuxMint VM IP address>

#. Paste the command in clipboard to the shell in your LinuxMint VM to install kubectl

   .. code-block:: bash

     sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
     curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
     echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
     sudo apt-get update
     sudo apt-get install -y kubectl

#. Verify your kubectl installation

   .. code-block:: bash

      alias 'k=kubectl'
      k version --client

Access your Kubernetes Cluster
++++++++++++++++++++++++++++++

#. Logon to your Prism Central **https://<PC VM IP>:9440**

   .. note::
     if you haven't got a Karbon deployed kubernetes cluster in your HPOC, refer here :ref:`karbon_create_cluster`

#. Go to **Menu > Services > Karbon**

   .. figure:: images/choosekarbon.png

#. Select your karbon cluster

#. Click on Actions > Download Kubeconfig

   .. figure:: images/selectcluster.png

#. Click on **Copy the command to clipboard**

#. Paste the contents in your Linux Tools VM shell

#. Run the following command to verify your connectivity and display the nodes in the cluster

   .. code-block:: bash

    k get nodes -o wide

   .. figure:: images/nodelist.png

#. You can list the namespaces, storage claims, physical volumes and physical volume claims using the following commands

   .. code-block:: bash

      k get ns
      k get sc,pv,pvc

   .. figure:: images/klistresources.png

   .. note::

     Nutanix Karbon has automatically provisioned these kubernetes resources so it is ready to use. You have the option to provision additional storage claims, physical volumes, etc by using the Karbon console or using kubectl with YAML files

Now that you have an understanding of your kubernetes cluster and available resources, go ahead and install Istio.

Download and Install Istio
+++++++++++++++++++++++++++

#. Download Istio

   .. code-block:: bash

  	curl -L https://istio.io/downloadIstio | sh -

#. Add Istio binaries path to the $PATH environment variable

   .. code-block:: bash

  	cd istio-x.x.x
  	export PATH=$PWD/bin:$PATH

#. Install Istio using ``istioctl`` tool

   .. code-block:: bash

  	istioctl install --set profile=demo

   .. figure:: images/installistio.png

   .. note::
     There are various profiles of Istio. However, we are installing the demo profile to get functionality to run this lab. Learn more about Istio install profiles by running the following commands:

     .. code-block:: bash

      istioctl profile list
      istioctl profile dump demo
      istioctl profile dump default
      istioctl profile diff demo default

#. Confirm installation by running the following command - make sure the status are successful

   .. code-block:: bash

	  istioctl verify-install

   .. figure:: images/verifyinstallistio.png

#. List, identify and explore the ``istioctl-system`` namespace pods.

#. Make sure they are all in a `running` state

   .. code-block:: bash

    kubectl get pods -n istio-system

   .. figure:: images/istioresources.png

This completes your Istio control plane installation.
