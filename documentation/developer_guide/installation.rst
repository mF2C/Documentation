Installation
============

Installing the mF2C System
--------------------------

**For Linux only**

1. download and run the Linux installation script (docker-compose required)

.. code-block:: bash

    git clone https://github.com/mF2C/mF2C.git
    cd linux
    ./mf2c-deployment.sh
    

Installing a Leader
~~~~~~~~~~~~~~~~~~~


Installing a regular agent
~~~~~~~~~~~~~~~~~~~~~~~~~~





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