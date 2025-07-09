# Day 5–6: Linux Log Analysis + OpenSearch Troubleshooting

## Objectives

Analyze `auth.log` for login activity  
Analyze `syslog` for service and error events  
Use `journalctl` for timeline-based review  
Investigate and troubleshoot `opensearch-dashboards` error

---

## Analyze `/var/log/auth.log`

---

### Findings:
Detected failed SSH login attempts from local and external IPs

Verified successful login as user richard

Observed sudo command executions with timestamp, TTY, and command path

---

### Additional Findings:
When looking through the syslogs, I found that the OpenSearch Dashboards app was attempting to connect to OpenSearch but was being refused
Traced the error with:
```bash
sudo journalctl -u opensearch-dashboards
```
Also was noted that the UFW was able to see the connection and was auditing, but not blocking the connection
Found out OpenSearch was not installed with 
```bash
which opensearch
```
verified service was not installed with
```bash
sudo systemctl status opensearch
```
I was having issues installing Wazuh and for the life of me could not figure out why it would not run. Turns out since OpenSearch was never fully installed, it broke Wazuh's backend indexing and caused opesearch-dashboards to throw an ECONNREFUSED error

---

### Planned Fix: Install Wazuh with OpenSearch (All-in-One)
Command Sequence:
```basg
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
chmod +x wazuh-install.sh
sudo ./wazuh-install.sh -a
```

This installs:

Wazuh Manager

OpenSearch

OpenSearch Dashboards

All necessary plugins

---

### Takeaways
journalctl and syslog were essential in uncovering service-level failures

The issue was not firewall-related (confirmed via [UFW AUDIT])

Missing OpenSearch caused all dashboard and Wazuh-related errors

Learning how to troubleshoot service communication is a core Blue Team skill

### Tools & Commands Used
| Tool/Command         | Purpose                                |
|----------------------|----------------------------------------|
| journalctl           | View service logs in chronological order |
| grep, cat, less      | Parse and read text logs               |
| ss -ltnp, curl       | Check for open/listening ports         |
| systemctl            | Check and manage service state         |
| dpkg, which          | Confirm package installation           |
