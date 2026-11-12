# SOC Home Lab Setup Guide

## Lab Goal

The goal of this lab is to simulate a small SOC environment where endpoint and network logs are collected, analyzed, and used for threat detection and incident response practice.

This lab is being built step by step for cybersecurity internship preparation.

## Planned Lab Components

| Component           | Purpose                                                     | Status                    |
|---------------------|-------------------------------------------------------------|---------------------------|
| Ubuntu Test Machine | Generate test activity and practice Linux/security commands | Completed                 |
| Ubuntu Scanner      | Run safe Nmap scans inside the private lab network          | Completed                 |
| Wazuh Server        | SIEM/XDR dashboard and alert analysis                       | Completed                 |
| Windows Endpoint    | Collect Windows event logs                                  | Completed                 |
| Suricata IDS        | Network intrusion detection                                 | Installed / Tested        |
| Wireshark           | Manual packet analysis                                      | Installed on host machine |
| Nmap                | Generate safe scan activity for detection practice          | Installed                 |
| OpenSSH Server      | Practice SSH access and failed login detection              | Installed                 |

## Current Completed Setup

### Ubuntu Test Machine

The first Ubuntu test machine has been created successfully in Oracle VirtualBox.

Completed work:

* Ubuntu 24.04.4 LTS installed
* VM configured with 4096 MB RAM and 2 processors
* NAT networking verified
* Basic tools installed: `nmap`, `net-tools`, `curl`, `git`, `openssh-server`
* SSH socket verified on port 22
* VirtualBox snapshots created for recovery
* Setup screenshots documented

Detailed documentation:

* [Day 02 – Ubuntu VM Setup](docs/day-02-ubuntu-vm-setup.md)

### Wazuh Server

The Wazuh server VM has been created and installed successfully using Ubuntu Server 24.04.4 LTS.

Completed work:

* Wazuh Server VM created in Oracle VirtualBox
* VM configured with 8192 MB RAM and 4 processors
* 50 GB virtual disk allocated
* NAT network configured for internet access
* Host-only network configured with IP address `192.168.56.10`
* Wazuh server installed successfully
* Wazuh services verified as running:

  * `wazuh-manager.service`
  * `wazuh-indexer.service`
  * `wazuh-dashboard.service`
  * `filebeat.service`
* Wazuh dashboard accessed from the Windows host browser
* Admin password saved privately and not uploaded to GitHub
* VirtualBox snapshot created after successful Wazuh installation

Detailed documentation:

* [Day 03 – Wazuh Server Setup](docs/day-03-wazuh-server-setup.md)

### Ubuntu Agent Enrollment

The Ubuntu Test Machine has been connected to the host-only lab network and enrolled as a Wazuh agent.

Completed work:

* Host-only network configured on Ubuntu Test Machine
* Static IP address assigned: `192.168.56.20`
* Connectivity to Wazuh Server verified
* Wazuh agent installed
* Wazuh agent service verified as running
* Ubuntu endpoint appeared as active in the Wazuh dashboard

Detailed documentation:

* [Day 04 – Ubuntu Agent Enrollment](docs/day-04-ubuntu-agent-enrollment.md)

### Ubuntu Scanner

A separate Ubuntu Scanner VM was created to run safe Nmap scans inside the private lab network.

Completed work:

* Ubuntu Scanner VM created in Oracle VirtualBox
* Basic tools installed, including `nmap`
* Host-only adapter configured
* Static IP address assigned: `192.168.56.40`
* Connectivity verified to Wazuh Server at `192.168.56.10`
* Connectivity verified to Ubuntu Test Machine at `192.168.56.20`

Purpose:

The Ubuntu Scanner is used to generate safe scan activity against lab endpoints only.

### Failed SSH Login Detection

Manual failed SSH login attempts were generated from Ubuntu Scanner against the Ubuntu Test Machine.

Completed work:

* SSH service verified as running on Ubuntu Test Machine
* UFW rule added to allow SSH from Ubuntu Scanner
* Failed SSH attempts generated manually using the username `wronguser`
* Authentication logs reviewed on Ubuntu Test Machine
* Wazuh dashboard reviewed for failed SSH login events
* Wazuh showed SSH-related alerts for non-existent user login attempts
* Source IP confirmed as `192.168.56.40`
* Failed SSH login incident report written

Detailed documentation:

* [Failed SSH Login Incident Report](incident-reports/failed-ssh-login-incident.md)

### Suspicious Command Activity and EICAR Test

Suspicious command activity was generated on the Ubuntu Test Machine to simulate basic post-login enumeration behavior.

Completed work:

* Harmless enumeration commands executed on Ubuntu Test Machine
* Sensitive file access attempt performed using `/etc/shadow`
* `/etc/shadow` contents were not uploaded to GitHub
* Wazuh dashboard reviewed for sudo command activity
* Wazuh event details confirmed the executed command
* Suspicious command incident report written

EICAR safe malware testing was also performed using ClamAV.

Completed work:

* ClamAV installed on Ubuntu Test Machine
* EICAR test file created locally for safe anti-malware testing
* ClamAV detected the EICAR test file as `Eicar-Signature FOUND`
* Wazuh reviewed for EICAR-related events
* Limitation documented: Wazuh did not clearly show an EICAR malware detection event during this test
* EICAR test file removed after testing
* Actual EICAR test file was not uploaded to GitHub

Detailed documentation:

* [Suspicious Command Incident Report](incident-reports/suspicious-command-incident.md)
* [EICAR Test Notes](sample-alerts/eicar-test-notes.md)

### Suricata Network Detection and Wireshark Evidence

Suricata was installed on the Ubuntu Test Machine to provide network-based detection inside the lab environment.

Completed work:

