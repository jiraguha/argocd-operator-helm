***********
OpenShift 4 
***********

A installation guide for Argo CD Operator (Helm), Argo CD, 
Argo CD CLI and the Guestbook Example in OpenShift 4.x (OCP_, OKD_, CRC_).

.. _OCP: https://www.openshift.com/products/container-platform
.. _OKD: https://www.okd.io
.. _CRC: https://github.com/code-ready/crc

.. include:: ../_static/common_clusteradmin.txt


Prerequisites
=============

.. image:: https://img.shields.io/badge/ocp-≥%204.2-red.svg
   :target: https://www.openshift.com/products/container-platform
   :alt: OpenShift Container Platform
.. image:: https://img.shields.io/badge/okd-≥%204.4.0--beta5-red.svg
   :target: https://www.okd.io
   :alt: OKD - The Community Distribution of Kubernetes
.. image:: https://img.shields.io/badge/crc-≥%201.4.0-red.svg
   :target: https://github.com/code-ready/crc
   :alt: CodeReady Containers

As usual you need to have access to an up and running OpenShift 4.x cluster. 


.. include:: ../_static/common_operator_installation.txt


Operator Lifecycle Manager
--------------------------

This installation method installs the operator using an `OLM Catalog`_.

.. _OLM Catalog: https://quay.io/repository/disposab1e/argocd-operator-helm-registry-ocp

.. image:: https://img.shields.io/badge/git%20clone-https%3A%2F%2Fgithub.com%2Fdisposab1e%2Fargocd--operator--helm.git-9cf.svg
   :target: https://github.com/disposab1e/argocd-operator-helm
   :alt: git clone

.. code-block:: bash

    oc apply -f guides/ocp4/olm/namespace.yaml
    oc apply -f guides/ocp4/olm/catalog-source.yaml
    oc apply -f guides/ocp4/olm/operator-group.yaml
    oc apply -f guides/ocp4/olm/subscription.yaml
    
    oc rollout status -w deployment/argocd-operator-helm -n argocd

Manual
------

You can install the operator from local sources without having to install an OLM Catalog.
This can be helpful if your cluster runs in an isolated environment with no direct internet access.

.. Note:: No channel subscription and automatic operator updates available with this method. 

.. image:: https://img.shields.io/badge/git%20clone-https%3A%2F%2Fgithub.com%2Fdisposab1e%2Fargocd--operator--helm.git-9cf.svg
   :target: https://github.com/disposab1e/argocd-operator-helm
   :alt: git clone

.. code-block:: bash

    oc apply -f guides/ocp4/manual/namespace.yaml
    oc apply -f guides/ocp4/manual/service-account.yaml
    oc apply -f guides/ocp4/manual/role.yaml
    oc apply -f guides/ocp4/manual/role-binding.yaml
    oc apply -f guides/ocp4/manual/crd.yaml
    oc apply -f guides/ocp4/manual/deployment.yaml

    oc rollout status -w deployment/argocd-operator-helm -n argocd

Argo CD Installation
=====================

Install Argo CD server components and Argo CD command line interface.

Argo CD
-------

Install Argo CD with `OpenShift OAuth`_ integration (recommended) 
or with the `Default Authentication`_ provider.

.. Note:: This operator shares all `configuration values`_ from the Argo CD Helm Chart.

OpenShift OAuth
"""""""""""""""

.. image:: https://img.shields.io/badge/git%20clone-https%3A%2F%2Fgithub.com%2Fdisposab1e%2Fargocd--operator--helm.git-9cf.svg
   :target: https://github.com/disposab1e/argocd-operator-helm
   :alt: git clone

.. code-block:: bash
   
    oc apply -f examples/ocp-oauth.yaml -n argocd

    oc get ArgoCDs argocd -n argocd

    oc rollout status -w deployment/argocd-application-controller -n argocd
    oc rollout status -w deployment/argocd-dex-server -n argocd
    oc rollout status -w deployment/argocd-redis -n argocd
    oc rollout status -w deployment/argocd-repo-server -n argocd
    oc rollout status -w deployment/argocd-server -n argocd

Default Authentication
""""""""""""""""""""""

