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


.. _CIMI: https://www.dmtf.org/sites/default/files/standards/documents/DSP0263_2.0.0.pdf
.. _SlipStream: http://ssapi.sixsq.com/#cimi-api