* Suricata installed on Ubuntu Test Machine
* Suricata version verified
* Suricata rules updated
* Suricata configuration tested successfully
* Host-only lab interface confirmed as `enp0s8`
* Suricata started on the host-only interface
* Custom lab rules created for scanner-to-target traffic
* Ubuntu Scanner generated ICMP and TCP traffic against Ubuntu Test Machine
* Suricata `fast.log` reviewed
* Suricata alerts confirmed traffic from `192.168.56.40` to `192.168.56.20`
* Wireshark used on Windows host to capture host-only lab traffic
* Suricata and Wireshark incident report written

Detailed documentation:

* [Suricata and Wireshark Network Incident Report](incident-reports/suricata-wireshark-network-incident.md)

### Windows Endpoint Log Collection

A Windows Endpoint VM was created and enrolled as a Wazuh agent to collect Windows Security Event Logs.

Completed work:

* Windows 11 Enterprise Evaluation VM created in Oracle VirtualBox
* VM configured with 4096 MB RAM and 2 processors
* 64 GB virtual disk allocated
* NAT network configured for internet access
* Host-only network configured with IP address `192.168.56.30`
* Connectivity to Wazuh Server verified at `192.168.56.10`
* Wazuh agent installed on Windows Endpoint
* Wazuh service started successfully
* Windows Endpoint appeared as active in the Wazuh dashboard
* Windows failed logon auditing enabled
* Local test user created for safe failed login testing
* Windows Security Event ID `4625` confirmed locally
* Wazuh dashboard reviewed for Windows failed login events
* Wazuh showed `Logon Failure - Unknown user or bad password`
* Windows failed login incident report written

Detailed documentation:

* [Windows Failed Login Incident Report](incident-reports/windows-failed-login-incident.md)

## Lab Network Design

The lab uses NAT networking for internet access and a host-only network for internal lab communication.

IP address structure:

| Machine             | Planned IP Address   | Role                                  |
|---------------------|----------------------|---------------------------------------|
| Wazuh Server        | 192.168.56.10        | SIEM/XDR server                       |
| Ubuntu Test Machine | 192.168.56.20        | Monitored endpoint / scan target      |
| Ubuntu Scanner      | 192.168.56.40        | Scanner / Nmap machine                |
| Windows Endpoint    | 192.168.56.30        | Windows endpoint / Windows event logs |

The Wazuh Server currently uses the host-only IP address:

```text
192.168.56.10
```

The Ubuntu Test Machine is connected to the host-only lab network with the IP address `192.168.56.20` and has been enrolled as an active Wazuh agent.

The Windows Endpoint is connected to the host-only lab network with the IP address `192.168.56.30` and has been enrolled as an active Wazuh agent.

## Detection Scenarios

The lab is being used to test and document these detection scenarios:

| Scenario                                         | Status                 |
|--------------------------------------------------|------------------------|
| Nmap port scan detection                         | Completed              |
| Failed SSH login / brute-force attempt detection | Completed              |
| Suspicious command execution                     | Completed              |
| Safe malware test detection using EICAR          | Completed / Documented |
| Suricata network alert                           | Completed              |
| Windows failed login detection                   | Completed              |

## Setup Status

* [x] GitHub repository created
* [x] Ubuntu test machine created
* [x] Ubuntu test machine updated
* [x] Basic tools installed on Ubuntu test machine
* [x] SSH socket enabled and verified
* [x] VirtualBox snapshots created
* [x] Wazuh server VM created
* [x] Host-only network configured for Wazuh server
* [x] Wazuh server installed
* [x] Wazuh services verified
* [x] Wazuh dashboard accessed successfully
* [x] Ubuntu test machine connected to host-only network
* [x] Wazuh agent installed on Ubuntu test machine
* [x] Ubuntu test machine active in Wazuh dashboard
* [x] Ubuntu scanner VM created
* [x] Ubuntu scanner connected to host-only network
* [x] Nmap scan performed inside lab network
* [x] UFW firewall logs captured as scan evidence
* [x] Nmap scan incident report written
* [x] Failed SSH login detection completed
* [x] Failed SSH login incident report written
* [x] Suspicious command detection completed
* [x] Suspicious command incident report written
* [x] EICAR safe malware test completed
* [x] EICAR test notes written
* [x] Suricata installed
* [x] Suricata network alert test completed
* [x] Wireshark host-only traffic captured
* [x] Suricata and Wireshark incident report written
* [x] Windows endpoint created
* [x] Windows endpoint connected to host-only network
* [x] Wazuh agent installed on Windows endpoint
* [x] Windows endpoint active in Wazuh dashboard
* [x] Windows failed login detection completed
* [x] Windows failed login incident report written
* [x] Architecture diagram completed
* [x] Final documentation polish completed

## Setup Progress Logs

* [Day 02 – Ubuntu VM Setup](docs/day-02-ubuntu-vm-setup.md)
* [Day 03 – Wazuh Server Setup](docs/day-03-wazuh-server-setup.md)
* [Day 04 – Ubuntu Agent Enrollment](docs/day-04-ubuntu-agent-enrollment.md)
* [Nmap Scan Incident Report](incident-reports/nmap-scan-incident.md)
* [Failed SSH Login Incident Report](incident-reports/failed-ssh-login-incident.md)
* [Suspicious Command Incident Report](incident-reports/suspicious-command-incident.md)
* [EICAR Test Notes](sample-alerts/eicar-test-notes.md)
* [Suricata and Wireshark Network Incident Report](incident-reports/suricata-wireshark-network-incident.md)
* [Custom Suricata Lab Rules](detection-rules/custom-suricata-lab-rules.md)
* [Windows Failed Login Incident Report](incident-reports/windows-failed-login-incident.md)

## Next Steps

1. Prepare the project for portfolio and resume use.
2. Continue improving the lab in future versions.