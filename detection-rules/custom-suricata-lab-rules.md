# Custom Suricata Lab Rules

## Purpose

This file documents the custom Suricata rules used during the SOC Home Lab network detection test.

The rules were created for lab testing only. They are not production-grade detection rules.

## Lab Context

| Machine             | Role                     | IP Address    |
|---------------------|--------------------------|---------------|
| Ubuntu-Scanner      | Traffic source / scanner | 192.168.56.40 |
| Ubuntu-Test-Machine | Target / Suricata sensor | 192.168.56.20 |

Suricata was installed on the Ubuntu-Test-Machine and monitored the host-only lab interface:

```text
enp0s8
```

## Rules Used

```text
alert icmp 192.168.56.40 any -> 192.168.56.20 any (msg:"LAB ICMP ping from Ubuntu Scanner"; sid:1000001; rev:1;)

alert tcp 192.168.56.40 any -> 192.168.56.20 any (msg:"LAB TCP traffic from Ubuntu Scanner"; sid:1000002; rev:1;)
```

## Rule Explanation

### ICMP Rule

This rule detects ICMP traffic from the Ubuntu-Scanner to the Ubuntu-Test-Machine.

It was triggered when the scanner ran:

```text
ping -c 4 192.168.56.20
```

### TCP Rule

This rule detects TCP traffic from the Ubuntu-Scanner to the Ubuntu-Test-Machine.

It was triggered when the scanner ran:

```text
sudo nmap -sS -Pn 192.168.56.20
```

## Detection Result

Suricata generated alerts in `fast.log`.

Example alert messages:

```text
LAB ICMP ping from Ubuntu Scanner
LAB TCP traffic from Ubuntu Scanner
```

These alerts confirmed traffic from:

```text
192.168.56.40 -> 192.168.56.20
```

## Limitations

These rules are intentionally simple and specific to the lab IP addresses.

They are useful for proving that Suricata can detect traffic in the lab, but they are not advanced Nmap detection rules.

Future improvements:

* Create more specific Suricata rules for scan behavior.
* Forward Suricata eve.json logs into Wazuh.
* Create Wazuh rules for Suricata alerts.
* Test detection using default Emerging Threats rules.