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

