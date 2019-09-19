Installation
============

Installing the mF2C System
--------------------------

*For Linux only*

1. download and run the Linux installation script (**docker-compose required**)

.. code-block:: bash

    git clone https://github.com/mF2C/mF2C.git
    cd mF2C/docker-compose
    sudo ./install.sh

To check the status of the mF2C system:

.. code-block:: bash
    sudo ./install.sh -s

To stop the agent:

.. code-block:: bash
    sudo ./install.sh -S
    

Installing a Leader
~~~~~~~~~~~~~~~~~~~


**Prerequisites for the discovery module:**

A device with a wireless card that supports "master mode" (i.e. that can act as an access point). You can check whether your card supports master mode by running the following command, looking for the "Supported interface modes". You should find "AP" in the list (i.e. Master mode) :

.. code-block:: bash

    sudo iw list
    
Then, the installation script should be run as follows to start the agent with the "leader" role:

.. code-block:: bash

    sudo ./install.sh -L
    
As far as the discovery module is concerned, the installation script grabs the name of the wireless interface to be used. It then makes sure the discovery container is run with the --cap-add=NET_ADMIN, since network admin capabilities are needed to access the wireless interface of the host machine. It also programmatically associates the physical wireless interface to the newly created container. Note that Discovery is attached to the host network.

**Prerequisites for the data management module:**

This module is responsible for transparently replicating the necessary data from children to their leader so that the leader has a global view of its cluster. This allows the different components in the leader to forget about data transfers and replicas, and access all the data in the cluster as if it was only in the leader. 

To achieve this behaviour, you should modify the `.env` file adding the IP addresses of this leader's children as follows: 

.. code-block:: txt

	CHILDREN_DC=host1:port1;host2:port2;...;hostn:portn



Installing a regular agent
~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, the installation script will start a normal agent.

.. code-block:: bash

    sudo ./install.sh


**Prerequisites for the data management module:**

This module is responsible for transparently replicating the necessary data from children to their leader so that the leader has a global view of its cluster.  

To achieve this behaviour, you should modify the `.env` file adding the IP address of this agent's leader as follows: 

.. code-block:: txt

	LEADER_DC=host:port
	
	

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



Use of the Certificate Authority server
----------------------------------------

