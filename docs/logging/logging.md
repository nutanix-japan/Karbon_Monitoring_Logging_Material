---
title:: Logging in Karbon Kubernetes Clusters
---

# Elastic Logging Kibana

In this lab, we will run through setting up logging with
[ELK Stack](https://www.elastic.co/what-is/elk-stack) in your Karbon deployed kubernetes cluster using Helm. If you haven't setup Helm, use these Helm [instructions](../appendix/helm.md) to deploy it in your Linux Tools VM.

This setup will collect logs from all applications deployed in Karbon kubernetes cluster.

!!!info
        All logs for kubernetes nodes (Master, ETCD and Workers) are collected by a separate instance of Elastisearch in the `ntnx-system` namespace. This is deployed by default in all Karbon kubernetes clusters.

The high level steps included in this lab are:

-   Elasticsearch installation -
    [Elasticsearch](https://www.elastic.co/what-is/elasticsearch) is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured.
-   Filebeat [installation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html#filebeat-overview)
    is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
-   Kibana installation - [Kibana](https://www.elastic.co/what-is/kibana) is an open source frontend application that sits on top of the Elastic Stack, providing search and data visualization capabilities for data indexed in Elasticsearch.

In a production implementation make sure sufficient thought has been put in for design components of Elasticsearch in terms of:

-   Log retention which directly affects storage requirements 
    - Physical Volumes (PV) - provisioned by Nutanix Volumes
    - In this implementation we will use a 30 GB PV (this is also customizable)
-   Log rotation
-   Namespace requirements for ELK - it is best to have a separate namespace for logging implementation and define resource boundaries

## Connect to your Linux Tools VM 

1.  Logon to your Linux Tools VM console as ``root`` user (default password) and open terminal.

    !!!info
           If you are using your PC/Mac you can also putty/ssh to your Linux Tools VM

    ```bash
    ssh -l root <Linux Tools VM IP address>
    ```
## Access your Karbon Kubernetes Cluster

1.  Logon to your Prism Central ``https://<PC VM IP>:9440``

    !!!note
            If you haven't got a Karbon deployed kubernetes cluster in your HPOC, refer [here](../appendix/create_kube.md) to create a kubernetes cluster in Nutanix.

2.  Go to **Menu > Services > Karbon**

    ![](images/choosekarbon.png)

3.  Select your Karbon cluster

4.  Click on Actions > Download Kubeconfig

    ![](images/selectcluster.png)

5.  Click on **Copy the command to clipboard**

6.  Paste the contents in your Linux Tools VM shell

7.  Run the following command to verify your connectivity and display
    the nodes in the cluster

    ```bash
    alias 'k=kubectl'
    k get nodes -o wide
    ```

    ![](images/nodelist.png)

8.  You can list the namespaces, storage claims, physical volumes and
    physical volume claims using the following commands

    ```bash
    k get ns 
    k get sc,pv,pvc 
    k get po -n ntnx-system
    ```
    ![](images/klistresources.png)

    !!!info
            Nutanix Karbon has automatically provisioned these kubernetes
            resources so it is ready to use. You have the option to provision
            additional storage claims, physical volumes, etc by using the Karbon
            console or using kubectl with YAML files

## Install ELK Stack

We will install the following to get a working implementation of ELK Stack.
### Install Elasticsearch

1.  Create a new namespace for your ELK stack

    ```bash
    alias 'k=kubectl' 
    k create ns elk #change default namespace to ELK k
    k config set-context --current --namespace=elk
    ```

2.  If you would like to customise the size of PV and container
    resources, configure a HELM values file

3.  Create a file using the content above and call it
    `elastic_values.yaml`

    ```bash
    cat <<EOF > elastic_values.yaml
    ---
    # Elasticsearch roles that will be applied to this nodeGroup
    # These will be set as environment variables. E.g. node.master=true
    roles:
      - master
      - data
      - ingest
    
    replicas: 1
    minimumMasterNodes: 1
    clusterHealthCheckParams: 'wait_for_status=yellow&timeout=1s'

    # Set to use default service account

    rbac:
      create: false
      serviceAccountName: "default"
      automountToken: true

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
    EOF
    ```

4.  Run the following command to install elasticsearch

    ```bash
    helm install elasticsearch elastic/elasticsearch -f elastic_values.yaml
    ```
    ```text title="You will see output as follows:"
    helm install elasticsearch elastic/elasticsearch -f elastic_values.yaml
    NOTES:
    1. Watch all cluster members come up.
    $ kubectl get pods --namespace=elk -l app=elasticsearch-master -w
    2. Retrieve elastic user's password.
    $ kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
    3. Test cluster health using Helm test.
    $ helm --namespace=elk test elasticsearch
    ```
    ```bash title="Check if pods are coming up"
    helm install elasticsearch elastic/elasticsearch -f elastic_values.yaml
    ```
    ```bash title="Retrieve elastic user's password"
    k get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
    ```
    ```bash title="Test cluster health using Helm test"
    helm --namespace=elk test elasticsearch
    ```

    !!!tip
            In case there are issues with downloading the pod from Docker hub, follow the instructions [here](../appendix/privatereg.md) to set your service account of choice to use a Docker registry secret containing your Docker public hub credentials.

5.  Wait for the command to execute and check logs to make sure all your elasticsearch resources are running

    ```bash title="Check events to make sure there are no errors"
    k get events
    ```
    ```bash title="Check all pods and other services are running"
    k get all
    ```
    ```bash title="Output"
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
    ```

6.  Check the Physical Volumes to get an understanding of what is provisioned to to support Elasticsearch and its storage requirements - here it is 30 GB in capacity. This can be modified in the HELM values file.

    ```bash title="Check PVC resources"
    k get pvc
    ```
    ```bash title="There will be three to support the three volumes - one for each pod and PV"
    # NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
    # elasticsearch-master-elasticsearch-master-0   Bound    pvc-c8aad9f5-f24c-4e2e-917e-55107e072114   30Gi       RWO            default-storageclass   162m
    # elasticsearch-master-elasticsearch-master-1   Bound    pvc-141cc537-250d-472e-b686-c7dfafabf29a   30Gi       RWO            default-storageclass   162m
    # elasticsearch-master-elasticsearch-master-2   Bound    pvc-04302b11-a6e0-459c-8b74-0978f392df07   30Gi       RWO            default-storageclass   162m
    ```
    ```bash title="Check all the events to make sure there are no errors"
    k get events
    ```

8.  We have now installed Elasticsearch
### Install Filebeat

1.  Configure a values file using the following commands: this is
    required to satisfy Karbon kubernetes cluster and volume mount
    requirements

    ```bash
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
    ```

2.  Run the following command to install filebeat

    ```bash
    helm install filebeat elastic/filebeat -f filebeat_values.yaml
    ```
    ```bash title="Output"
    NAME: filebeat
    LAST DEPLOYED: Fri Dec  2 22:28:14 2022
    NAMESPACE: elk
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    1. Watch all containers come up.
    $ kubectl get pods --namespace=elk -l app=filebeat-filebeat -w
    ```
    ```bash
    k get pods --namespace=elk -l app=filebeat-filebeat -w
    ```
    ```bash title="Output - filebeat is deployed as a DaemonSet (one on each worker node)"
    # NAME                      READY   STATUS              RESTARTS   AGE

    # filebeat-filebeat-m6hf4   1/1     Running             0          26s
    # filebeat-filebeat-72b79   1/1     Running             0          26s

    # NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    # daemonset.apps/filebeat-filebeat   2         2         2       2            2           <none>          3h4m
    ```

3.  We have now installed Filebeat and it will start collecting logs from all Karbon deployed kubenetes nodes

4.  To verify if Filebeat is setup properly to receive logs from
     kubernetes nodes and containers, check the data ingestion stats of
    Elastisearch ClusterIP service

5.  Port-foward Elasticsearch Services IP to your local machine

    ```bash
    k port-forward service/elasticsearch-master 9200:9200 &
    ```

6.  Run curl command to see the data indices ingestion details

    ```bash
    curl -l "localhost:9200/_cat/indices?pretty&s=i"
    ```
    ```bash
    # The output looks as follows and data ingest details are in the last two columns

    # Observe the filebeat line

    # green open filebeat-7.10.0-2020.12.02-000001 ufD341lKTwin_jpknbOIyA 1 1 2089328   0     1gb 529.1mb
    ```

7.  This confirms that we are ingesting data into Elasticsearch using filebeat

### Install Kibana

1.  Run the following command to install Kibana visualisation GUI

    ```bash
    helm install kibana elastic/kibana
    ```
    ```bash
    # You will see the following output
    # NAME: kibana
    # LAST DEPLOYED: Wed Dec  2 10:47:10 2020
    # NAMESPACE: elk
    # STATUS: deployed
    # REVISION: 1
    # TEST SUITE: None
    ```

2.  Now that we have installed all three necessary service in ELK stack,
    let us confirm that they are all ready and running.

    ```bash
    k get all
    ```
    
## Accessing Kibana GUI

It is now time to visualise our work and logs.

Note that the Kibana service is of type `Cluster IP`. Since we don't
have a LoadBalancer in our environment, we need port-forward the
ClusterIP service to our workstation LinuxMintVM.

1.  Run the following command to port forward Kibana service. You are
    able to find the port number by listing Kibana service.

    ```bash
    k get svc/kibana-kibana
    ```
    ```bash title="Find the port number of the Kibana service"
    # output
    # service/kibana-kibana                   ClusterIP   172.19.118.48    <none>        5601/TCP            3h2m

    # Here the service port number is ``5601``
    ```
    ```bash title="Forward the port to your local machine"
    k port-forward svc/kibana-kibana 5601:5601 &
    ```

2.  Open a browser window on the workstation and access the following URL

    ```url
    http://localhost:5601
    ```

3.  You will see Kibana GUI

    ![](images/kibana-splash.png)

4.  Click on the menu and select **Observability > Overview**

    ![](images/kibana-menu.png)

5.  On the Overview page, you can see the log rates per minute. This
    gives you an overview of log production and injection rates.

    ![](images/kibana-logsperminute.png)

6.  Click on the menu and select **Observability > Logs**

7.  You will be able to see logs streaming from various sources in your
    kubernetes clusters

    ![](images/kibana-logs.png)

8.  You are able to use keyword search and also perform highligthing of
    text

    ![](images/kibana-logs-search.png)

9.  Experiment with **Stream Live** and other options

10. You have now successfully setup ELK stack and are able to view logs in Kibana

## Cleanup

Run the following commands to cleanup your ELK Stack implementation.

```bash
helm uninstall kibana 
helm uninstall filebeat 
helm uninstall Elasticsearch
```

To cleanup physical volumes configured as a part of this lab. You can
use the following commands:

```bash
# Change default namespace to ELK (just to make sure)
k config set-context --current --namespace=elk
```
```bash
k get pvc -n elk
# Get the names of pvc for elasticsearch master statefulsets
# NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
# elasticsearch-master-elasticsearch-master-0   Bound    pvc-baadcba4-5f26-44e0-93f6-e93dab7a5b82   30Gi       RWO            default-storageclass   59m
# elasticsearch-master-elasticsearch-master-1   Bound    pvc-0ee313b4-cd3d-4d55-84cc-53225af92da5   30Gi       RWO            default-storageclass   59m
```
```bash
k delete pvc <pvc-NAME>

# Be careful not to delete any other pvc
# It may take a while so please be patient
# You can specify grace time out period to be   (this is ok in the lab enviroment)
# k delete pvc <pvc-NAME> --force --grace-period=0

# Example:
# k delete pvc elasticsearch-master-elasticsearch-master-0
# k delete pvc elasticsearch-master-elasticsearch-master-1

# There is no requirement to delete PV as it will be automatically deleted as defined in the StorageClass settings.
```

## Takeaways    

-   ELK Stack is open-source logging mechanism which can be easily
    implemented in a kubernete environment
-   ELK Stack is easily configurable for customer's requirements
-   Design aspect is important in planning resource and retention
    requirement for logs
-   Open-source software like ELK Stack for logging management have
    limited support but are used widely. Advise the customer of support
    limitations in using these software
