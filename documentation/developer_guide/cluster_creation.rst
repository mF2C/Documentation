Cluster Creation
=====================

Start a Leader
~~~~~~~~~~~~~~~~~~

1. Modify the enviromental variables of the Policies module in the docker-compose


.. code-block:: yaml

     environment:
       - "DEBUG=True"
       - "MF2C=True"
       - "isLeader=True"
       - "leaderIP="
       - "WIFI_DEV_FLAG=<wifi_iface>"


2. Run the wireless network card attach script before starting the mF2C Agent


.. code-block:: bash

    $ sudo ./powerWiFi <wifi_iface>


3. Run the agent

.. code-block:: bash

    $ cd mF2C/docker-compose
    $ git pull
    $ docker-compose -p mF2C up



Start a normal Agent attached to the Leader
~~~~~~~~~~~~~~~~~~


1. Modify the enviromental variables of the Policies module in the docker-compose


.. code-block:: yaml

     environment:
       - "DEBUG=True"
       - "MF2C=True"
       - "isLeader=False"
       - "leaderIP=172.16.1.25"
       - "WIFI_DEV_FLAG=<wifi_iface>"

*Change the leaderIP value with the correct one*

2. Run the wireless network card attach script before starting the mF2C Agent

.. code-block:: bash

    $ sudo ./powerWiFi <wifi_iface>


3. Run the agent

.. code-block:: bash

    $ cd mF2C/docker-compose
    $ git pull
    $ docker-compose -p mF2C up

