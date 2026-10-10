# Network Intrusion Detection System (NIDS) Using Suricata

**Detect. Alert. Respond.**

A cybersecurity project developed as part of my Cybersecurity Internship with CodeAlpha.

**Author:** Agwasie Blessing Uchechi  
**Role:** Cybersecurity Intern — CodeAlpha  
**Project Type:** Network Security and Intrusion Detection


## 📌 Project Overview

A Network Intrusion Detection System (NIDS) monitors network traffic, looks for suspicious or malicious activity, and raises alerts so that threats can be investigated and stopped.

This project sets up a NIDS using **Suricata** on **Kali Linux** running in Oracle VirtualBox. It detects suspicious activity with custom rules, automatically blocks offending IP addresses, and summarizes alerts in a simple dashboard.

## 🎯 Project Objectives

The objectives of this project are to:

- Install and configure Suricata as a network-based intrusion detection system.
- Write custom rules to detect suspicious network activity.
- Monitor network traffic continuously for potential threats.
- Generate and analyze alerts from Suricata logs.
- Implement an automated response to block malicious IP addresses.
- Visualize detected alerts using a dashboard.

## 🛠️ Tools and Technologies

- Kali Linux
- Suricata 8.0.7
- Oracle VirtualBox
- Nmap
- Python 3
- iptables

## 📁 Project Structure

```
CodeAlpha_ProjectName/
├── README.md
├── local.rules          # Custom Suricata detection rules
├── ids_response.py      # Automated response script
├── dashboard.py         # Alert dashboard generator
└── screenshots/         # Evidence of each step
```

## 📚 Implementation

### 1. Installation

```bash
Sudo apt update
Sudo apt install suricata -y
```

### 2. Configuration

- Configured Suricata to monitor the network interface `eth0`.
- Set the protected network (`HOME_NET`) to `10.0.0.0/24`.
- Enabled `local.rules` under `rule-files` in `suricata.yaml`.
- Validated the configuration:

```bash
Sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

Result: 383 rules loaded from 2 rule files, with 0 failed and 0 skipped.

### 3. Custom Detection Rules

**ICMP Ping Detection**

```
Alert icmp any any -> any any (msg:”ICMP ping detected”; itype:8; sid:1000001; rev:1;)
```

Detects ICMP echo requests.

**Possible Port Scan Detection**

```
Alert tcp any any -> $HOME_NET any (msg:”Possible port scan detected”; flags:S; threshold:type threshold, track by_src, count 10, seconds 5; sid:1000002; rev:1;)
```

Detects repeated TCP SYN packets from one source (10 within 5 seconds), which may indicate port scanning.

### 4. Continuous Monitoring

Start Suricata on the network interface:

```bash
Sudo suricata -c /etc/suricata/suricata.yaml -i eth0 -D
```

Watch alerts in real time:

```bash
Sudo tail -f /var/log/suricata/fast.log
```

### 5. Testing

- Generated ICMP traffic using `ping`.
- Ran an Nmap SYN scan against the lab gateway: `nmap -sS 10.0.0.1`
- Confirmed in `fast.log` that the custom rules generated the expected alerts.

### 6. Automated Response

`ids_response.py` follows `fast.log` in real time. When it detects a port scan alert, it blocks the source IP address:

```bash
Iptables -I INPUT -s <attacker_ip> -j DROP
```

Safety features:

- **Whitelist:** the gateway (`10.0.0.1`), the host itself (`10.0.0.2`) and localhost are never blocked.
- **No duplicate blocks:** each IP is blocked only once per run.
- **Audit trail:** every block is recorded with a timestamp in `/var/log/ids_response.log`.

Run the script:

```bash
Sudo python3 ids_response.py
```

Testing: a simulated alert from `10.0.0.99` was added to `fast.log`. The script printed `BLOCKED 10.0.0.99`, and `sudo iptables -L INPUT -n` showed a `DROP` rule for that address. The test block was then removed:

```bash
Sudo iptables -D INPUT -s 10.0.0.99 -j DROP
```

### 7. Alert Dashboard

`dashboard.py` reads `fast.log` and creates an HTML dashboard with bar charts showing:

- Alerts by source IP
- Alerts by alert type
- Alerts by target IP
- Alerts by hour

```bash
Sudo python3 dashboard.py
Firefox /tmp/dashboard.html
```

Suricata also writes structured alerts to `eve.json`, which could feed a fuller dashboard such as Kibana or Grafana in the future.

##  Results

- Suricata configuration validated successfully.
- ICMP ping detection tested successfully.
- Port scan detection tested successfully.
- Automated IP blocking tested successfully using a simulated attacker.
- Alert dashboard generated from real alert data.

##  Challenges

- Troubleshooting Suricata configuration errors.
- Configuring the correct network interface.
- Locating the custom rules file (`/var/lib/suricata/rules/local.rules`).
- Writing and validating custom detection rules and thresholds.
- Recovering the lab environment after an unexpected power-off.
- Preventing the response script from blocking trusted hosts.

##  Limitations and Future Work

- The response script was tested with a simulated alert rather than a live external attacker.
- Blocking is based on a single rule and a simple log parser.
- Future improvements: a Kibana or Grafana dashboard, email or Slack notifications, and running the response script as a system service.

##  Key Learnings

Through this project, I strengthened my understanding of:

- Network monitoring and intrusion detection concepts.
- Writing custom Suricata rules and detection thresholds.
- Reading and analyzing alert logs.
- Traffic analysis with Nmap and Suricata.
- Automated incident response using Python and iptables.

##  Conclusion

Intrusion detection is a key part of network defense because it gives early warning of attacks such as port scans before they develop into serious incidents.

This project demonstrates how a NIDS can detect suspicious traffic, alert on it, and automatically respond to it in a controlled lab environment.


## 👤 Author

**Agwasie Blessing Uchechi**  
Cybersecurity Intern — CodeAlpha


*This project was developed in a controlled lab environment for educational purposes as part of my CodeAlpha internship. Only scan or test systems you own or have explicit permission to test.*


