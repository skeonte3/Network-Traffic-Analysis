# Network Traffic Analysis with Wireshark

## Project Overview
**Tool:** Wireshark  
**Environment:** Home network (Windows 11, Wi-Fi interface)  
**Objective:** Capture and analyze live network traffic to identify active protocols, assess encryption posture, and detect anomalous patterns using the same methodology employed by SOC analysts in enterprise environments.

---

## Methodology
1. Launched Wireshark and selected the active Wi-Fi interface
2. Captured live traffic while browsing the web
3. Applied display filters to isolate specific protocols (DNS, HTTP, TCP RST)
4. Analyzed each finding and documented security implications
5. Saved capture file as `.pcap` for further analysis

---

## Findings

### Finding 1 — DNS Traffic Analysis
**Filter used:** `dns`

![DNS Filter] <img width="1920" height="1080" alt="dns-filter png" src="https://github.com/user-attachments/assets/2f356b20-ccb0-4bde-bac1-22d6d14a29ea" />

**Domains observed:**
- google.com, play.google.com (Google services)
- msedge.net, azureedge.net (Microsoft Edge / Azure CDN)
- footprintdns.com (Microsoft CDN infrastructure)
- windowsupdate.com (Windows Update service)

**Analysis:**  
All resolved domains belong to known legitimate services (Google, Microsoft). No suspicious or unknown domains were detected in DNS queries. In a real SOC environment, analysts monitor DNS traffic for domains associated with command-and-control (C2) servers, domain generation algorithms (DGA), or newly registered domains which are common indicators of malware activity.

**Security Implication:**  
Clean DNS traffic with no indicators of compromise. However, DNS tunneling attacks can hide malicious traffic within normal-looking DNS queries — future analysis could include checking for abnormally large DNS payloads or high-frequency queries to a single domain.

---

### Finding 2 — HTTP Traffic Analysis
**Filter used:** `http`  
**Result:** Only 4 unencrypted HTTP packets detected out of 16,000+ total packets  
**Destination:** ctld1.windowsupdate.com (Windows Update)

**Analysis:**  
The extremely low volume of unencrypted HTTP traffic indicates strong encryption posture. The majority of traffic was encrypted via HTTPS (TLS 1.2/1.3) and QUIC. The only HTTP traffic observed was from the Windows Update service which is expected and benign.

**Security Implication:**  
Good security posture — encrypted traffic prevents man-in-the-middle (MITM) attacks and eavesdropping. In an enterprise SOC environment, any unencrypted HTTP traffic carrying sensitive data would be flagged as a finding and reported.

---

### Finding 3 — TCP Reset (RST) Analysis
**Filter used:** `tcp.flags.reset==1`  
**Source IP:** 99.84.237.54  
**Destination:** 192.168.1.227 (local machine)  
**Port:** 443

**Analysis:**  
A high volume of TCP RST packets were observed originating from 99.84.237.54, which resolves to AWS CloudFront — Amazon's content delivery network. RST packets indicate abrupt connection terminations. In this case the volume is consistent with a browser or streaming service dropping idle connections through a CDN.

**Security Implication:**  
In an enterprise environment, a high volume of RST packets from an external IP would trigger an alert and require investigation. Possible malicious causes include port scanning, firewall blocking, or a remote server rejecting connections. In this case the source IP is a known legitimate CDN so no threat is indicated. An analyst would cross-reference the IP against threat intelligence feeds like VirusTotal or AbuseIPDB to confirm.

---

## Overall Assessment

| Category | Finding | Risk Level |
|----------|---------|------------|
| DNS Traffic | All domains legitimate, no C2 indicators | Low |
| HTTP Traffic | Minimal unencrypted traffic, Windows Update only | Low |
| TCP RSTs | High volume from AWS CloudFront, consistent with CDN behavior | Low |
| Encryption Posture | 99%+ of traffic encrypted via TLS/QUIC | Positive |

**Conclusion:** No indicators of compromise (IOCs) detected. Network traffic consistent with normal home user activity. Encryption posture is strong. Recommended follow-up would include monitoring DNS queries over time for anomalies and running this analysis against a pcap file containing known malicious traffic for practice.

---

## Tools Used
- Wireshark 4.x
- Display filters: `dns`, `http`, `tcp.flags.reset==1`

## Skills Demonstrated
- Network packet capture and analysis
- Protocol identification (DNS, HTTP, TCP, QUIC, TLS)
- Threat assessment and IOC identification
- Security documentation and reporting
- SOC analyst methodology
