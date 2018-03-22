Ubuntu
======

Installing with Docker
----------------------

To install mF2C with Docker, you must ensure that your Ubuntu 
machine is at least equipped with Docker and Docker Compose. As `sudo`:

1. prepare the environment (https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository)

```bash
# remove previous installations 
apt-get remove -y docker docker-engine docker-ce docker.io

apt-get update

apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

apt-key fingerprint 0EBFCD88
```

    1a. for x86_64 / amd64:
    
    ```bash
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
    ```

    1a. for armhf:
    
    ```bash
    add-apt-repository \
        "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
    ```

2. install Docker

```bash 
apt-get update

apt-get install -y docker-ce
```

3. install Docker Compose (https://docs.docker.com/compose/install/#install-compose)

```bash
curl -L https://github.com/docker/compose/releases/download/1.20.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

