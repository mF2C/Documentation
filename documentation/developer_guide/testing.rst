=======
Testing
=======

Manual Testing
==============

CIMI
----
*(these instructions have not been tested in Windows)*

CIMI will be running over HTTPS (through Traefik). 
Since this is a development and testing environment, we'll use a special CIMI HTTP header (:code:`slipstream-authn-info:internal ADMIN`)
to bypass user authentication and authorization, by impersonating *admin*.
Let's also assume that the TLS certificates in-use were self-signed, thus we'll need :code:`-k`.
Finally, for creating and modifying resources, the expected payload is always JSON, so we'll need the HTTP header :code:`content-type:application/json`).

For simplicity, let's setup the 4 possible operations in CIMI:

**CREATE**

.. code-block:: bash

    alias mf2c-curl-post="curl -XPOST -k -H 'slipstream-authn-info:internal ADMIN' -H 'content-type:application/json' "

**READ**

.. code-block:: bash

    alias mf2c-curl-get="curl -XGET -k -H 'slipstream-authn-info:internal ADMIN' "

**UPDATE**

.. code-block:: bash

    alias mf2c-curl-put="curl -XPUT -k -H 'slipstream-authn-info:internal ADMIN' -H 'content-type:application/json' "

**DELETE**

.. code-block:: bash

    alias mf2c-curl-delete="curl -XDELETE -k -H 'slipstream-authn-info:internal ADMIN' "

Let's also assume the test CIMI server is running at *localhost*.


Cloud Entry Point
~~~~~~~~~~~~~~~~~

When CIMI is ready, the *cloud-entry-point* should be available:

.. code-block:: bash

    mf2c-curl-read https://localhost/api/cloud-entry-point


Adding a new user
~~~~~~~~~~~~~~~~~

If the SMTP configuration is enabled, you shall receive a user validation email (check the SPAM folder).

