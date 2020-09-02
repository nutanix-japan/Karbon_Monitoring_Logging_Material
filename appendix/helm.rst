.. _helm:

.. title:: Helm Introduction

-------------------------------
What is Helm?
-------------------------------

Helm is a package manager for Kubernetes based applications and resources deployment.

Helm also provides an easy way of accomplishing the following:

- Install and uninstall kubernetes applications
- Versioning complex kubernetes applications
- Upgrade and rollback kubernetes applications

You can think of Helm as ``yum`` or ``apt-get`` package managers in Linux.

Helm has the following main components (not limited to):

- Helm command line tool - provides interface to all Helm functionality
- Charts - charts are manifests (yaml) files for desired kubernetes resources
- Value file(s) - all customisable values for the application deployment can be configured here

.. note::

  Helm used to be a server-client application. Helm being the client and Tiller server component being installed in a kubernetes cluster. With Helm 3.x onwards, Tiller is removed and only Helm is used for ease.

-------------------------------
Installing Helm 3.x
-------------------------------

Use the following commands to install helm in your Linux Mint VM.

.. code-block:: bash

 curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
 sudo apt-get install apt-transport-https --yes
 echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
 sudo apt-get update
 sudo apt-get install helm
 helm version
