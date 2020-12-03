.. _loggin:

.. title:: Logging in Karbon Kubernetes Clusters


Elastic Logging Kibana
++++++++++++++++++++++++

In this lab, we will run through setting up logging with `ELK Stack <https://www.elastic.co/what-is/elk-stack>`_ in your Karbon deployed kubernetes cluster using helm. If you haven’t setup Helm, use these :ref:`_helm` instructions to deploy it in your Linux Mint VM.

This setup will collect logs from all Karbon deployed kubernetes nodes (Master, ETCD and Workers).

- We will also go through setting up logging for an application in a separate namespace

The high level steps included in this lab are:

- Elastisearch installation - `Elasticsearch <https://www.elastic.co/what-is/elasticsearch>`_ is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured.
- Filebeat installation - `Filebeat <https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html#filebeat-overview>`_ is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- Kibana installation - `Kibana <https://www.elastic.co/what-is/kibana>`_ is an open source frontend application that sits on top of the Elastic Stack, providing search and data visualization capabilities for data indexed in Elasticsearch.

In a production implementation make sure sufficient thought has been put in for design compoents of Elasticsearch components in terms of

- Log retention which directly affects storage requirements - Physical Volumes (PV)- provisioned by Nutanix Volumes
  - In this implementation we will use a 100 GB PV (this is customizable)
- Log rotation (5 days in this implementation)
- Namespace requirements for ELK - Best to have a new namespace an define resource boundaries

Access your Kubernetes Cluster
-------------------------------

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
      k get po -n ntnx-system

   .. figure:: images/klistresources.png

   .. note::

     Nutanix Karbon has automatically provisioned these kubernetes resources so it is ready to use. You have the option to provision additional storage claims, physical volumes, etc by using the Karbon console or using kubectl with YAML files

You can also notice that Prometheus pods are running in the ``ntnx-system``. We will make use of this Prometheus implementation as a data source for Grafana.

Now that you have an understanding of available kubernetes cluster resources, go ahead and install ELK Stack.

Install ELK Stack
------------------

We will install the following to get a working implementation of ELK Stack.

Install Elasticsearch
^^^^^^^^^^^^^^^^^^^^^^

#. Create a new namespace for your ELK stack

   .. code-block:: bash

    alias 'k=kubectl'
    k create ns elk
    #change default namespace to ELK
    k config set-context --current --namespace=elk

#. If you would like to customise the size of PV and container resources, configure a HELM values file

#. Create a file using the content above and call it ``elastic_values.yaml``

   .. code-block:: bash

    cat <<EOF > elastic_values.yaml
    ---
    # Elasticsearch roles that will be applied to this nodeGroup
    # These will be set as environment variables. E.g. node.master=true
    roles:
      master: "true"
      ingest: "true"
      data: "true"

    replicas: 3
    minimumMasterNodes: 1

    # Shrink default JVM heap.
    esJavaOpts: "-Xmx128m -Xms128m"

    # Allocate smaller chunks of memory per pod.
    resources:
      requests:
        cpu: "100m"
        memory: "512M"
      limits:
        cpu: "1000m"
        memory: "512M"

    # Request smaller persistent volumes.
    volumeClaimTemplate:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 30Gi

#. Run the following command to install elasticsearch

   .. code-block:: bash

      helm install elasticsearch elastic/elasticsearch -f elastic_values.yaml

      # You will see output as follows:
      # NAME: elasticsearch
      # LAST DEPLOYED: Wed Dec  2 10:15:16 2020
      # NAMESPACE: elk
      # STATUS: deployed
      # REVISION: 1
      # NOTES:
      # 1. Watch all cluster members come up.
      #   $ k get pods --namespace=elk -l app=elasticsearch-master -w
      # 2. Test cluster health using Helm test.
      #   $ helm test elasticsearch

#. Wait for the command to execute and check logs to make sure all your elasticsearch resrouces are running

   .. code-block:: bash

    # to check events
    k get events

    # to check all pods and other services are running
    k get all

    # You will see output as follows:
    # NAME                             READY   STATUS    RESTARTS   AGE
    # elasticsearch-master-0           1/1     Running   0          155m
    # elasticsearch-master-1           1/1     Running   0          155m
    # elasticsearch-master-2           1/1     Running   0          155m

    # NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
    # service/elasticsearch-master            ClusterIP   172.19.171.221   <none>        9200/TCP,9300/TCP   156m
    # service/elasticsearch-master-headless   ClusterIP   None             <none>        9200/TCP,9300/TCP   156m
    #
    # NAME                                    READY   AGE
    # statefulset.apps/elasticsearch-master   3/3     156m

