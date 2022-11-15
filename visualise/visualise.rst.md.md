.. \_visualise:

.. title:: Visualise Metrics

  -------------------------------
  Visualising Metrics in Karbon
  -------------------------------

In this section we will configure dashboards to visualise the metrics
collected by Prometheus.

As discussed in the previous section all Prometheus collected metrics
can be queried using PromQL (Prometheus Query Language). In this section
we will configure a custom dashboard which uses a PromQL query to get
data from Prometheus data source.

Configuring a Custom Dashboard +++++++++++++++++++++++++++++++

Let's check the load per cpu in the Karbon kubernetes nodes

1.  In Grafana UI; Click on **+** and **Dashboard**

    .. figure:: images/new-db-graf.png

2.  Click on **+ Add new panel**

    Query to use with the dashboard:

    .. code-block:: bash

    instance:node_load1_per_cpu:ratio

3.  In the **New dashboard / Edit panel window** paste the query from
    above in the Metrics field

4.  In the Panel Settings change the title of the panel to **Load Per
    CPU**

    .. figure:: images/promql-panel-title.png

5.  Click on **Save**

6.  Confirm creation of panel and dashboard in the main window

7.  **(Optional Step)** - Explore editing the panel, editing X and Y
    axis, try some aggregate functions by selecting the **Panel** and
    **Edit** options

    .. figure:: images/panel-edit.png

Configuring a Grafana.com Dashboard
+++++++++++++++++++++++++++++++++++++

Grafana has an open community of dashboard developers who constantly
upload dashboard to grafana.com. These can be used used in your
environment as well by their identification numbers.

In this section we will configure a grafana.com dashboard.

1.  In Grafana; Click on **+** and **Dashboard**

2.  Click on **Import**

    .. figure:: images`\new`{=tex}-db-graf-import.png

3.  Enter `1860` in the id text box

4.  Click on **Load**

    .. figure:: images/enter-id-graf.png

5.  In the next screen, give a name for your dashboard (or keep existing
    one), select the **Prometheus data source** (that we configured in
    the previous session) and click on **Import**

    .. figure:: images/custom-id-graf.png

    .. note::

    It may take up to 30 seconds for the dashboard to load as there are
    quite a few queries and graph rendering that has to happen in the
    background

    .. figure:: images/node-exporter.png

6.  Explore the CPU/Memory/Net/Disk and (and other areas of interest) in
    the dashboard.

    .. figure:: images/CPU-Memory-Net-Disk.png

    Here is a screenshot of the **Storage Disk** metrics

    .. figure:: images/Storage-Disk.png

7.  Try Grafana.com id `8588` by creating a new dashboard and explore

8.  `Here <https://grafana.com/grafana/dashboards?dataSource=prometheus&orderBy=reviewsCount&direction=desc>`\_
    are few other Grafana.com dashboards that you can import from to
    have additional visualisation of metrics exposed by your
    applications which Prometheus collects and stores.

    .. figure:: images/granfana-com.png

You have now completed this section of configuring dashboards in
Grafana.
