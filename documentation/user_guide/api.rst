API
===

The mF2C system is equipped with a RESTful (HTTP) API for the management of infrastructure resources.
This API is based on the Cloud Infrastructure Management Interface (CIMI_) specification from DMTF.

The CIMI standard defined patterns for all the usual database actions: Search (or Query), Create, Read, Update, and Delete (SCRUD).

+---------------+-------------+---------------------+
| Action        | HTTP Method | Target              |
+===============+=============+=====================+
| Search        | GET or PUT  | resource collection |
+---------------+-------------+---------------------+
| Add (create)  | POST        | resource collection |
+---------------+-------------+---------------------+
| Read          | GET         | resource            |
+---------------+-------------+---------------------+
| Edit (update) | PUT         | resource            |
+---------------+-------------+---------------------+
| Delete        | DELETE      | resource            |
+---------------+-------------+---------------------+


The mF2C API implementation has been adopted from the open source implementation used in SlipStream_.


**At the moment of writing this documentation, other mF2C RESTful APIs are also being exposed to the users,
to facilitate the ongoing developments.**

CIMI
----

CIMI in the entry point for all users and internal components to interact with the mF2C system resources.
For the sake of simplicity, let's assume CIMI is running at *https://cimi*.

Entry Point
~~~~~~~~~~~

To allow the self-discovery of the existing system resources and allowed operations and requests, CIMI
provides a public entry-point at *https://cimi/api/cloud-entry-point*.

Create a user
~~~~~~~~~~~~~

1. create a regular user *testuser* with password *testpassword*

.. code-block:: bash

    cat >addRegularUser.json <<EOF
    {
        "userTemplate": {
            "href": "user-template/self-registration",
            "password": "testpassword",
            "passwordRepeat" : "testpassword",
            "emailAddress": "your_email@",
            "username": "testuser"
        }
    }
    EOF
    curl -XPOST -k -H "Content-type: application/json" https://cimi/api/user -d @addRegularUser.json


2. an email will be sent to you (if running in test mode, then **you might have to check your SPAM folder**). Copy the API address in that email, and paste it in your browser. Ex: *https://cimi/api/CALLBACK_ENDPOINT*.

3. login as *testuser*

.. code-block:: bash

    cat >regularUser.json <<EOF
    {
        "sessionTemplate": {
            "href": "session-template/internal",
            "username": "testuser",
            "password": "testpassword"
        }
    }
    EOF
    curl -XPOST https://cimi/api/session -d @regularUser.json -H 'content-type: application/json' --cookie-jar ~/cookies -b ~/cookies -sS # use -k if running in test mode


Get an existing resource collection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's say we want to get a list of all the events registered in the database (the ones we have access to, and assuming the CIMI resource *events* exists):

.. code-block:: bash

    curl -XGET https://cimi/api/event --cookie-jar ~/cookies -b ~/cookies -sS


Filter for a specific dataset
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CIMI provides mechanisms to search for data based on a filter. To provided grammar looks like the following:

.. code-block:: bash

    # note that <***> must be replaced
    curl -XGET 'https://cimi/api/event?$filter=<AttrName>="<Value>"&$filter=<AttrName2><=<Value2>&$orderby=<AttrName3>:desc' --cookie-jar ~/cookies -b ~/cookies -sS


Delete a specific resource
~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's delete our own session:

.. code-block:: bash

    curl -XDELETE https://cimi/api/session/<SessionID> --cookie-jar ~/cookies -b ~/cookies -sS




.. _CIMI: https://www.dmtf.org/sites/default/files/standards/documents/DSP0263_2.0.0.pdf
.. _SlipStream: http://ssapi.sixsq.com/#cimi-api


Service Manager
---------------
The Service Manager is an internal component of the mF2C system that will not be exposed. However, in the current development state and for testing purposes, it is accessible through *http://service-manager:46200*.

Service registration
~~~~~~~~~~~~~~~~~~~~
For any workflow to work, the first step is to submit a service to the service manager:

.. code-block:: bash

    cat >service.json <<EOF
    {
	  "name": "hello-world",
	  "description": "Hello World Service",
	  "resourceURI": "/hello-world",
	  "exec": "hello-world",
	  "exec_type": "docker",
	  "exec_ports": [8080, 8081],
	  "category": {
	    "cpu": "low",
	    "memory": "low",
	    "storage": "low",
	    "disk": "low",
	    "network": "low",
	    "inclinometer": false,
	    "temperature": false,
	    "jammer": false,
	    "location": false,
	    "battery_level": true,
	    "door_sensor": true,
	    "pump_sensor": true,
	    "accelerometer": true,
	    "humidity": true,
	    "air_pressure": true,
	    "ir_motion": true
	  }
	}
    EOF
    curl -XPOST -k http://service-manager:46200/api/service-management/categorizer -d @service.json -H "Content-type: application/json"

