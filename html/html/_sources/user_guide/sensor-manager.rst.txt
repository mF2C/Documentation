Sensor Manager
==============

The mF2C Sensor Manager provides the ability for applications to subscribe to sensor data.
Multiple applications can subscribe to data from the same sensor, as this provides an
abstraction over the actual hardware interaction with *sensor drivers* and only requires 
a subscription to a stream of values.

Drivers are automatically spawned according to the sensors idenfified by resource
categorisation and present in `device-dynamic`. The mapping between sensor hardware
and sensor drivers is defined in the sensor manager repository.

Creating and building a sensor driver
-------------------------------------

Sensor drivers are distributed as Docker images that read sensor data and publish it back
to the sensor manager. The interface is language-agnostic. An example sensor driver written
in golang is included in the sensor driver repository and published as a Docker image.

The following environment variables are passed to the sensor driver application:

.. code-block:: bash

    SENSOR_MANAGER_HOST=<host>
    SENSOR_MANAGER_PORT=<port>
    SENSOR_MANAGER_PATH_SUFFIX=[path suffix]
    SENSOR_MANAGER_USERNAME=<username>
    SENSOR_MANAGER_PASSWORD=<password>
    SENSOR_MANAGER_TOPIC=<topic>
    SENSOR_CONNECTION_INFO=<json>

`SENSOR_MANAGER_HOST` and `_PORT` define the location of the endpoint the sensor manager
is listening for values at. The endpoint is a WebSocket-based MQTT server. It may be running
behind a reverse proxy which rewrites its address, so `_PATH_SUFFIX` is provided, or is empty,
for any changes to the HTTP URL path, applied as a suffix.

`_USERNAME` and `_PASSWORD` are MQTT connection credentials for your sensor driver. Publish 
values to the topic specified in `_TOPIC`. Any custom physical sensor connection information 
is available as a JSON object in `SENSOR_CONNECTION_INFO`.

Outgoing values must be JSON objects conforming to the following schema:

.. code-block:: json

    {
        "SensorId":   "string, an ID that differentiates this piece of hardware sensor from others",
        "SensorType": "string, the type of the sensor (hardware)",
        "Quantity":   "string, the SI dimension, e.g. humidity",
        "Timestamp":  "string, RFC 3339, when the measurement was taken",
        "Value":      "float, the measurement value",
        "Unit":       "string, the SI base unit of the value"
    }

The resulting application must be packaged as a Docker image accessible to the machine on
which it will be deployed. 

As a final step, add the sensor driver information to `sensor-container-map.json` present in
the sensor manager repository.

Deploying the sensor manager
----------------------------

Sensor manager and related images are hosted on Docker Hub under:

  - `mf2c/sensor-manager`
  - `mf2c/sensor-manager-mosquitto`
  - `mf2c/sensor-manager-example-driver`
  - `mf2c/sensor-manager-example-application`

The sensor manager repository includes a script (`deploy-mf2c.sh`) that deploys the sensor manager
onto mF2C as an optional add-on.

Usage: `sh deploy-mf2c.sh <mf2c-ip>`

Component configuration is customisable and described in the repository's `docker-compose.yml`.

Reading sensor values
---------------------

A client application reads sensor data by subscribing to an MQTT topic with credentials obtained
from the sensor manager's API. An example application written in golang is included in the
sensor-manager repository.

There is only one external API endpoint: `/topics`. Hosted behind the default reverse proxy,
this is translated to `/sensor-manager/api/topics`. This endpoint returns a JSON list of topics
that are currently served by the sensor manager. 

The client application connects to the sensor manager's WebSocket-based MQTT server on the 
corresponding topic with the specified credentials and begins reading the data. By default, the
server is behind a reverse proxy and its path changed to `/sensor-manager/stream/`.

Values returned by the stream are in the following format:

.. code-block:: json

    {
        "Timestamp": "string, RFC 3339, when the measurement was taken",
        "Value":     "float, the measurement value",
        "Unit":      "string, the SI base unit of the value"
    }

