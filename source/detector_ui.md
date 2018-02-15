# Detector GUI user manual.

## 1. Overview

Detector’s GUI primary target is to provide a simple and intuitive way to provision Ruleset updates for Suricata and handle data exchange with Central system. Along the way it also provides means to seamlessly install essential extra tools to handle alarms locally, monitor system’s overall health and capture traffic if necessary.

## 2. Dashboard

* Registration Status: displays systems current state from Central system perspective.
* Last sync with Central: Registered system has online connection to the Central system to proactively retrieve Suricata rules provided and/or reviewed by CERT-EE.
* Rules: State of the received rules, unpublished rules
* Installed Components: 
* Component problems: System is constantly following the state of the components and this is the compact overview of those.

![Dashboard view](../images/image_4.png)

## 3. Rule manager

Provides intuitive way to manage rulesets or single rules to provide configuration to Suricata.

![Rule manager view](../images/image_5.png)

## 4. Components

Interface to manage and review component states and statuses. Possible actions include Install, uninstall, enable, disable, restart and recheck status.

* Auto upgrade: Debian’s "unattended-upgrades" functionality.
* Elasticsearch: Essential component, database for storing alerts coming from Suricata
* Evebox:GUI and parser to process alerts from Suricata and manage the alert flow.
* * Evebox-agent is part of evebox
* Moloch: Packet capturing, indexing and collecting system.
* Netdata: System metrics collecting and monitoring to provide accurate and live data about Detector Systems performance.
* Nginx: Web server
* NFSen: Lightweight packet capturing and graphing system.
* Suricata:Fast and robust network threat detection engine.
* Telegraf: System metrics collecting and transporting system to provide overview for CERT-EE
* OpenVPN: Provides private tunnel for CERT-EE to manage the system and assist local operator if necessary.

![Components view](../images/image_6.png)

## 5. Settings

* Alerts: It is possible to manage the verbosity and depth level of information which will be sent to CERT-EE

* Rules: Simple setup to trigger automated update of rules.

* Network: Capturing interface(s) reconfiguration.

* NFSen sampling: Sampling rate reconfiguration.

![Settings view](../images/image_7.png)