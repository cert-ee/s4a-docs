# Central components manual for S4A

## Prerequisites

- Working SaltStack master that is running at least version 2018.3.0
- Virtual or physical hardware to run component servers
- Basic knowledge of SaltStack. Mainly configuring pillars and applying states.

### Optional

- salt-cloud setup for provisioning central component vm-s

## Used software

- [SaltStack](https://saltstack.com/)
- [NodeJS](https://nodejs.org/en/) >= ver 10 LTS
- [Yarn](https://yarnpkg.com/en/)
- [Pm2](https://github.com/Unitech/pm2)
- [MongoDB](https://www.mongodb.com/) - version 4.0
- [Evebox](https://github.com/jasonish/evebox)
- [Elasticsearch](https://www.elastic.co/)
- [OpenVPN](https://openvpn.net/)
- [Grafana](https://grafana.com/)
- [InfluxDB](https://www.influxdata.com/)
- [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)
- [Nginx](https://nginx.org/en/)

## Installation

**Note! Double curly brackets are used to inidicate values that should be replaced.**

1. Ensure pillar definitions and state files for all components are present on the server. These can be found at [S4A repository](https://github.com/cert-ee/s4a). These include:
    - central
    - influx/grafana/telegraf
    - vpn
    - detector.elastic
    - nginx
    - repo
    - mongodb
    - reactor.sign_crt
    - reactor.vpn

By default states should be placed in `/srv/salt` and pillar files in `/srv/pillar`.

1. Configure pillar values. At a minimum> 
    - You should generate a hash for the first user in pillar/central/init.sls.
    - You should fix all of the example.com hostnames to actually point to your hosts.

1. In the master configuration, define [reactor events](https://docs.saltstack.com/en/latest/topics/reactor/):

    - By default this should be put in either `/etc/salt/master`, or `/etc/salt/master.d/reactor.conf`. More info at the reactor events link above.
    
    ```yaml
    reactor:
      - 'vpn/s4a/serial':
        - /srv/reactor/vpn.sls
      - 'salt/beacon/*/inotify//etc/openvpn/keys':
        - /srv/reactor/sign_crt.sls
    ```

    **NB! The second beacon definition matches on inotify beacon events for that specific folder, so make sure that the beacon is defined in pillar and assigned to central and that the path reflects the path set in openvpn pillar.**

1. Next up, we need to provision some virtual machines where to run the various central services. In case of a working [salt-cloud](https://docs.saltstack.com/en/latest/topics/cloud/) setup, you can use that. If provisioning manually, make sure to also add these machines as minions to your salt-master:
    - central.s4a.{{ example.com }}
    - influx.s4.{{ example.com }}
    - vpn.s4a.{{ example.com }}
    - es.s4a.{{ example.com }}
    - repo.s4a.{{ example.com }}

    - Single host central deployments are also possible, but currently untested. This will require significant modifications to the top file definitions.
    - The machine names are arbitrary but if using other names, make sure to change them in the top.sls files and highstate commands as well.
    
    **Template top file definitions are also included in the GitHub repository.**
1. Define pillar top file definitions, by default `/srv/pillar/top.sls`:

    ```yaml
    '*':
      - detector
    'central.s4a.{{ example.com }}':
      - vpn
      - beacons
      - central
      - detector
    'vpn.s4a.{{ example.com }}':
      - vpn
    ```

1. Define sls top file definitions, by default `/srv/salt/top.sls`:

    ```yaml
    'central.s4a.{{ example.com }}':
      - vpn.easy-rsa.config
      - beacon
      - central.bundle
    'es.s4a.{{ example.com }}':
      - detector.elastic
    'influx.s4a.{{ example.com }}':
      - influx
    'vpn.s4a.{{ example.com }}':
      - vpn
    'repo.s4a.{{ example.com }}':
      - repo
    ```

1. Restart salt-master: `systemctl restart salt-master.service` (To make sure, the reactor definitions are loaded)

1. Highstate repo.s4a.{{ example.com }}:

    ```bash
    salt 'repo.s4a.{{ example.com }}' state.highstate
    ```

1. Build deb packages using the scripts in the S4A GitHub repository.
1. Upload them to repo.s4a.{{ example.com }} and register them in the repo:

    ```bash
    reprepro -b /srv/repo.s4a.{{ example.com }}/repositories/ includedeb xenial /path/to/debs/*.deb
    ```

1. Highstate the servers:

    ```bash
    salt -E '(central|es|influx|keys).s4a.{{ example.com }}' state.highstate
    ```

    1. Now execute certificate transfer state. This will send the ca.crt, ta key and server cert and key, and crl.pem to the salt master

    ```bash
    salt 'central.s4a.{{ example.com }}' state.apply vpn.easy-rsa.send_certs
    ```

    1. Finally highstate the OpenVPN server.

    ```bash
    salt 'vpn.s4a.{{ example.com }}' state.highstate
    ```

1. Done

## Manual install

This will walk you through installing the central side manually.
The guide assumes the use of two machines. One machine for the APT repository. Second one that will host central components, elasticsearch, evebox, influx, openvpn.

**Note! Double curly brackets are used to inidicate values that should be replaced.**

### Repo server installation

1. Install reprepro

  ```bash
  sudo apt-get update && apt-get install reprepro
  mkdir -p /srv/{{ fqdn }}/repositories
  mkdir -p /srv/{{ fqdn }}/repositories/conf
  ```

1. Grab template files from https://github.com/cert-ee/s4a/tree/master/saltstack/salt/repo/files/repo/conf
 Place these files in the previously created mkdir -p /srv/{{ fqdn }}/repositories/conf
 directory and configure them to suit your needs

1. Install and configure nginx. You can find a template config file at:
  https://github.com/cert-ee/s4a/blob/master/saltstack/salt/repo/files/nginx.conf

### Central server installation

1. Configure repositorys and install packages for dependencies, elasticsearch and S4A central

    ```bash
    # Add elasticsearch
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
    echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list

    # Add MongoDB
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
    echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

    # Add NodeJS
    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

    # Add yarn
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

    # Add S4A repo
    curl -sL https://{{ Path to your repo key }} | sudo apt-key add -
    echo "deb [trusted=yes arch=amd64] https://{{ salt['pillar.get']('detector:repo') }}/ xenial universe"

    # Add Oracle java
    sudo add-apt-repository ppa:webupd8team/java

    # Refresh pacakge repos and install dependencies
    sudo apt-get update
    sudo apt-get install apt-transport-https python-software-properties wget curl
    sudo apt-get install oracle-java8-installer
    sudo apt-get install elasticsearch
    sudo apt-get install -y mongodb-org
    sudo apt-get install -y nodejs
    sudo apt-get install yarn
    sudo apt-get install s4a-central

    # Configure limits
    cat <<EOF >> /etc/security/limits.conf
    elasticsearch - nofile 65535
    elasticsearch - memlock unlimited
    root - memlock unlimited
    EOF

    # Set sysctl values. You should also configure these in /etc/sysctl.conf
    sysctl vm.swappiness=0
    sysctl vm.max_map_count=262144

    ```

1. Install evebox

    ```bash
    sudo add-apt-repository ppa:longsleep/golang-backports

    wget -qO - https://evebox.org/files/GPG-KEY-evebox | sudo apt-key add -
    echo "deb http://files.evebox.org/evebox/debian stable main" | sudo tee /etc/apt/sources.list.d/evebox.list

    sudo apt-get update
    sudo apt-get install git golang-go evebox
    ```

    1. Download GeoLite2-City from: http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz and extract it  to `/etc/evebox/`

    1. Configure an elasticsearch template for Evebox. Sample here:  https://github.com/cert-ee/s4a/blob/master/saltstack/salt/central/files/evebox/elasticsearch-template-es5x.json

    1. Configure the following files:
        - `/etc/evebox/evebox.yaml`
        - `/etc/evebox/agent.yaml`
        - `/etc/default/evebox`

        Templates for these files can be found here: https://github.com/cert-ee/s4a/tree/master/saltstack/salt/central/files/evebox

    1. Ensure evebox and evebox-agent services are running

1. Nginx as a reverse proxy

    1. Install packages

        ```bash
        sudo apt-get update && apt-get install nginx php-fpm
        ```

    1. Obtain a Let's Encrypt or another SSL certificate

    1. Configure nginx per the templates at:  https://github.com/cert-ee/s4a/tree/master/saltstack/salt/central/files/nginx

    1. Create users at: `/etc/nginx/.htpasswd`

1. Deploy OpenVPN server

    **NB! Manual install with no SaltStack means that OpenVPN certificates will need manual signing**

    1. `sudo apt-get update && sudo apt-get install openvpn`

    1. Create keys and certificates with the included easy-rsa package. Refer to the official OpenVPN install guide linked below. 

    1. Configure OpenVPN based on template files: `https://github.com/cert-ee/s4a/tree/master/saltstack/salt/vpn/files`

    The OpenVPN documentation is pretty good: https://openvpn.net/index.php/open-source/documentation/howto.html#config.
    Use the "subnet" topology when setting up.

1. Optionally add InfluxDB and Grafana

    ```bash
    # Add InfluxDB
    curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
    source /etc/lsb-release
    echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
    sudo apt-get update
    sudo apt-get install influxdb
    sudo systemctl start influxdb

    # Add Grafana
    curl -sL https://packagecloud.io/gpg.key | sudo apt-key add -
    echo "deb https://packagecloud.io/grafana/stable/debian/ jessie main" | sudo tee /etc/apt/sources.list.d/grafana.list
    sudo apt-get update && sudo apt-get install grafana
    sudo systemctl start grafana-server
    ```
1. Profit!
