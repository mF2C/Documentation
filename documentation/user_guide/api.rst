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

Submit a service
~~~~~~~~~~~~~~~~
1. For any workflow to work, the first step is to submit a service to the service manager:

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

Check QoS provider
~~~~~~~~~~~~~~~~~~

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

Lifecycle Management module
-----------------------------
The Lifecycle Management component is part of the Platform Manager's Service Orchestration module. It is an internal component responsible for managing the services running in the mF2C clusters. For IT-1 this component will be accessible through *http://lifecycle:46000/api/v1*.

Services are based on docker images. Thus, when a service is deployed in one or more agents, the lifecycle creates a docker container in each of them.

Deploy and start a service
~~~~~~~~~~~~~~~~~~~~~~~~~~~
1. The lifecycle offers different ways for deploying and starting a service in a set of mF2C agents. If the lifecycle is working together with the other mF2C components, then the call should contain only three parameters: the service identifier, a user identifier (the on that generates the call), and the SLA agreement identifier.

.. code-block:: bash

    cat >post_service1.json <<EOF
    {
    "service_id": "service/c6aeb25c-ab2f-4207-8304-1eaf8ebcda6e",
	  "user_id": "rsucasas",
	  "agreement_id": "agreement/19e4cf61-6a9b-4d88-9f78-2408d568ed0e"
	}
    EOF
    curl -H "Content-Type: application/json" -X POST https://lifecycle:4600/api/v1/lifecycle -d @post_service1.json --insecure

2. If the user wants to specify the agents where the service will be deployed, then we need another parameter: a list of agents

.. code-block:: bash

    cat >post_service2.json <<EOF
    {
    "service_id": "service/c6aeb25c-ab2f-4207-8304-1eaf8ebcda6e",
	  "user_id": "rsucasas",
	  "agreement_id": "agreement/19e4cf61-6a9b-4d88-9f78-2408d568ed0e",
	  "agents_list": [{"agent_ip": "192.168.252.41"}, {"agent_ip": "192.168.252.42"}]
	}
    EOF
    curl -H "Content-Type: application/json" -X POST https://lifecycle:4600/api/v1/lifecycle -d @post_service2.json --insecure


3. Finally, if the user wants to specify the service to be deployed, then we need to include the service content in the call to the lifecycle:

.. code-block:: bash

    cat >post_service3.json <<EOF
    {
    "service": {
		"id": "service/9fbfd7cd-4154-450e-aeab-7a6b84153206",
		"name": "app_compss_test",
		"description": "app-compss Service",
		"exec": "mf2c/compss-test:latest",
		"resourceURI": "/app-compss",
		"category": {
			"cpu": "low",
			"memory": "low",
			"storage": "low",
			"inclinometer": false,
			"temperature": false,
			"jammer": false,
			"location": false,
			"accelerometer": false,
			"humidity": false,
			"battery_level": false,
			"door_sensor": false,
			"pump_sensor": false,
			"air_pressure": false,
			"ir_motion": false
		},
		"exec_type": "compss",
		"exec_ports": [
  			46100,
  			46101,
  			46102,
  			46103
  		]
  	},
	  "user_id": "rsucasas",
	  "agreement_id": "agreement/19e4cf61-6a9b-4d88-9f78-2408d568ed0e",
	  "agents_list": [{"agent_ip": "192.168.252.41"}, {"agent_ip": "192.168.252.42"}]
	}
    EOF
    curl -H "Content-Type: application/json" -X POST https://lifecycle:4600/api/v1/lifecycle -d @post_service3.json --insecure

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
    "service_instance_id":"9a22a9a7-6a9c-40e9-b2cf-983dde76293e",
	  "operation":"stop"
	  }
    EOF
    curl -H "Content-Type: application/json" -X PUT https://lifecycle:4600/api/v1/lifecycle -d @put_stop_service_instance.json --insecure


2. And it can be restarted again with the following command:

.. code-block:: bash

    cat >put_start_service_instance.json <<EOF
    {
    "service_instance_id":"9a22a9a7-6a9c-40e9-b2cf-983dde76293e",
	  "operation":"start"
	  }
    EOF
    curl -H "Content-Type: application/json" -X PUT https://lifecycle:4600/api/v1/lifecycle -d @put_start_service_instance.json --insecure


Sart a job (COMPSs services)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Services based on COMPSs can also run specific jobs in the mF2C agents. The lifecycle calls the COMPSs master, and this master distributes the job in the different worker agents.

1. Start a job:

.. code-block:: bash

    cat >put_start_job.json <<EOF
    {
    "service_instance_id":"9a22a9a7-6a9c-40e9-b2cf-983dde76293e",
  	"operation":"start-job",
  	"parameters":"<ceiClass>es.bsc.compss.test.TestItf</ceiClass><className>es.bsc.compss.test.Test</className><methodName>main</methodName><parameters><params paramId='0'><direction>IN</direction><type>OBJECT_T</type><array paramId='0'><componentClassname>java.lang.String</componentClassname><values><element paramId='0'><className>java.lang.String</className><value xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance' xmlns:xs='http://www.w3.org/2001/XMLSchema' xsi:type='xs:string'>3</value></element></values></array></params></parameters>"
	  }
    EOF
    curl -H "Content-Type: application/json" -X PUT https://lifecycle:4600/api/v1/lifecycle -d @put_start_job.json --insecure

For IT-1 we need to pass the lifecycle part of the XML needed ('parameters') to call the COMPSs REST API.

Terminate a service instances
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The call to terminate a service instance, stops and removes the service instance (docker containers) from the agents.

1. Terminate a service instance:

.. code-block:: bash

    cat >delete_service_instance.json <<EOF
    {
    "service_instance_id":"9a22a9a7-6a9c-40e9-b2cf-983dde76293e"
	  }
    EOF
    curl -H "Content-Type: application/json" -X DELETE https://lifecycle:4600/api/v1/lifecycle -d @delete_service_instance.json --insecure


Get service instances
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. To get all service instances:

.. code-block:: bash

    curl https://lifecycle:4600/api/v1/lifecycle/service-instance/all --insecure


2. To get a specific service instance:

.. code-block:: bash

    curl https://lifecycle:4600/api/v1/lifecycle/service-instance/4e1ab919-7a02-4260-993a-e0f5382ea580 --insecure