#. Check the Physical Volumes to get an understanding of what is provisioned to to support Elasticsearch and its storage requirements - here it is 30 GB in capacity. This can be modified in the HELM values file.

   .. code-block:: bash
    # Check the Physical Volumes created to support Elasticsearch and its storage requirements

    k get pv

    # There will be three to support the three pods in the StatefulSet
    # Note the binding status in the output

    # NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                            STORAGECLASS           REASON   AGE
    # pvc-04302b11-a6e0-459c-8b74-0978f392df07   30Gi       RWO            Delete           Bound    elk/elasticsearch-master-elasticsearch-master-2                  default-storageclass            161m
    # pvc-141cc537-250d-472e-b686-c7dfafabf29a   30Gi       RWO            Delete           Bound    elk/elasticsearch-master-elasticsearch-master-1                  default-storageclass            161m
    # pvc-c8aad9f5-f24c-4e2e-917e-55107e072114   30Gi       RWO            Delete           Bound    elk/elasticsearch-master-elasticsearch-master-0                  default-storageclass

#. Check the Physical Volumes Claims to get an understanding of what is provisioned to to support Elasticsearch and its storage requirements

   .. code-block:: bash

    k get pvc

    # There will be three to support the three volumes - one for each pod and PV

    # NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
    # elasticsearch-master-elasticsearch-master-0   Bound    pvc-c8aad9f5-f24c-4e2e-917e-55107e072114   30Gi       RWO            default-storageclass   162m
    # elasticsearch-master-elasticsearch-master-1   Bound    pvc-141cc537-250d-472e-b686-c7dfafabf29a   30Gi       RWO            default-storageclass   162m
    # elasticsearch-master-elasticsearch-master-2   Bound    pvc-04302b11-a6e0-459c-8b74-0978f392df07   30Gi       RWO            default-storageclass   162m

    # Check all the events to make sure there are no klistresources

    k get events

#. We have now installed Elasticsearch

Install Filebeat
^^^^^^^^^^^^^^^^^^^^^^

#. Configure a values file using the following commands: this is required to satisfy Karbon kubernetes cluster and volume mount requirements

   .. code-block:: bash

     cat <<EOF > filebeat_values.yaml
     ---
     extraVolumeMounts:
        - name: varnutanix
          mountPath: /var/nutanix
          readOnly: true
     extraVolumes:
        - name: varnutanix
          hostPath:
           path: /var/nutanix
     EOF

#. Run the following command to install filebeat

   .. code-block:: bash

      helm install elasticsearch elastic/filebeat -f filebeat_values.yaml

      # You will see output as follows:

      # NAME: filebeat
      # LAST DEPLOYED: Wed Dec  2 10:45:24 2020
      # NAMESPACE: elk
      # STATUS: deployed
      # REVISION: 1
      # TEST SUITE: None

      # Note that the filebeat is deployed as a DaemonSet (one on each worker node)

      k get all

      # NAME                      READY   STATUS              RESTARTS   AGE

      # filebeat-filebeat-m6hf4   1/1     Running             0          26s
      # filebeat-filebeat-72b79   1/1     Running             0          26s

      # NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
      # daemonset.apps/filebeat-filebeat   2         2         2       2            2           <none>          3h4m

#. We have now installed Filebeat and it will start collecting logs from all Karbon deployed kubenetes nodes

#. To verify if Filebeat is setup properly to receive logs from kubernetes nodes and containers, check the data ingestion stats of Elastisearch ClusterIP service

#. Port-foward Elasticsearch Services IP to your local machine

   .. code-block:: bash

     k port-forward service/elasticsearch-master 9200:9200 &

#. Run curl command to see the data indices ingestion details

   .. code-block:: bash

     curl -l "localhost:9200/_cat/indices?pretty&s=i"

     # The output looks as follows and data ingest details are in the last two columns

     # Observe the filebeat line

     # green open filebeat-7.10.0-2020.12.02-000001 ufD341lKTwin_jpknbOIyA 1 1 2089328   0     1gb 529.1mb

#. This confirms that we are ingesting data into Elasticsearch using filebeat


Install Kibana
^^^^^^^^^^^^^^^

#. Run the following command to install Kibana visualisation GUI

   .. code-block:: bash

    $ helm install kibana elastic/kibana

    # You will see the following output

    # NAME: kibana
    # LAST DEPLOYED: Wed Dec  2 10:47:10 2020
    # NAMESPACE: elk
    # STATUS: deployed
    # REVISION: 1
    # TEST SUITE: None