.. code-block:: bash

    curl -XPOST -k -H 'content-type:application/json' https://localhost/api/user -d '''
    {
        "userTemplate": {
            "href": "user-template/self-registration",
            "password": "testpassword",
            "passwordRepeat" : "testpassword",
            "emailAddress": "your_email@",
            "username": "testuser"
        }
    }'''


Login
~~~~~

You **must have** validated the user before you can login. 
To login, simply create a session.

.. code-block:: bash

    curl -XPOST -k -H 'content-type:application/json' https://localhost/api/session --cookie-jar ~/cookies -b ~/cookies -d '''
    {
        "sessionTemplate": {
            "href": "session-template/internal",
            "username": "testuser",
            "password": "testpassword"
        }
    }'''

Get a collection of any resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve all the records of a certain resource type, simply do:

.. code-block:: bash

    mf2c-curl-read https://localhost/api/<resourceName>

where *resourceName* is something like *user*, *service*, *device*, etc.


Filter a collection of resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To filter for a specific set of resources, use CIMI's filtering grammar. Example:

.. code-block:: bash

    mf2c-curl-read 'https://cimi/api/<resourceName>?$filter=<AttrName>="<Value>"&$filter=<AttrName2><=<Value2>&$orderby=<AttrName3>:desc'


Get a specific resource
~~~~~~~~~~~~~~~~~~~~~~~

To get a specific resource, use its unique ID:

.. code-block:: bash

    mf2c-curl-read https://localhost/api/<resourceName>/<uuid>


Create a new service
~~~~~~~~~~~~~~~~~~~~

Example with only required fields:

.. code-block:: bash

    mf2c-curl-post https://localhost/api/service -d '''
    {
       "name": "compss-hello-world",
       "exec": "mf2c/compss-test:it2",
       "exec_type": "compss",
       "agent_type": "normal"
    }'''

Example with all optional fields:

.. code-block:: bash

    mf2c-curl-post https://localhost/api/service -d '''
    {
        "name": "compss-hello-world",
        "description": "Hello World Service",
        "exec": "mf2c/compss-test:it2",
        "exec_type": "compss",
        "exec_ports": [8080],
        "agent_type": "normal",
        "num_agents": "2",
        "cpu_arch": "x86-64",
        "os": "linux", 
        "memory_min": 1000,
        "storage_min": 100, 
        "disk": 100, 
        "req_resource": ["Location"],
        "opt_resource": ["SenseHat"]
    }'''

Create a service instance
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/service-instance -d '''
    {
        "user": {"href": "user/asasdasd"},
        "service": {"href": "service/asasdasd"},
        "agreement": {"href": "sla/asdasdasd"},
        "status": "running",
        "agents": [
            {
            "agent": {"href": "device/testdevice"}, 
            "port": 8081, 
            "container_id": "0938afd12323", 
            "status": "running", 
            "num_cpus": 3, 
            "allow": true
            }
        ]
    }'''


Create a "sharing-model" record
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/sharing-model -d '''
    {
        "description": "example of a sharing model resource instance",
        "max_apps": 3,
        "gps_allowed": false,
        "max_cpu_usage": 1,
        "max_memory_usage": 1024,
        "max_storage_usage": 500,
        "max_bandwidth_usage": 20,
        "battery_limit": 10
    }'''


Create a user profile
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/user-profile -d '''
    {
        "service_consumer": true,
        "resource_contributor": false
    }'''

Create an service level agreement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/agreement -d '''
    {
        "id": "a02",
        "name": "Agreement 02",
        "state": "stopped",
        "details":{
            "id": "a02",
            "type": "agreement",
            "name": "Agreement 02",
            "provider": { "id": "mf2c", "name": "mF2C Platform" },
            "client": { "id": "c02", "name": "A client" },
            "creation": "2018-01-16T17:09:45.0Z",
            "expiration": "2019-01-17T17:09:45.0Z",
            "guarantees": [
                {
                    "name": "TestGuarantee",
                    "constraint": "[test_value] < 10"
                }
            ]
        }
    }'''


Create an SLA violation
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/sla-violation -d '''
    {
        "guarantee" : "TestGuarantee",
        "datetime" : "2018-04-11T10:39:51.527008088Z",
        "agreement_id" : {"href": "agreement/4e529393-f659-44d6-9c8b-b0589132599b"},
        "constraint": "var1 < 100 and var2 > 100",
        "values": { "var1": 101, "var2": 100 }
    }'''


Add a new device
~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/device -d '''
    {
        "deviceID": "123",
        "isLeader": false,
        "os": "Linux-4.13.0-38-generic-x86_64-with-debian-8.10",
        "arch": "x86_64",
        "cpuManufacturer": "Intel(R) Core(TM) i7-8550U CPU @ 1.80GHz",
        "physicalCores": 4,
        "logicalCores": 8,
        "cpuClockSpeed": "1.8000 GHz",
        "memory": 7874.2109375,
        "storage": 234549.5078125,
        "powerPlugged": true,
        "networkingStandards": "['eth0', 'lo']",
        "ethernetAddress": "[snic(family=<AddressFamily.AF_INET: 2>, address='172.17.0.3', netmask='255.255.0.0', broadcast='172.17.255.255', ptp=None), snic(family=<AddressFamily.AF_PACKET: 17>, address='02:42:ac:11:00:03', netmask=None, broadcast='ff:ff:ff:ff:ff:ff', ptp=None)]",
        "wifiAddress": "Empty",
        "hwloc": "<xmlString>",
        "cpuinfo": "<rawCPUinfo>"
    }'''


Add the device-dynamic info
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/device-dynamic -d '''
    {
        "device": {"href": "device/123"},
        "ramFree": 4795.15234375,
        "ramFreePercent": 60.9,
        "storageFree": 208409.25,
        "storageFreePercent": 93.6,
        "cpuFreePercent": 93.5,
        "powerRemainingStatus": "30.75885328836425",
        "powerRemainingStatusSeconds": "3817",
        "ethernetAddress": "[snic(family=<AddressFamily.AF_INET: 2>, address='172.17.0.3', netmask='255.255.0.0', broadcast='172.17.255.255', ptp=None), snic(family=<AddressFamily.AF_PACKET: 17>, address='02:42:ac:11:00:03', netmask=None, broadcast='ff:ff:ff:ff:ff:ff', ptp=None)]",
        "wifiAddress": "Emp": [1595,8644,16,74,0,0,0,0],
        "wifiThroughputInfo": ["E","m","p","t","y"],
        "myLeaderID": {"href": "device/1"}
    }'''


Create a fog area
~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/fog-area -d '''
    {
        "leaderDevice": {"href": "device/123refegh"},
        "numDevices": 10,
        "ramTotal": 56789.90,
        "ramMax": 4569.34,
        "ramMin": 1478.34,
        "storageTotal": 120003456798.23456,
        "storageMax": 345678000.23456,
        "storageMin": 3456789.248,
        "avgProcessingCapacityPercent": 88.6,
        "cpuMaxPercent": 98.2,
        "cpuMinPercent": 56.7,
        "avgPhysicalCores": 4,
        "physicalCoresMax": 6,
        "physicalCoresMin":  2,
        "avgLogicalCores" : 4,
        "logicalCoresMax": 6,
        "logicalCoresMin": 2,
        "powerRemainingMax": "Device has unlimited power source",
        "powerRemainingMin": "88.2"
    }'''

Add the service operation report
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mf2c-curl-post https://localhost/api/service-operation-report -d '''
    {
        "serviceInstance": {"href": "service-instance/asasdasd"},
        "operation": "newMethod",
        "execution_time": 123.32
    }'''

