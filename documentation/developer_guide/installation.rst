Installation
============

Installing the mF2C System
--------------------------

*For Linux only*

1. download and run the Linux installation script (**docker-compose required**)

.. code-block:: bash

    git clone https://github.com/mF2C/mF2C.git
    cd mF2C/linux
    
    

Installing a Leader
~~~~~~~~~~~~~~~~~~~

**Prerequisites for the discovery module:**

A device with a wireless card that supports "master mode" (i.e. that can act as an access point). You can check whether your card supports master mode by running the following command, looking for the "Supported interface modes". You should find "AP" in the list (i.e. Master mode) :

.. code-block:: bash

    sudo iw list
    
Then, the following script should be run as follows to start the agent with the "leader" role:

.. code-block:: bash

    ./mf2c-deployment.sh --isLeader
    
As far as the discovery module is concerned, this script grabs the name of the wireless interface to be used. It then makes sure the discovery container is run with the --cap-add=NET_ADMIN, since network admin capabilities are needed to access the wireless interface of the host machine. It also programmatically associates the physical wireless interface to the newly created container. Finally, the wireless interface is brought up within the container.
    
Installing a regular agent
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    ./mf2c-deployment.sh




Installing the mF2C Cloud Agent
-------------------------------

1. install Docker, by following the instructions at https://docs.docker.com/install/

2. make sure Docker Compose is also installed (https://docs.docker.com/compose/install/)

3. install `git`:

.. code-block:: bash

    # assuming Ubuntu
    apt-get update
    apt-get install -y git

4. (recommended) use the `/opt` directory as working directory:

.. code-block:: bash

    cd /opt

5. clone the main mF2C repository:

.. code-block:: bash

    git clone https://github.com/mF2C/mF2C

6. go in and choose the right distribution - **docker-compose-cloud**

.. code-block:: bash

    cd mF2C/docker-compose-cloud

7. using the version 3 Compose file in this folder, deploy the mF2C cloud agent core engine: 

.. code-block:: bash

    docker-compose -f docker-compose-core.yml -p mf2c up

8. note that step 7. will only deploy the core services for mF2C. To deploy the remaining services, make sure to add the proper credentials to **.env** and run:

.. code-block:: bash

    docker-compose -f docker-compose-components.yml -p mf2c up


*The full installation might take a few minutes, depending on 
the user's local Docker images and network connection* 



Container Monitoring
~~~~~~~~~~~~~~~~~~~~

To add container monitoring simply run:

.. code-block:: bash

    docker run --volume=/:/rootfs:ro \
        --volume=/var/run:/var/run:rw \
        --volume=/sys:/sys:ro \
        --volume=/var/lib/docker/:/var/lib/docker:ro \
        --volume=/dev/disk/:/dev/disk:ro \
        --publish=8080:8080 --detach=true \
        --name=cadvisor google/cadvisor:latest

**Note** that this monitoring page will be publicly available in port 8080.


Updating Components
-------------------

with docker-compose
~~~~~~~~~~~~~~~~~~~

If the mF2C agent has been installed with Docker Compose, then to update a single component without 
having to re-deploy the full stack, simply run:

.. code-block:: bash

    docker-compose -f <yml_file> -p mf2c up -d <service_name>



Use of the Certificate Authority servers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Relationship to Fog Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three CAs. They are completely separate from the Fog components eg Cimi, Discovery, Lifecycle, UserManagement and exist on a remote server. They interact only via the network. The CAs issue certificates that are critical for the running of the CAU demo.

Requirements
~~~~~~~~~~~~

A host VM with 4GB of memory, 15GB of disk as a minimum and running Centos 7.4 and Docker 18.03.

The VM is hosted on the Tiscali Engineering Openstack.

Scripts need to be present client-side to run the CAU demo.

Expected configuration
~~~~~~~~~~~~~~~~~~~~~~

The following ports need to be opened at the firewall inbound to the CAs.
•       51443 – root CA
•       52443 – Untrust CA
•       53433 – Fog CA
Most firewalls allow unrestricted outbound connections so no ports need be opened client-side.

The firewall ports forward to the containers running the CAs.
[centos@machine38ca0207-da55-46d4-973e-4343f9d28d0b ~]$ sudo firewall-cmd --list-forward
port=51080:proto=tcp:toport=80:toaddr=172.18.0.2
port=52080:proto=tcp:toport=80:toaddr=172.18.0.3
port=53080:proto=tcp:toport=80:toaddr=172.18.0.4
port=53443:proto=tcp:toport=8443:toaddr=172.18.0.4
port=52443:proto=tcp:toport=8443:toaddr=172.18.0.3
port=51443:proto=tcp:toport=8443:toaddr=172.18.0.2
port=51022:proto=tcp:toport=22:toaddr=172.18.0.2
port=52022:proto=tcp:toport=22:toaddr=172.18.0.3
port=53022:proto=tcp:toport=22:toaddr=172.18.0.4

Domain names and DNS
~~~~~~~~~~~~~~~~~~~~

The DNS name for the VM host is it1demo.mf2c-project.eu

The DNS name is registered and published by Tiscali Engineering. Contact Antonio for assistance.

The CA containers have these IP addresses. If the container is restarted the IP addresses might change in which case alter /etc/hosts on all containers and change the firewall rules on the VM host for port forwarding.

172.18.0.2      rootca.it1demo.mf2c-project.eu
172.18.0.3      untrust.it1demo.mf2c-project.eu
172.18.0.4      fog.it1demo.mf2c-project.eu

213.205.14.13           VM host for the containers