The Certification Authority (https://github.com/mF2C/certauth) is a JAVA Jersey ReST application deployed on a Tomcat container.  It provides 8 different certification CA endpoints.  The CAs are independent components that exist independently to the mF2C Agents and fog clusters as well as the CAU middleware.  The different CAs provide X.509 certificates to uniquely identify mF2C Agents and infrastructure components within mF2C.  The CAU middleware interacts with the unstrusted CA services over HTTPS to request certificates for candidate Agents.  Trusted infrastructure components need to obtain a certificate and the associate RSA private key from the appropriate trusted CA service.

The how-to documentation at  https://github.com/mF2C/certauth/blob/master/src/main/resources/vanilla-ca-howto.pdf provides detailed information on the list and usage of the certification service endpoints.

Requirements
~~~~~~~~~~~~

A host VM with 4GB of memory, 15GB of disk as a minimum and running Centos 7.4 and Docker 18.03.  
The VM is hosted on the Tiscali Engineering Openstack. 

Domain names and DNS
~~~~~~~~~~~~~~~~~~~~

The DNS name is registered and published by Tiscali Engineering. Contact Antonio for assistance.


Installation requirements and procedures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The server is deployed as a Docker container.  Refer to the how-to documentation (https://github.com/mF2C/certauth/blob/master/src/main/resources/vanilla-ca-howto.pdf) on the installation requirements and steps to build and deploy the CA application.  Please note that you need to have access to the Engineering box and the items listed in the following section to deploy the application.


CA certificates, private keys and Tomcat scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These are available from the 'CA credentials' and 'tomcat' folders at https://repository.atosresearch.eu/owncloud/index.php/apps/files/?dir=%2FmF2C%2FWorking%20Folders%2FWP5%20PoC%20integration%2FCA


Cau-client component
~~~~~~~~~~~~~~~~~~~~

Description:
This component is a JAVA application.  It supports the Agent Discovery and Authentication process.  It is triggered by the policy block via TCP-IP to kick off the agent authentication process.  It starts by establishing a TLS connection via TCP-IP to the regional CAU to request an Agent certificate.  After successfully obtained the signed certificate, it performs a TLS handshake via TCP-IP with the Leader Agent's CAU to exchange keys to secure future communication.

Installation:
The component is installed by running the mF2C docker-compose.yml.  

Configuration:
The cau-client listens on port 46065 for the policy block trigger.  This value is fixed for the IT1 demo.  
You also need to tell cau-client where the regional CAU and the leader agent CAU are located.  This is done by amending the cau-client block in the docker-compose.yml file, providing values to the CAU_URL and LCAU_URL environemnt variables.  For example:

    environment:
      - CAU_URL=10.0.0.129:46400
      - LCAU_URL=10.0.0.129:46410


Use of the SLA Management component
-----------------------------------

The SLA Management is a lightweight implementation of an SLA system, inspired by the WS-Agreement standard. It features (i) a REST interface to manage agreements, (ii) a background agreement assessment.

To make use of the SLA Management on IT-1, you must install an agreement in the system. This agreement will be detected by the GUI when creating a service instance and will be associated to it.

An agreement is represented by a simple JSON structure. Below is the default agreement that you should install. This agreement will check that the service operations are executed in less than one second. Modify the constraint to allow different time threshold.

.. code-block:: bash

 {
    "name": "*",
    "details":{
	"id": "2018-000234",
	"type": "agreement",
	"name": "*",
	"provider": { "id": "mf2c", "name": "mF2C Platform" },
	"client": { "id": "a-client", "name": "A client" },
	"creation": "2018-01-16T17:09:45Z",
	"expiration": "2020-01-17T17:09:45Z",
	"guarantees": [
	    {
		"name": "*",
		"constraint": "[execution_time] < 1000"
	    }
	]
    }
 }

To install the agreement, type the following command, where $CIMI_URL is the URL of the CIMI server in the leader agent.

.. code-block:: bash

  curl -X POST -d @agreement.json -H"Content-type:application/json" $CIMI_URL/api/agreement

Advance usage
~~~~~~~~~~~~~

The LifecycleManager is responsible, on a service instance creation, to generate an agreement and to start its assessment.
At the moment, the agreement generation is not available. For this reason, you must install an agreement as explained above, which will be utilized when creating services using the GUI. If you plan to have different SLAs for the different services, an agreement must be manually created on CIMI for each service instance that needs to have an SLA. In this case, you must also modify the fields .name and .details.name of the agreement to match the name of an installed service. Install as many agreements as service kinds you want to observe.

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


Check QoS provider
------------------

Before to check the QoS of a specific service, some previous steps are required.

1. Submit an Agreement:

.. code-block:: bash

    cat >agreement.json <<EOF
    {
    "name": "AGREEMENT 1",
    "state": "started",
    "details":{
        "id": "agreement",
        "type": "agreement",
        "name": "AGREEMENT 1",
        "provider": { "id": "mf2c", "name": "mF2C Platform" },
        "client": { "id": "c02", "name": "A client" },
        "creation": "2018-01-16T17:09:45.01Z",
        "expiration": "2019-01-17T17:09:45.01Z",
        "guarantees": [
              {
                "name": "TestGuarantee",
                "constraint": "execution_time < 10.0"
              }
              ]
      }
  }
    EOF
    curl -XPOST -k https://cimi/api/agreement -d @agreement.json -H "Content-type: application/json" -H 'slipstream-authn-info: super ADMIN'

2. Submit a Service Instance specifying the *<service-id>* and the *<agreement-id>*:

.. code-block:: bash

    cat >service-instance.json <<EOF
    {
    "service" : "service/<service-id>",
    "status" : "not-defined",
    "agreement" : "agreement/<agreement-id>",
    "agents" : [ {
      "agent" : {
        "href" : "agent/default-value"
      },
      "allow" : true,
      "ports" : [ 46100, 46101, 46102, 46103 ],
      "status" : "not-defined",
      "agent_param" : "not-defined",
      "url" : "192.168.252.41",
      "container_id" : "-",
      "master_compss" : true,
      "num_cpus" : 7
    } ],
    "user" : "testuser"
  }
    EOF
    curl -XPOST -k https://cimi/api/service-instance -d @service-instance.json -H "Content-type: application/json" -H 'slipstream-authn-info: super ADMIN'

3. Submit a Service Operation Report specifying the <service-instance-id>:

.. code-block:: bash

    cat >service-operation-report.json <<EOF
    {
      "serviceInstance": {"href": "service-instance/<service-instance-id>"},
      "operation": "TestGuarantee",
      "execution_time": 50.0
  }
    EOF
    curl -XPOST -k https://cimi/api/service-operation-report -d @service-operation-report.json -H "Content-type: application/json" -H 'slipstream-authn-info: super ADMIN'

Finally, check the QoS of a service instance specifying the id:

.. code-block:: bash

    curl -XGET http://service-manager:46200/api/service-management/qos/<service-instance-id>

As a result of the operation, the service instance will be returned.