#. Now that we have installed all three necessary service in ELK stack, let us confirm that they are all ready and running.

   .. code-block:: #!/usr/bin/env bash

    k get all

    # Note all the pods, services and other resources for ELK Stack
    # Make sure they are all running without issues

  	# NAME                                 READY   STATUS    RESTARTS   AGE
    # pod/elasticsearch-master-0           1/1     Running   0          3h34m
    # pod/elasticsearch-master-1           1/1     Running   0          3h34m
    # pod/elasticsearch-master-2           1/1     Running   0          3h34m
    # pod/filebeat-filebeat-72b79          1/1     Running   0          3h4m
    # pod/filebeat-filebeat-m6hf4          1/1     Running   0          3h4m
    # pod/kibana-kibana-5b4c966bc9-z65s5   1/1     Running   0          3h2m
    #
    # NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
    # service/elasticsearch-master            ClusterIP   172.19.171.221   <none>        9200/TCP,9300/TCP   3h34m
    # service/elasticsearch-master-headless   ClusterIP   None             <none>        9200/TCP,9300/TCP   3h34m
    # service/kibana-kibana                   ClusterIP   172.19.118.48    <none>        5601/TCP            3h2m
    #
    # NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    # daemonset.apps/filebeat-filebeat   2         2         2       2            2           <none>          3h4m
    #
    # NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
    # deployment.apps/kibana-kibana   1/1     1            1           3h2m
    #
    # NAME                                       DESIRED   CURRENT   READY   AGE
    # replicaset.apps/kibana-kibana-5b4c966bc9   1         1         1       3h2m
    #
    # NAME                                    READY   AGE
    # statefulset.apps/elasticsearch-master   2/3     3h34m

Accessing Kibana GUI
^^^^^^^^^^^^^^^^^^^^

It is now time to visualise our work and logs.

Note that the Kibana service is of type ``Cluster IP``. Since we don't have a LoadBalancer in our environment, we need port-forward the ClusterIP service to our workstation LinuxMintVM.

#. Run the following command to port forward Kibana service. You are able to find the port number by listing Kibana service.

   .. code-block:: bash

     k get svc/kibana-kibana

     # service/kibana-kibana                   ClusterIP   172.19.118.48    <none>        5601/TCP            3h2m
     # Here the service port number is ``5601``

     k port-forward svc/kibana-kibana 5601:5601 &

#. Open a browser window on the workstation and access the following URL

   ``http://localhost:5601``

#. You will see Kibana GUI

   .. figure:: images/kibana-splash.png

#. Click on the menu and select **Observability > Overview**

   .. figure:: images/kibana-menu.png

#. On the Overview page, you can see the log rates per minute. This gives you an overview of log production and injection rates.

   .. figure:: images/kibana-logsperminute.png

#. Click on the menu and select **Observability > Logs**

#. You will be able to see logs streaming from various sources in your kubernetes clusters

   .. figure:: images/kibana-logs.png

#. You are able to use keyword search and also perform highligthing of text

   .. figure:: images/kibana-logs-search.png

#. Experiment with **Stream Live** and other options

#. You have now successfully setup ELK stack and are able to view logs in Kibana

Cleanup
^^^^^^^^

Run the following commands to cleanup your ELK Stack implementation.

.. code-block:: bash

  helm uninstall kibana
  helm uninstall filebeat
  helm uninstall Elasticsearch

To cleanup physical volumes configured as a part of this lab. You can use the following commands:

.. code-block:: bash

  # Change default namespace to ELK (just to make sure)

  k config set-context --current --namespace=elk
  k get pvc -n elk

  # Get the names of pvc for elasticsearch master statefulsets
  # NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
  # elasticsearch-master-elasticsearch-master-0   Bound    pvc-baadcba4-5f26-44e0-93f6-e93dab7a5b82   30Gi       RWO            default-storageclass   59m
  # elasticsearch-master-elasticsearch-master-1   Bound    pvc-0ee313b4-cd3d-4d55-84cc-53225af92da5   30Gi       RWO            default-storageclass   59m

  k delete pvc <pvc-NAME>

  # Be careful not to delete any other pvc
  # It may take a while so please be patient
  # You can specify grace time out period to be   (this is ok in the lab enviroment)
  # k delete pvc <pvc-NAME> --force --grace-period=0

  # Example:
  # k delete pvc elasticsearch-master-elasticsearch-master-0
  # k delete pvc elasticsearch-master-elasticsearch-master-1

  # There is no requirement to delete PV as it will be automatically deleted as defined in the StorageClass settings.

Takeaways
^^^^^^^^^^
- ELK Stack is open-source logging mechanism which can be easily implemented in a kubernete environment
- ELK Stack is easily configurable for customer's requirements
- Design aspect is important in planning resource and retention requirement for logs
- Open-source software like ELK Stack for logging management have limited support but are used widely. Advise the customer of support limitations in using these software
