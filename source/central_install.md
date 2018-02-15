# Central components manual for S4A

## Prerequisites

- Working SaltStack master that is running at least version 2017.7.1
- Virtual or physical hardware to run component servers
- Basic knowledge of SaltStack. Mainly configuring pillars and applying states.

### Optional

- salt-cloud setup for provisioning central component vm-s

## Used software

- [SaltStack](https://saltstack.com/)
- [NodeJS](https://nodejs.org/en/) >= v8
- [Yarn](https://yarnpkg.com/en/)
- [Pm2](https://github.com/Unitech/pm2)
- [MongoDB](https://www.mongodb.com/) - version 3.4
- [Evebox](https://github.com/jasonish/evebox)
- [Elasticsearch](https://www.elastic.co/)
- [OpenVPN](https://openvpn.net/)
- [Grafana](https://grafana.com/)
- [InfluxDB](https://www.influxdata.com/)
- [Nginx](https://nginx.org/en/)

## Installation

1. Ensure pillar definitions and state files for all components are present on the server. These can be found at [S4A repository](https://github.com/cert-ee/s4a). These include:
    - central
    - influx/grafana/telegraf
    - vpn
    - detector.elastic
    - nginx
    - sks
    - repo
    - mongodb
    - reactor.sign_crt
    - reactor.vpn

1. Configure pillar values if needed
    - At minimum you should generate a hash for the first user in pillar/central/init.sls.

1. In the master configuration, define reactor events:

    ```yaml
    reactor:
      - 'vpn/s4a/serial':
        - /srv/reactor/vpn.sls
      - 'salt/beacon/*/inotify//etc/openvpn/keys':
        - /srv/reactor/sign_crt.sls
    ```

    **NB! The second beacon definition matches on inotify beacon events for that specific folder, so make sure that the beacon is defined in pillar and assigned to central and that the path reflects the path set in openvpn pillar.**

1. Set up virtual machines, either with salt-cloud or manually:
    - central.s4a.cert.ee
    - influx.s4.cert.ee
    - vpn.s4a.cert.ee
    - keys.s4a.cert.ee
    - es.s4a.cert.ee
    - repo.s4a.cert.ee

    **Template top file definitions are also included in the GitHub repository.**
1. Define pillar top file definitions:

    ```yaml
    '*':
      - detector
    'central.s4a.cert.ee':
      - vpn
      - beacons
      - central
      - detector
    'vpn.s4a.cert.ee':
      - vpn
    'keys.s4a.cert.ee':
      - sks
    ```

1. Define sls top file definitions:

    ```yaml
    'central.s4a.cert.ee':
      - vpn.easy-rsa.config
      - beacon
      - central.bundle
    'es.s4a.cert.ee':
      - detector.elastic
    'influx.s4a.cert.ee':
      - influx
    'vpn.s4a.cert.ee':
      - vpn
    'keys.s4a.cert.ee':
      - sks.config
    'repo.s4a.cert.ee':
      - repo
    ```

1. Restart salt-master: `systemctl restart salt-master.service`

1. Highstate repo.s4a.cert.ee:

    ```bash
    salt 'repo.s4a.cert.ee' state.highstate
    ```

1. Build deb packages using the scripts in the S4A GitHub repository.
1. Upload them to repo.s4a.cert.ee and register them in the repo:

    ```bash
    reprepro -b /srv/repo.s4a.cert.ee/repositories/ includedeb xenial /path/to/debs/*.deb
    ```

1. Highstate the servers:

    ```bash
    salt -E '(central|es|influx|keys).s4a.cert.ee' state.highstate
    ```

    1. Now execute certificate transfer state. This will send the ca.crt, ta key and server cert and key, and crl.pem to the salt master

    ```bash
    salt 'central.s4a.cert.ee' state.apply vpn.easy-rsa.send_certs
    ```

    1. Finally highstate the OpenVPN server.

    ```bash
    salt 'vpn.s4a.cert.ee' state.highstate
    ```

1. Done
