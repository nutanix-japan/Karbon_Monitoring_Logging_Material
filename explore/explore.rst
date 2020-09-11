.. _explore:

.. title:: Explore Monitoring Resoruces

-------------------------------
Exploring Prometheus in Karbon
-------------------------------

In this section we will explore the ``ntnx-system`` namespace in a given Nutanix Karbon deployed kubernetes cluster and its resources in terms of Prometheus monitoring

Node Exporter
++++++++++++++

Node exporter is used to export metrics from each of the Kubernetes nodes. So a Kubernetes `daemonset <https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/>`_ is used to accomplish this task.

Daemonset makes sure there is an instance of node-exporter po,zd in each node of the cluster as shown below.

.. figure:: images/node-exporter-daemonset.png

In case the node-exporter pod fails, the kubernetes control loop will replace it with a new node-exporter pod at all times.
So we are sure that metrics are being collected and uploaded to the Prometheus metrics store.

Exploring Prometheus
++++++++++++++++++++++++++++

We will now check a few metrics that are collected and exposed by Promethues implementation that comes with Karbon deployed kubernetes clusters

Let's access the Prometheus UI to check for available metrics.

#. Find the Prometheus Service's port number using the following command.

   .. code-block:: bash

    k get svc -n ntnx-system

   .. figure:: images/prom-svc-port.png

#. Now let's forward Prometheus service's ``9090``  port to your Linux Mint VM to be able to access Prometheus UI.

   .. code-block:: bash

    k port-forward svc/prometheus-k8s 9090:9090 &

#. Now you should be able to access Prometheus UI in your Linux Mint VM using the following URL

   .. code-block:: bash

     http://localhost:9090

   .. figure:: images/prom-splash.png

#. Explore available metrics by selecting **insert metric at cursor** drop-down list. Here you will see a list of available metrics.

#. Here I will choose **go_memstats_alloc_bytes** metrics and check if I get any data. Note that the query text-box is now populated with the query **go_memstats_alloc_bytes**

#. Click on **Execute**. You will see the results of memory used by each kubernetes resource (pod) as show in the following figure.

   .. figure:: images/prom-query.png

#. Notice that there is also a **Graph** option in Prometheus GUI for the selected metrics. Click on **Graph** to visualise the chosen metrics.

   .. figure:: images/prom-graph.png

#. You can also type in a keyword in the query window and suggestions for available metrics would show. You can see that typing the keyword ``node`` has brought up all available node based metrics.

   .. figure:: images/prom-nodes-search.png
   
In day to day operations, most of the metrics made available in the default implementation of Prometheus in Nutanix Karbon deployed kubernetes clusters. Highlight this fact to a customer during a design workshop.

Prometheus Operator
++++++++++++++++++++

As we already know Kubernetes has resources such as deployments and daemonsets (not limited to) for managing stateless applications, Operators are kubernetes extensions that will allow for stateful applications to be managed. As stateful applications need more domain specific knowledge of scaling, upgrading and reconfiguring, operators provide a way of doing this.

Operators can be developed individually or obtained from a community maintaining these.

For example: Prometheus as a stateful application has an operator and popular databases like MS SQL do too.

Nutanix Karbon deployed clusters come with a Prometheus operator and it runs as a `replica set <https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/>`_ maintaining at least one operator pod being alive at any given time. We can use this prometheus operator to deploy a separate instance of Prometheus to collect and maintain metrics from other applications in our Karbon deployed Kubernetes cluster.

A good design would advocate the following for Prometheus implementation: You may come across this design question while working with a customer on a Karbon (DevOps) opportunity. Please take a moment to go through this.

1. Use the Nutanix deployed prometheus instance to monitor nodes and system namespaces
2. Deploy separate instances of Prometheus collectors for user applications and user namespaces using the Nutanix deployed prometheus operator

A view of this is illustrated in the following diagram:

.. figure:: images/prom-instance-design.png

**Image Source:** `Prometheus Instances for Kubernetes and Applications <https://miro.medium.com/max/700/1*PK2FdiI5mwLTTOgik0BaJg.png>`_

Nutanix's very own DevOps Architect Christophe Jauffret has written a great article `here <https://medium.com/@christophe_99995/applications-metrics-monitoring-on-nutanix-karbon-c1d1158ebcfc>`_ to deploy a separate instance of Prometheus to monitor user applications over here. You could recommend this method to a prospective customer.

.. note::

 The steps in the Christophe's article above will work in your lab environment. This might take up to 45 minutes. Please only do this if you have extra time in this session.
