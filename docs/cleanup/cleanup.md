---
title: Cleanup and Takeaways
---

## Cleanup

Cleanup port-forwarding for Prometheus and Grafana. Just **ctrl+c** the
commands.

If you have sent these commands to background, you can bring them to
foreground by using

```bash
fg
```

Cleanup your namespace. This will also delete all the resources inside
the namespace.

```bash
# change to ntnx-system to be able to list deployed helm charts in that namespace
k config set-context $(k config current-context) --namespace=ntnx-system
```
```bash
helm list  
NAME NAMESPACE REVISION APP VERSION 
grafana-1597884244  #copy your grafana chart name to use in the next uninstall command
ntnx-system 1 7.1.1
```
```bash
helm uninstall grafana-1597884244 #this chart name will vary in your implementation
```

!!!note
       **Do not** delete Prometheus implementation in **ntnx-system** namespace. Leave it as is.

## Takeaways

We have been through implementation and use of kubernetes monitoring and logging in this bootcamp and we can't help but notice that this process is quite simple.

-   Prometheus is open-source metrics collector
-   Nutanix Karbon deployed kubernetes has a default implementation of Prometheus to monitor the health of kubernetes nodes and appplications
-   It is a good practice to deploy a separate instance of Prometheus to monitor user applications
-   Grafana is a visualisation tool which we could deploy to visualise collected metrics
-   Elastic Stack is open-source logging collector, analyser and visualisation framework
-   Nutanix Karbon deployed kubernetes has a default implementation of Elastic Stack (Elastisearch and Kibana), and Fluent Bit for logs processing
-   Kibana is the log visualisation tool of choice in the Elastic Stack
-   Customers are able to deploy their own instance of Elastic Stack for their applications
-   Kubernetes operators are a easy way of installing, upgrading, life-cycle managing a complex stateful application
