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

    curl -XGET https://nuv.la/api/event --cookie-jar ~/cookies -b ~/cookies -sS


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


