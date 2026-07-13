# Home SOC Lab: Wazuh SIEM + pfSense + Windows Hardening

## Description
This project documents the implementation of a complete security monitoring ecosystem. The objective was to integrate advanced Windows endpoint telemetry (via Sysmon) and perimeter network logs (via pfSense) into a centralized Wazuh SIEM server.

---

### Detailed PDF Report
#### You can consult the complete documentation with all steps, screenshots, and troubleshooting here:[home_lab_SOC_report.pdf](https://github.com/user-attachments/files/29964565/home_lab_SOC_report.pdf)



---

## Architecture and Network
The lab was isolated in a local network segment to simulate a corporate environment:
*   **Wazuh Manager:** Ubuntu 24.04 LTS (IP: 10.0.0.101)
*   **Firewall/Gateway:** pfSense (WAN + LAN Segment - IP: 10.0.0.1)
*   **Endpoint:** Windows 11 Enterprise (Wazuh Agent + Sysmon)

---

## Implementation Journey

### 1. Wazuh Server (SIEM)
*   **Challenge:** Initial installation attempt on Ubuntu 26.04 (incompatibility detected) and "Low Disk Space" error.
*   **Solution:** Downgrade to Ubuntu 24.04 and virtual disk expansion to **70GB**.
*   **Configuration:** Activation of the **Vulnerability Detection** module and `ossec.conf` configuration to accept remote logs (Syslog port 514).

### 2. Windows 11 & Sysmon Telemetry
*   **Hardening:** Sysmon installation using the **SwiftOnSecurity** reference configuration.
*   **System Bypass:** Used Regedit (`LabConfig`) to bypass Windows 11 TPM and RAM requirements in a virtual environment.
*   **Troubleshooting:** Diagnosis of instability (BSOD) after Sysmon installation, resolved by increasing RAM and CPU resources in the VM.

### 3. pfSense Integration (Network Visibility)
*   Syslog configuration to export system and firewall logs to Wazuh.
*   **Rule Logic:** Identification of rule hierarchy (Native rule 87701 vs. Custom rule 100010).

---

## Detection Intelligence (Custom Rules)

### Windows Log Clearing Detection
Identifies when a user attempts to erase activity traces using `wevtutil`.

```xml
<rule id="100002" level="12">
  <if_group>windows</if_group>
  <match>wevtutil</match>
  <description>SOC ALERT: Log clearing tool detected!</description>
  <mitre><id>T1070.001</id></mitre>
</rule>
```
<img width="857" height="296" alt="image" src="https://github.com/user-attachments/assets/eba25bae-a28a-4998-9b0d-7a9e405df716" />

## pfSense Log Normalization
Forces visibility of logs that were initially ignored by the analysis engine.

```xml
<group name="pfsense_custom,">
  <rule id="100010" level="5">
    <if_sid>1002</if_sid>
    <match>syslog|pfsense|filterlog|auth.notice</match>
    <description>pfSense: Activity detected now</description>
  </rule>
</group>
```
<img width="999" height="451" alt="image" src="https://github.com/user-attachments/assets/89553da2-da57-4802-a6ba-e8a161cd6723" />

## Lessons Learned (Troubleshooting)
1. Time Synchronization: Time discrepancies between machines silently cause logs to be discarded. NTP is mandatory for reliable log ingestion.
2. Field Mapping vs. Global Search: Initially, I tried using the <field> tag with the data.win.eventdata prefix, but the rule failed to trigger. I learned that for lab purposes and rapid troubleshooting, the <match> tag is more efficient as it performs a global search across the raw log, bypassing potential field mapping failures.
3. Active Response: IP blocking automation (firewall-drop) transforms the SIEM from a passive observer into a powerful reactive security tool.

   
## Lab Evidence

<img width="1688" height="774" alt="image" src="https://github.com/user-attachments/assets/0ce13234-a466-43a0-a211-6e102c710986" />
<img width="1714" height="706" alt="image" src="https://github.com/user-attachments/assets/bd4e8d22-0b1e-4dae-8d60-477ea36cfb26" />
<img width="1542" height="783" alt="image" src="https://github.com/user-attachments/assets/4f06d206-8238-4d9d-a94d-29655ff7712c" />
<img width="1706" height="719" alt="image" src="https://github.com/user-attachments/assets/2a36d7b0-854b-4835-896f-4daf9fdaa7c0" />

