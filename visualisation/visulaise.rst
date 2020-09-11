.. _visualise:

.. title:: Visualise Metrics

-------------------------------------
Configuring Dashboards in Grafana
-------------------------------------

In this section we will configure dashboards to visualise the metrics collected by Prometheus.

As discussed in the previous section all Prometheus collected metrics can be queried using PromQL (Prometheus Query Language). In this section we will configure a custom dashboard which uses a PromQL query to get data from Prometheus data source.

Configuring a Custom Dashboard
+++++++++++++++++++++++++++++++

Let’s check the load per cpu in the Karbon kubernetes nodes

#. In Grafana UI; Click on **+** and **Dashboard**

   .. figure:: images/new-db-graf.png

#. Click on **+ Add new panel**

   Query to use with the dashboard:

   .. code-block:: bash

    instance:node_load1_per_cpu:ratio

#. In the **New dashboard / Edit panel window** paste the query from above in the Metrics field

#. In the Panel Settings change the title of the panel to **Load Per CPU**

   .. figure:: images/promql-panel-title.png

#. Click on **Save**

#. Confirm creation of panel and dashboard in the main window

#. **(Optional Step)** - Explore editing the panel, editing X and Y axis, try some aggregate functions by selecting the **Panel** and **Edit** options

   .. figure:: images/panel-edit.png

Configuring a Grafana.com Dashboard
+++++++++++++++++++++++++++++++++++++

Grafana has an open community of dashboard developers who constantly upload dashboard to grafana.com. These can be used used in your environment as well by their identification numbers.

In this section we will configure a grafana.com dashboard.

#. In Grafana; Click on **+** and **Dashboard**

#. Click on **Import**

   .. figure:: images\new-db-graf-import.png

#. Enter ``1860`` in the id text box

#. Click on **Load**

   .. figure:: images/enter-id-graf.png

#. In the next screen, give a name for your dashboard (or keep existing one), select the **Prometheus data source** (that we configured in the previous session) and click on **Import**

   .. figure:: images/custom-id-graf.png

   .. note::

     It may take up to 30 seconds for the dashboard to load as there are quite a few queries and graph rendering that has to happen in the background

   .. figure:: images/node-exporter.png

#. Explore the CPU/Memory/Net/Disk and  (and other areas of interest) in the dashboard.

   .. figure:: images/CPU-Memory-Net-Disk.png

   Here is a screenshot of the **Storage Disk** metrics

   .. figure:: images/Storage-Disk.png

#. Try Grafana.com id `8588` by creating a new dashboard and explore

#. `Here <https://grafana.com/grafana/dashboards?dataSource=prometheus&orderBy=reviewsCount&direction=desc>`_ are few other Grafana.com dashboards that you can import from to have additional visualisation of metrics exposed by your applications which Prometheus collects and stores.

   .. figure:: images/granfana-com.png

You have now completed this section of configuring dashboards in Grafana.
