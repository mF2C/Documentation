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

**Prerequisites for the policies module:**

One of the responsibilities of this module is to provide the execution of the strategy for the Leader Failure, by selecting a backup. The static list of devices allowed to become a backup by the user must be provided before executing the agent. You should modify the `.env` file adding the topology like the following line:

.. code-block:: txt

	TOPOLOGY=[(123,'192.168.5.2'),(456,'192.168.5.4')]

Here, you should provide for each allowed backup, an integer to identify internally the agent and the IP address, each device surrounded with parenthesis and the IP address with single quoting. An empty topology forces the Leader to not select any backup.

*This configuration can be applied to a regular agent aswell. If the agent becomes a leader due a leader failure, the provided topology is used to select the new backup*

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
----------------------------------------

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

Refer to the documentation for a list of the firewall ports that need to be opened and the firewall rules to implement the port-forwarding.
 https://repository.atosresearch.eu/owncloud/index.php/apps/files/?dir=%2FmF2C%2FWorking%20Folders%2FWP5%20PoC%20integration%2FCA


Domain names and DNS
~~~~~~~~~~~~~~~~~~~~


The DNS name is registered and published by Tiscali Engineering. Contact Antonio for assistance.

Refer to the documentation at  https://repository.atosresearch.eu/owncloud/index.php/apps/files/?dir=%2FmF2C%2FWorking%20Folders%2FWP5%20PoC%20integration%2FCA for a list of IP addresses and domain names.


Passwords, certificates and ssh keys
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These are listed in the restricted access documentation and zip files located at https://repository.atosresearch.eu/owncloud/index.php/apps/files/?dir=%2FmF2C%2FWorking%20Folders%2FWP5%20PoC%20integration%2FCA

Cau-client component
~~~~~~~~~~~~~~~~~~~~

Description:
This component is a JAVA application.  It supports the Agent Discovery and Authentication process.  It is triggered by the policy block via TCP-IP to kick off the agent authentication process.  It starts by establishing a TLS connection via TCP-IP to the regional CAU to request an Agent certificate.  After successfully obtained the signed certificate, it performs a TLS handshake via TCP-IP with the Leader Agent's CAU to exchange keys to secure future communication.

Installation:
The component is installed by running the mF2C docker-compose.yml.  

Configuration:
The cau-client listens on port 46065 for the policy block trigger.  You can change this by amending the value of the 'expose' instruction in the cau-client block in the docker compose.yml file.  See below:
	
    expose:
      - 46065 #replace this value

You also need to tell cau-client where the regional CAU and the leader agent CAU are located.  This is done by amending the cau-client block in the docker-compose.yml file, providing values to the CAU_URL and LCAU_URL environemnt variables.  For example:

    environment:
      - CAU_URL=10.0.0.129:46400
      - LCAU_URL=10.0.0.129:46410


Use of the SLA Management component
-----------------------------------

The SLA Management is a lightweight implementation of an SLA system, inspired by the WS-Agreement standard. It features (i) a REST interface to manage agreements, (ii) a background agreement assessment. 

An agreement is represented by a simple JSON structure. An example is shown below:

.. code-block:: bash

 {
    "name": "an-agreement-name",
    "details":{
	"id": "2018-000234",
	"type": "agreement",
	"name": "an-agreement-name",
	"provider": { "id": "mf2c", "name": "mF2C Platform" },
	"client": { "id": "a-client", "name": "A client" },
	"creation": "2018-01-16T17:09:45Z",
	"expiration": "2019-01-17T17:09:45Z",
	"guarantees": [
	    {
		"name": "TestGuarantee",
		"constraint": "[execution_time] < 100"
	    }
	]
    }
 }

The LifecycleManager is responsible, on a service instance creation, to generate an agreement and to start its assessment.
At the moment, the agreement generation is not available. For this reason, an agreement must be manually created on CIMI for each
service instance that needs to have an SLA. 

Currently, the assessment only is able to evaluate execution_time metrics, which are retrieved from the service-operation-report
resource. The Distributed Execution Runtime (DER) stores instances of this resource when completing an operation. Any non-DER
service instance can store the appropriate service-operation-report to have its agreement evaluated. For DER service instances,
the guarantee name must match the operation names.

The steps to evaluate an agreement for a service instance are: 

1. Create an sla-agreement CIMI resource using the excerpt above as template. Add as many guarantees as operations you need to 
   observe, and set the guarantee name to the COMPSs name of the operation (qualified class name '.' method name). Take note of 
   the agreement ID auto generated by CIMI.
2. Start the service instance through the Lifecycle Manager passing the agreement ID as parameter. The Lifecycle Manager also
   starts the agreement assessment. Alternatively, you can manually update the agreement field of an existing service instance 
   and update the status field to "started" of the corresponding agreement resource.
3. Once the service is started, instances of the sla-violation resource are created if any guarantee term is not fulfilled.
