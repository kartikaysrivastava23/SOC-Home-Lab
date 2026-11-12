# Incident Report: Windows Failed Login Detection

## Summary

A Windows failed login detection test was performed inside the private SOC Home Lab environment.

A Windows 11 Enterprise Evaluation VM was created, connected to the host-only lab network, enrolled as a Wazuh agent, and used to generate failed login events. The failed login activity was confirmed locally in Windows Security logs and centrally in the Wazuh dashboard.

## Lab Environment

| Machine          | Role                       | IP Address    |
|------------------|----------------------------|---------------|
| Wazuh Server     | SIEM/XDR server            | 192.168.56.10 |
| Windows Endpoint | Monitored Windows endpoint | 192.168.56.30 |

## Activity Performed

The Windows Endpoint VM was installed and configured with two network adapters:

| Adapter           | Purpose                    | IP Address    |
|-------------------|----------------------------|---------------|
| NAT adapter       | Internet access            | 10.0.2.15     |
| Host-only adapter | Internal lab communication | 192.168.56.30 |

The Windows Endpoint was able to communicate with the Wazuh Server:

```cmd
ping 192.168.56.10
```

The Wazuh agent was installed on the Windows Endpoint and started successfully.

The Windows Endpoint appeared in the Wazuh dashboard as an active agent:

```text
Agent name: windows-endpoint
Agent ID: 002
IP address: 192.168.56.30
Operating system: Microsoft Windows 11 Enterprise Evaluation
Wazuh agent version: 4.14.5
Status: Active
```

## Failed Login Test

Windows failed logon auditing was enabled using:

```cmd
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

A local test user was created:

```cmd
net user testuser * /add
```

Failed login attempts were generated using incorrect credentials for the test user.

## Local Windows Evidence

The Windows Security log showed failed login events with Event ID `4625`.

Key details:

```text
Event ID: 4625
Provider: Microsoft-Windows-Security-Auditing
Message: An account failed to log on
Account Name: testuser
Failure Reason: Unknown user name or bad password
```

## Wazuh Evidence

The Wazuh dashboard showed Windows failed login events from the `windows-endpoint` agent.

Key details:

```text
Agent name: windows-endpoint
Agent ID: 002
Agent IP: 192.168.56.30
Rule description: Logon Failure - Unknown user or bad password
Rule level: 5
Rule ID: 60122
```

The event was searchable using:

```text
4625
testuser
```

## Evidence Screenshots

| Screenshot                                                                                                                             | Description                                            |
|:---------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------|
| [01-windows-vm-settings.png](../screenshots/day-10-windows-endpoint-logs/01-windows-vm-settings.png)                                   | Windows VM configuration                               |
| [02-windows-desktop-running.png](../screenshots/day-10-windows-endpoint-logs/02-windows-desktop-running.png)                           | Windows desktop after installation                     |
| [03-windows-network-verification.png](../screenshots/day-10-windows-endpoint-logs/03-windows-network-verification.png)                 | Windows IP configuration and Wazuh Server connectivity |
| [04-windows-agent-deploy-command.png](../screenshots/day-10-windows-endpoint-logs/04-windows-agent-deploy-command.png)                 | Wazuh Windows agent deployment command                 |
| [05-windows-wazuh-agent-running.png](../screenshots/day-10-windows-endpoint-logs/05-windows-wazuh-agent-running.png)                   | Wazuh service running on Windows                       |
| [06-windows-agent-active-in-wazuh.png](../screenshots/day-10-windows-endpoint-logs/06-windows-agent-active-in-wazuh.png)               | Windows endpoint active in Wazuh                       |
| [07-windows-failed-login-local-event.png](../screenshots/day-10-windows-endpoint-logs/07-windows-failed-login-local-event.png)         | Local Windows Security Event ID 4625                   |
| [08-windows-failed-login-wazuh-event.png](../screenshots/day-10-windows-endpoint-logs/08-windows-failed-login-wazuh-event.png)         | Wazuh failed login event for Windows endpoint          |
| [09-windows-failed-login-testuser-search.png](../screenshots/day-10-windows-endpoint-logs/09-windows-failed-login-testuser-search.png) | Wazuh search results for 'testuser'                    |

## Analysis

The failed login attempts were successfully detected both locally on the Windows Endpoint and centrally in Wazuh.

This confirms that Wazuh can collect and display Windows Security Event Log data from an enrolled Windows endpoint.

Compared to Linux authentication logs, Windows events provide structured event IDs such as `4625`, which makes searching and filtering failed login activity easier.

## Severity

Medium

## MITRE ATT&CK Mapping

| Tactic            | Technique                     |
|-------------------|-------------------------------|
| Credential Access | T1110.001 – Password Guessing |

## Verdict / Triage Decision

| Field             | Value                                                                                                                                                                                             |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Classification    | True Positive                                                                                                                                                                                     |
| Activity Type     | Authorized lab simulation                                                                                                                                                                         |
| Detection Outcome | Confirmed through Windows Security Event ID `4625` and Wazuh alerts                                                                                                                               |
| Status            | Closed                                                                                                                                                                                            |
| Escalation        | Not escalated                                                                                                                                                                                     |
| Reason            | The failed login attempts were intentionally generated using a local test user inside the private Windows VM. The activity was confirmed locally in Windows Security logs and centrally in Wazuh. |

## Recommended Response

* Identify the affected endpoint.
* Review the failed username.
* Confirm whether the login attempts were expected or unauthorized.
* Check for repeated failed login attempts from the same source.
* Review related Windows Security events.
* If suspicious, disable or lock affected accounts and investigate source activity.
* Continue monitoring for repeated authentication failures.

## Limitations

This was a controlled lab test using a local Windows VM and a test user.

The failed login activity was generated safely inside the private lab environment. No real user accounts or external systems were targeted.

## Lessons Learned

This test showed how Windows endpoint logs can improve SOC visibility.

Wazuh successfully collected Windows failed login events and displayed them in the dashboard. This added endpoint diversity to the lab by including both Linux and Windows log sources.