.. image:: https://img.shields.io/badge/git%20clone-https%3A%2F%2Fgithub.com%2Fdisposab1e%2Fargocd--operator--helm.git-9cf.svg
   :target: https://github.com/disposab1e/argocd-operator-helm
   :alt: git clone

.. code-block:: bash
   
    oc apply -f examples/ocp.yaml -n argocd

    oc get ArgoCDs argocd -n argocd

    oc rollout status -w deployment/argocd-application-controller -n argocd
    oc rollout status -w deployment/argocd-dex-server -n argocd
    oc rollout status -w deployment/argocd-redis -n argocd
    oc rollout status -w deployment/argocd-repo-server -n argocd
    oc rollout status -w deployment/argocd-server -n argocd

Argo CD Route
-------------

Argo CD web frontend is immediately accessible through an SSL passthrough route.

.. image:: ../_static/ocp4_route.png

.. image:: https://img.shields.io/badge/Browser-https%3A%2F%2Fargocd--server--argocd.apps--crc.testing%2F-9cf
   :target: https://argocd-server-argocd.apps-crc.testing
   :alt: Point your Browser to

.. image:: ../_static/common_argocd_login_oauth.png

.. Note:: 
   To access this route in ContainerReady Containers (CRC) from the `Argo CD CLI`_ you have to manually add this route to your ``/etc/hosts`` file.


.. code-block:: bash
   
    sudo vi /etc/hosts

    # End of section
    # Added by Docker Desktop
    # To allow the same kube context to work on the host and the container:
    127.0.0.1 kubernetes.docker.internal
    192.168.64.73 api.crc.testing oauth-openshift.apps-crc.testing argocd-server-argocd.apps-crc.testing




Argo CD CLI
-----------

Argo CD provides a command line interface. 
To be in sync with your Argo CD version you can download it directly from 
the Argo CD installation.


Download CLI
""""""""""""

.. code-block:: bash
   
    curl --insecure https://argocd-server-argocd.apps-crc.testing/download/argocd-linux-amd64 -o argocd  



Login Argo CD
"""""""""""""

.. code-block:: bash
   
    argocd login --insecure --username admin --password admin argocd-server-argocd.apps-crc.testing

.. include:: ../_static/common_guestbook_example.txt


Cleanup
=======

`Ants`_ are very clean species too!

.. _Ants: https://en.wikipedia.org/wiki/Ant


Guestbook Example
-----------------

.. code-block:: bash

    argocd app delete guestbook


Argo CD installation
--------------------

The uninstallation process will remove the Argo CD installation (CR) but NOT the CRD's. You have to remove them manually:

.. code-block:: bash
    
    oc delete ArgoCD argocd -n argocd

    oc delete crd appprojects.argoproj.io
    oc delete crd applications.argoproj.io



Operator Marketplace installation
---------------------------------

.. code-block:: bash

    oc delete -f guides/ocp4/olm/subscription.yaml
    oc delete csv argocd-operator-helm.v0.0.7 -n argocd
    oc delete crd argocds.argoproj.io
    oc delete -f guides/ocp4/olm/catalog-source.yaml
    oc delete -f guides/ocp4/olm/operator-group.yaml
    oc delete -f guides/ocp4/olm/namespace.yaml


Manual installation
-------------------

.. code-block:: bash

    oc delete -f guides/ocp4/manual/deployment.yaml
    oc delete -f guides/ocp4/manual/crd.yaml
    oc delete -f guides/ocp4/manual/role-binding.yaml
    oc delete -f guides/ocp4/manual/role.yaml
    oc delete -f guides/ocp4/manual/service-account.yaml
    oc delete -f guides/ocp4/manual/namespace.yaml


.. include:: ../_static/urls.txt
