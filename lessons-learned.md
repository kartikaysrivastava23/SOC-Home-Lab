# Lessons Learned

## Technical Lessons

- Wazuh can collect and centralize security events from a monitored Linux endpoint.
- Linux authentication logs are useful for detecting failed SSH login attempts.
- Wazuh provided clear visibility for failed SSH login activity and sudo command execution.
- Basic Nmap scans may not always be clearly classified by Wazuh without additional log sources or custom rules.
- UFW firewall logs can provide useful evidence of blocked scan traffic.
- Suricata provides stronger network-based visibility for scanner-to-target traffic.
- Wireshark is useful for confirming packet-level activity on the lab network.
- ClamAV can detect the EICAR safe test file, but Wazuh needs additional configuration to clearly display that detection.
- Wazuh can collect Windows Security Event Logs from a monitored Windows endpoint.
- Windows Security Event ID `4625` is useful for identifying failed login activity.

## SOC Lessons

- A SIEM does not automatically detect every suspicious action without the correct logs and rules.
- Multiple evidence sources are important during investigation.
- Endpoint logs, firewall logs, Suricata alerts, and Wireshark captures each provide different types of visibility.
- Not every suspicious command is automatically malicious; context matters.
- Incident reports should clearly explain the activity, evidence, impact, limitations, and recommended response.

## Challenges Faced

- Wazuh did not clearly classify the Nmap scan during the first test.
- UFW firewall logs were useful locally, but did not clearly appear in Wazuh Threat Hunting.
- Suricata required custom lab rules to produce clear detection evidence.
- Wireshark captured host-only traffic, but the clearest scanner-to-target evidence came from Suricata logs.
- The EICAR test was detected by ClamAV, but not clearly displayed as a Wazuh malware event.

## Improvements Made

- Added a separate Ubuntu Scanner VM for more realistic testing.
- Used UFW firewall logging to improve scan evidence.
- Added Suricata for network-based detection.
- Used Wireshark to support packet-level traffic analysis.
- Documented limitations honestly instead of claiming false detections.
- Added a Windows Endpoint VM and collected Windows failed login events using Wazuh.
- Added a final architecture diagram showing Linux, Windows, scanner, and Wazuh components.

## Future Improvements

- Forward Suricata `eve.json` logs into Wazuh.
- Configure Wazuh to collect and parse UFW firewall logs more clearly.
- Configure Wazuh to collect ClamAV scan results.
- Create custom Wazuh rules for Nmap scan patterns, UFW blocks, ClamAV detections, and Windows failed login activity.
- Add Sysmon to the Windows Endpoint for deeper Windows process and event visibility.
- Create a small incident response playbook.

## Final Reflection

This project helped me understand how a small SOC-style lab works in practice.

I learned how to set up a SIEM, enroll a Linux endpoint, generate safe security events, review logs, analyze alerts, and write incident reports. I also learned that detection depends heavily on correct log collection, rules, and context.

The project improved my practical understanding of Linux logs, Windows Security Event Logs, Wazuh, Nmap, SSH authentication events, Suricata, Wireshark, firewall logs, and basic incident response documentation.