Lifecycle Management module
-----------------------------
The Lifecycle Management component is part of the Platform Manager's Service Orchestration module. It is an internal component responsible for managing the services running in the mF2C clusters. For IT-2 this component will be accessible through *http://lifecycle:46000/api/v2*.

Services are based on docker images. Thus, when a service is deployed in one or more agents, the lifecycle creates a docker container in each of them.

Deploy and start a service
~~~~~~~~~~~~~~~~~~~~~~~~~~~
The lifecycle offers different ways for deploying and starting a service in a set of mF2C agents.

1. If the lifecycle is working together with the other mF2C components, then the call should contain only three parameters: the service identifier, a user identifier (the on that generates the call), and the SLA agreement identifier.

.. code-block:: bash

    cat >post_service1.json <<EOF
    {
      "service_id": "service/6d1ba52b-4ce7-4333-914f-e434ddeeb591",
      "user_id": "user/testuser1",
      "agreement_id": "agreement/a7a30e2b-2ba1-4370-a1d4-af85c30d8713"
    }
    EOF
    curl -H "Content-Type: application/json" -X POST https://lifecycle:46000/api/v2/lm/service -d @post_service1.json --insecure

2. If the user wants to specify the agents where the service will be deployed, then we need another parameter: a list of agents

.. code-block:: bash

    cat >post_service2.json <<EOF
    {
      "service_id": "service/6d1ba52b-4ce7-4333-914f-e434ddeeb591",
      "user_id": "user/testuser1",
      "agreement_id": "agreement/a7a30e2b-2ba1-4370-a1d4-af85c30d8713"
      "agents_list": [{"agent_ip": "192.168.252.41"}, {"agent_ip": "192.168.252.42"}]
    }
    EOF
    curl -H "Content-Type: application/json" -X POST https://lifecycle:46000/api/v2/lm/service -d @post_service2.json --insecure

3. Finally, if the user wants to specify the service to be deployed, then we need to include the service content in the call to the lifecycle:

.. code-block:: bash

    cat >post_service3.json <<EOF
    {
    "service": {
      "name": "nginx-server-mf2c",
      "description": "nginx running on docker - mf2c version",
      "exec": "nginx",
      "os": "linux",
      "disk": 100,
      "category": 0,
      "num_agents": 2,
      "exec_type": "docker",
      "exec_ports": [80],
      "agent_type": "normal",
      "cpu_arch": "x86-64",
      "memory_min": 1000,
      "storage_min": 100,
      "req_resource": [],
      "opt_resource": []
    },
    "user_id": "user/testuser1",
    "agreement_id": "agreement/a7a30e2b-2ba1-4370-a1d4-af85c30d8713"
    "agents_list": [{"agent_ip": "192.168.252.41"}, {"agent_ip": "192.168.252.42"}]
    }
    EOF
    curl -H "Content-Type: application/json" -X POST https://lifecycle:46000/api/v2/lm/service -d @post_service3.json --insecure

If the service is successfully deployed, then the response should contain the resulting service instance object:

.. code-block:: bash

    {
      "service_instance": {
        "updated": "2018-05-08T10:00:58.397607Z",
        "agents": [
          {
            "port": 46100,
            "allow": true,
            "container_id": "fc6ecf2060c1a648d9376dab995543434f7b344bfcae9c178e02bde90d213777",
            "status": "started",
            "agent": {
              "href": "agent/default-value"
            },
            "url": "192.168.252.41",
            "master_compss": true,
            "num_cpus": 7
          },
          {
            "port": 46100,
            "allow": true,
            "container_id": "7d7acb04e1389c3a7242db40c005555c54340c7fce6a96647664ae7dd4659087",
            "status": "started",
            "agent": {
              "href": "agent/default-value"
            },
            "url": "192.168.252.42",
            "num_cpus": 7
          }
        ],
        "user": "user",
        "resourceURI": "http://schemas.dmtf.org/cimi/2/ServiceInstance",
        "acl": {
          "owner": {
            "type": "ROLE",
            "principal": "user"
          },
          "rules": [
            {
              "type": "ROLE",
              "right": "ALL",
              "principal": "user"
            },
            {
              "type": "ROLE",
              "right": "ALL",
              "principal": "ANON"
            }
          ]
        },
        "id": "service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7",
        "agreement": "not-defined",
        "operations": [
          {
            "rel": "edit",
            "href": "service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7"
          },
          {
            "rel": "delete",
            "href": "service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7"
          }
        ],
        "status": "started",
        "created": "2018-05-08T10:00:34.387Z",
        "service": "app_compss_test_01"
      },
      "message": "Deploy service",
      "error": false
    }


