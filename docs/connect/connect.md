---
title: Installing Grafana in Karbon
---

In this exercise we will install Grafana into the same ``ntnx-system`` namespace using Helm. 

If you haven't got Helm deployed use these [instructions](../appendix/helm.md) to deploy it in your Linux Tools VM.

## Overview 

1.  Connect to Linux Tools VM (if not already connected)
2.  Install Grafana into ``ntnx-system`` namespace

## Connect to your Linux Tools VM 

1.  Logon to your Linux Tools VM console as ``root`` user (default password) and open terminal.

    !!!info
           If you are using your PC/Mac you can also ssh/putty to your Linux Tools VM

    ```bash
    ssh -l root <Linux Tools VM IP address>
    ```
    
## Install Grafana 

1.  Create a values files to instruct Helm chart to use ``default`` service account
    ```bash
    cat << EOF > values.yaml
    serviceAccount:
      create: false
      name: default
    persistence:
      type: pvc
      enabled: true
      size: 10Gi
    service:
      enabled: true
      type: NodePort
    EOF
    ```

2.  Install Grafana using Helm and using ``values.yaml`` file

    ```bash
    helm install grafana/grafana --generate-name --namespace ntnx-system \
    --set persistence.enabled=true,persistence.type=pvc,persistence.size=10Gi \
    --set service.type=NodePort -v values.yaml
    ```

    ![](images/install-graf.png)

    !!!info
            In case there are issues with downloading the pod from Docker hub, follow the instructions [here](../appendix/privatereg.md) to set your service account of choice to use a registry secret.

2.  Now let's get the password for Grafana implementation using which we can logon to Grafana console
    
    ```bash 
    kubectl get secret --namespace grafana grafana-1669965734 \
    -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
    ```
    ```bash title="Output for password - this will be different for you"
    waeIaO6AuKWW2x7aqnOyzCRnU6GIVQRvDLltm2Qr
    ```

3.  Get the Grafana URL to visit by running these commands in the same shell

    ```bash title=""
    export NODE_PORT=$(kubectl get --namespace grafana -o jsonpath="{.spec.ports[0].nodePort}" services grafana-1669965734)
    export NODE_IP=$(kubectl get nodes --namespace grafana -o jsonpath="{.items[0].status.addresses[0].address}")
    echo http://$NODE_IP:$NODE_PORT
    ```
    ```url title="Output - be sure to use your IP addresses"
    http://10.38.9.180:31800
    ```

4.  Login to Grafana in a browser using the access URL and password from
    the previous steps

    ![](images/login-graf.png)

    ![](images/splash-graf.png)

This completes your Grafana installation.

## Configure Grafana 

We will now configure the following in Grafana to visualise the health and status of Karbon kubernetes nodes, resources and some applications.

To be able to set up views, we need to do the following:

-   Setting up a data source for Grafana - Prometheus in our case
-   Test data source for Grafana

Once the data source is configured we will do the following:

-   Configure a custom dashboard
-   Import a Grafana community configured dashboard

## Setting up a Data Source

1.  In Grafana UI, click on Data Sources (Add your first data source)

    ![](images/datasource-graf.png)

2. Select Prometheus

    ![](images/prom-ds.png)

3.  Enter the following in the datasource URL in the URL-field and click
    on **Save and Test**

    ```bash
    http://prometheus-k8s.ntnx-system.svc.cluster.local:9090
    ```

    ![](images/save-prom-ds.png)

    !!!note
           For the datasource URL above; the following are the parts of URL prometheus-k8s 
           - name of your Prometheus service ntnx-system - your namespace svc - your Prometheus service cluster.local - generic DNS name for your kubernetes cluster 9090 - prometheus ClusterIP port
           
           This is the DNS reference of the Prometheus service within your namespace. All services in the namespace are able to resolve by doing a DNS lookup with kube-dns DNS server.

4.  You should see a **Data source is working** message to confirm. If
    this is not working check for typos in the datasource URL.

    ![](images/working-ds.png)

We have now configured a datasource for Grafan. Let's move on to configuring dashboards and visualising metrics in the next section.
