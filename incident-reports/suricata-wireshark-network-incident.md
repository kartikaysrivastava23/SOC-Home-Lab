# Incident Report: Suricata Network Detection and Wireshark Evidence

## Summary

Network-based detection was tested inside the private lab environment using Suricata and Wireshark.

Suricata was installed on the Ubuntu-Test-Machine and configured to monitor the host-only lab interface. Ubuntu-Scanner generated ICMP and TCP traffic against the Ubuntu-Test-Machine. Suricata detected the lab traffic using custom lab rules.

Wireshark was also used on the Windows host to capture host-only lab traffic.

## Lab Environment

| Machine             | Role                                 | IP Address    |
| ------------------- | ------------------------------------ | ------------- |
| Wazuh-Server        | SIEM/XDR server                      | 192.168.56.10 |
| Ubuntu-Test-Machine | Monitored endpoint / Suricata sensor | 192.168.56.20 |
| Ubuntu-Scanner      | Scanner / traffic source             | 192.168.56.40 |
| Windows Host        | Wireshark packet capture             | 192.168.56.1  |

## Activity Performed

Suricata was installed on the Ubuntu-Test-Machine.

The lab interface was confirmed as:

```text
enp0s8 → 192.168.56.20
```

Suricata was started on the host-only interface:

```bash
sudo suricata -c /etc/suricata/suricata.yaml -i enp0s8 -k none --runmode=single -S /tmp/lab.rules -l /var/log/suricata
```

Custom lab rules were used to detect traffic from Ubuntu-Scanner to Ubuntu-Test-Machine.

Example rule purpose:

```text
Detect ICMP and TCP traffic from 192.168.56.40 to 192.168.56.20
```

Traffic was generated from Ubuntu-Scanner using:

```bash
ping -c 4 192.168.56.20
sudo nmap -sS -Pn 192.168.56.20
```

## Detection and Evidence

Suricata generated alerts in `fast.log`.

Important alert evidence included:

```text
LAB ICMP ping from Ubuntu Scanner
192.168.56.40 -> 192.168.56.20
```

```text
LAB TCP traffic from Ubuntu Scanner
192.168.56.40 -> 192.168.56.20
```

This confirmed that Suricata detected network traffic from the scanner machine to the monitored endpoint.

## Wireshark Evidence

Wireshark was used on the Windows host to capture traffic on the VirtualBox host-only adapter.

The Windows host successfully captured ICMP traffic between:

```text
192.168.56.1 ↔ 192.168.56.20
```

This confirmed that Wireshark could observe traffic on the host-only lab network.

## Evidence Screenshots

| Screenshot                                                                                                    | Description                                       |
|:--------------------------------------------------------------------------------------------------------------|:--------------------------------------------------|
| [01-suricata-installed.png](../screenshots/day-08-suricata-wireshark/01-suricata-installed.png)               | Suricata installed and version verified           |
| [02-suricata-rules-updated.png](../screenshots/day-08-suricata-wireshark/02-suricata-rules-updated.png)       | Suricata rules/configuration test completed       |
| [03-suricata-lab-interface.png](../screenshots/day-08-suricata-wireshark/03-suricata-lab-interface.png)       | Lab interface confirmed as `enp0s8`               |
| [04-suricata-running-enp0s8.png](../screenshots/day-08-suricata-wireshark/04-suricata-running-enp0s8.png)     | Suricata running on the lab interface             |
| [05-scanner-traffic-generated.png](../screenshots/day-08-suricata-wireshark/05-scanner-traffic-generated.png) | Ubuntu-Scanner generated Nmap traffic             |
| [06-suricata-log-evidence.png](../screenshots/day-08-suricata-wireshark/06-suricata-log-evidence.png)         | Suricata alerts showing scanner-to-target traffic |
| [07-wireshark-lab-traffic.png](../screenshots/day-08-suricata-wireshark/07-wireshark-lab-traffic.png)         | Wireshark capture of host-only lab traffic        |

## Analysis

Suricata provided clearer network-based visibility than endpoint-only monitoring for scan and traffic detection.

In earlier testing, Wazuh did not clearly classify a basic Nmap scan as an Nmap scan. In this scenario, Suricata produced direct network alert evidence showing traffic from the scanner IP address to the target IP address.

Wireshark provided supporting packet-level visibility on the host-only lab network.

## Severity

Medium

## MITRE ATT&CK Mapping

| Tactic    | Technique                         |
|-----------|-----------------------------------|
| Discovery | T1046 – Network Service Discovery |

## Verdict / Triage Decision

| Field               | Value                                                                                                                                                                       |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Classification      | True Positive                                                                                                                                                               |
| Activity Type       | Authorized lab simulation                                                                                                                                                   |
| Detection Outcome   | Confirmed through Suricata `fast.log` alerts                                                                                                                                |
| Supporting Evidence | Wireshark confirmed host-only lab traffic visibility                                                                                                                        |
| Status              | Closed                                                                                                                                                                      |
| Escalation          | Not escalated                                                                                                                                                               |
| Reason              | The ICMP and TCP traffic was intentionally generated from Ubuntu-Scanner to Ubuntu-Test-Machine. Suricata custom lab rules successfully detected scanner-to-target traffic. |

## Recommended Response

* Identify the source IP address.
* Confirm whether the traffic was authorized.
* Review Suricata alerts for scan or suspicious network activity.
* Check whether the source attempted connections to multiple ports.
* Correlate Suricata alerts with endpoint logs.
* Block unauthorized scanning sources if needed.
* Improve monitoring by forwarding Suricata logs into Wazuh.

## Limitations

Suricata successfully detected scanner-to-target traffic using custom lab rules.

Wireshark captured host-only lab traffic between the Windows host and Ubuntu-Test-Machine, but the screenshot did not show Ubuntu-Scanner traffic directly. The scanner-to-target evidence was confirmed through Suricata `fast.log`.

Future improvements:

* Forward Suricata `eve.json` logs into Wazuh.
* Create more specific Suricata rules for Nmap scan behavior.
* Capture scanner-to-target traffic directly in Wireshark from the sensor VM or correct virtual interface.
* Add Suricata alert screenshots from Wazuh after log forwarding is configured.

## Lessons Learned

This test showed the value of network-based detection.

Endpoint logs are useful for authentication and command activity, but network IDS tools like Suricata provide stronger visibility into traffic patterns such as scans, pings, and TCP connection attempts.

This also showed that SOC evidence is stronger when multiple sources are used together, such as Suricata logs, endpoint logs, Wazuh events, and Wireshark packet captures.