Stop and start a service instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. The service instance can be stopped with the following command:

.. code-block:: bash

  cat >put_stop_service_instance.json <<EOF
  {
    "operation":"stop"
  }
  EOF
  curl -H "Content-Type: application/json" -X PUT https://lifecycle:46000/api/v2/lm/service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7 -d @put_stop_service_instance.json --insecure


2. And it can be restarted again with the following command:

.. code-block:: bash

  cat >put_start_service_instance.json <<EOF
  {
    "operation":"start"
  }
  EOF
  curl -H "Content-Type: application/json" -X PUT https://lifecycle:46000/api/v2/lm/service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7 -d @put_start_service_instance.json --insecure


Sart a job (COMPSs services)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Services based on COMPSs can also run specific jobs in the mF2C agents. The lifecycle calls the COMPSs master, and this master distributes the job in the different worker agents.

1. Start a job:

.. code-block:: bash

    cat >put_start_job.json <<EOF
    {
    	"operation":"start-job",
    	"parameters":"<ceiClass>es.bsc.compss.test.TestItf</ceiClass><className>es.bsc.compss.test.Test</className><methodName>main</methodName><parameters><params paramId='0'><direction>IN</direction><type>OBJECT_T</type><array paramId='0'><componentClassname>java.lang.String</componentClassname><values><element paramId='0'><className>java.lang.String</className><value xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance' xmlns:xs='http://www.w3.org/2001/XMLSchema' xsi:type='xs:string'>3</value></element></values></array></params></parameters>"
	  }
    EOF
    curl -H "Content-Type: application/json" -X PUT https://lifecycle:46000/api/v2/lm/service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7 -d @put_start_job.json --insecure


Terminate a service instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The call to terminate a service instance, stops and removes the service instance (docker containers) from the agents.

1. Terminate a service instance:

.. code-block:: bash

    curl -H "Content-Type: application/json" -X DELETE https://lifecycle:46000/api/v2/lm/service-instance/2f6da0d0-e1e9-44fb-b03f-4259ce55a8f7 --insecure


Get service instances
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. To get all service instances:

.. code-block:: bash

    curl https://lifecycle:46000/api/v2/lm/service-instance/all --insecure


2. To get a specific service instance:

.. code-block:: bash

    curl https://lifecycle:46000/api/v2/lm/service-instance/4e1ab919-7a02-4260-993a-e0f5382ea580 --insecure

Landscaper module
-----------------------------
The Landscaper component is part of the Platform Manager's Service Orchestration module.
It constructs a graph model of the computing infrastructure. The graph details what service are running on what virtual infrastructure, and on which physical hosts that virtual infrastructure is running on.
For IT-1, the landscaper is accessible through *http://localhost:46020/

Get full graph of system
~~~~~~~~~~~~~~~~~~~~~~~~~~~
This method returns the entire graph of all infrastructure, containers and services currently deployed. It can be used to get a dump from the database

.. code-block:: bash

    curl -X GET https://localhost:46020/graph


Get full graph of system
~~~~~~~~~~~~~~~~~~~~~~~~~~~
This method returns the entire graph of all infrastructure, containers and services currently deployed. It can be used to get a dump from the database

.. code-block:: bash

    curl -X GET https://localhost:46020/graph


Get a service stack's subgraph
~~~~~~~~~~~~~~~~~~~~~~~~~~~
This method returns a subgraph from the model of all nodes involved in the running of this service. It starts with the service node (as input parameter) and works down through the entire structure of the graph

.. code-block:: bash

    curl -X GET https://localhost:46020/subgraph/<service_id>


Add GeoLocation info to nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Stores the geo-location as tags to selected nodes in the database. Useful to track static edge compute locations. Takes in an array of node Id's and geo locations


.. code-block:: bash
    cat >geo_location.json <<EOF
    { [
    	{
    	"id": "<node_id>",
    	"geo": "<geo_location_info"
    	}
      ]
    }
    EOF
    curl -H "Content-Type: application/json" -X PUT https://localhost:46020/coordinates -d @geo_location.json --insecure
