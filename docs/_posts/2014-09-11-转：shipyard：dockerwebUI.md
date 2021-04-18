---
layout:     post
title:      转：shipyard：dockerwebUI
date:       2014-09-11
tags: [cnblogs]
---
https://github.com/shipyard/shipyard/wiki/QuickStart

# QuickStart

###  Pages 13

- **[Agent](https://github.com/shipyard/shipyard/wiki/Agent)**
- **[Agent Migration](https://github.com/shipyard/shipyard/wiki/Agent-Migration)**
- **[API](https://github.com/shipyard/shipyard/wiki/API)**
- **[Application Routing](https://github.com/shipyard/shipyard/wiki/Application-Routing)**
- **[Applications](https://github.com/shipyard/shipyard/wiki/Applications)**
- **[Deployment](https://github.com/shipyard/shipyard/wiki/Deployment)**
- **[FAQ](https://github.com/shipyard/shipyard/wiki/FAQ)**
- **[Home](https://github.com/shipyard/shipyard/wiki)**
- **[newbie start](https://github.com/shipyard/shipyard/wiki/newbie-start)**
- **[QuickStart](https://github.com/shipyard/shipyard/wiki/QuickStart)**
- **[Roadmap](https://github.com/shipyard/shipyard/wiki/Roadmap)**
- **[shpd](https://github.com/shipyard/shipyard/wiki/shpd)**
- **[v2 Beta Testing](https://github.com/shipyard/shipyard/wiki/v2-Beta-Testing)**

##### Clone this wiki locally

This will get you up and running Shipyard.

# [](https://github.com/shipyard/shipyard/wiki/QuickStart#docker)Docker

In order to use Shipyard you will need [Docker](http://docker.io/). The setup of Docker itself is out of scope here. Check [http://www.docker.io/gettingstarted/](http://www.docker.io/gettingstarted/) for details. Once you have at least one Docker host, come back.

# [](https://github.com/shipyard/shipyard/wiki/QuickStart#docker-host-configuration)Docker Host Configuration

To setup the Docker host for the shipyard agent, start Docker with the following options if you are using Debian or Ubuntu < 14.04 (you can also edit `/etc/default/docker` and add these to `DOCKER_OPTS`) or `/etc/default/docker.io` if using Ubuntu 14.04:

`-H tcp://127.0.0.1:4500 -H unix:///var/run/docker.sock` - Note, this will only bind to the localhost address. If you would like to access the API tcp port externally replace 127.0.0.1 with 0.0.0.0.

For Redhat or CentOS edit `/etc/sysconfig/docker`

change: `other_args=""`

to: `other_args="-H tcp://127.0.0.1:4500 -H unix:///var/run/docker.sock"` - Note, this will only bind to the localhost address. If you would like to access the API tcp port externally replace 127.0.0.1 with 0.0.0.0.

Once this is changed, restart the Docker daemon. This will enable both the socket support and TCP on localhost.

You will also need to make sure port 4500 (or whichever you use for the agent below) is open to the Shipyard host.

### [](https://github.com/shipyard/shipyard/wiki/QuickStart#security-note)Security Note

Currently there is no authentication in Docker itself. Since Shipyard needs to remotely manage the hosts, you must make sure to keep them secured through firewall or other methods. We are working on authentication in the agent to alleviate this in the near future.

# [](https://github.com/shipyard/shipyard/wiki/QuickStart#shipyard)Shipyard

Once you have Docker installed and running, simply use the [Shipyard Deploy](https://github.com/shipyard/shipyard-deploy) image to deploy a local Shipyard stack. This will start five containers (redis, router, lb, db, and shipyard).

`docker run -i -t -v /var/run/docker.sock:/docker.sock shipyard/deploy setup`

Once you have Shipyard running, login at http://<docker-host-ip>:8000 using the default credentials:

- Username: `admin`
- Password: `shipyard`

!! It is recommended to change the admin password after logging in.

### [](https://github.com/shipyard/shipyard/wiki/QuickStart#note)Note

If you have trouble connecting to the Shipyard container make sure you have updated UFW ([http://docs.docker.io/en/latest/installation/ubuntulinux/#docker-and-ufw](http://docs.docker.io/en/latest/installation/ubuntulinux/#docker-and-ufw)).

### [](https://github.com/shipyard/shipyard/wiki/QuickStart#127011-dns-server-problem-on-ubuntu)127.0.1.1 DNS server problem on Ubuntu

On Ubuntu by default `NetworkManager` runs `dnsmasq` service, which sets `127.0.1.1` as your DNS server. This causes problems if you are forced to use internal DNS server. This can be fixed by executing `sudo sed 's@dns=dnsmasq@#dns=dnsmasq@' -i /etc/NetworkManager/NetworkManager.conf && sudo service network-manager restart && cat /etc/resolv.conf && nm-tool` (this command turns off `dnsmasq` usage in NetworkManager configuration, restarts network manager, prints out new DNS settings and prints new connection informations for verification).

!! This operation can be reversed by executing `sudo sed 's@#dns=dnsmasq@dns=dnsmasq@' -i /etc/NetworkManager/NetworkManager.conf && sudo service network-manager restart`.

## [](https://github.com/shipyard/shipyard/wiki/QuickStart#hosts)Hosts

To add a host, simply install the [Shipyard Agent](https://github.com/shipyard/shipyard-agent) and register/run:

- Download the latest agent from [https://github.com/shipyard/shipyard-agent/releases](https://github.com/shipyard/shipyard-agent/releases) to the Docker host
- `shipyard-agent -url http://my-shipyard-host:port -register`
- `shipyard-agent -url http://my-shipyard-host:port -key 1234567890abcdefg`
- Authorize the host in Shipyard

## [](https://github.com/shipyard/shipyard/wiki/QuickStart#images)Images

Once the import is complete, click the "Containers" link. On the right click "Create". In the first dropdown, select the `ehazlett/py-helloworld` image. In the "Ports" text field enter 8000. Scroll to the bottom and in the "Hosts" section, select your docker host and click "Create". After launch you will see the new container in the "Containers" section. Click on the ID to see details. In the bottom left the port mappings are listed. Click on the "Mapping" link. This should open a new tab to the application. You should see something like "Hello World from Flask".
