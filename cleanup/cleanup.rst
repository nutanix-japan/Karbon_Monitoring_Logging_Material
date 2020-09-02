.. _cleanup:

.. title:: Cleaning up your namespace and Istio installation

--------
Cleanup
--------

Cleanup port-forwarding for kiali and product-page(of your application). Just `ctrl+c` the commands.

If you have sent these commands to background, you can bring them to foreground by using

.. code-block:: bash

 fg

Cleanup your namespace. This will also delete all the resources inside the namespace.

.. code-block:: bash

 k delete ns -n xyz-ns

Cleanup Istio

.. code-block:: bash

 k delete ns -n istio-system


----------------
Takeaways
----------------

We have been through implementation and use of Istio service mesh in this bootcamp and we can't help but notice that
this process is quite simple.

- Istio is opensource software and it can be easily deployed in Nutanix Karbon created kubernetes clusters
- Istio has a control and data plane - all service mesh functionality is implemented by extending kubernetes resources with CRD(s)
- Istio support announcements can be found `here <https://istio.io/latest/news/support/>`_
- Applications do not need to Istio-aware leaving developers to focus on the application functionality and quality
- Istio can be used to have granular control of you application in terms of Security, Monitoring and Traffic management
