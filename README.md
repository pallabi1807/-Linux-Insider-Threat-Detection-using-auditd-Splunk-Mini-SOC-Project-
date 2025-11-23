Linux Insider Threat Detection Using Auditd + Splunk

Beginner-Friendly Mini SOC Project | Real-World Linux Monitoring

This project demonstrates how to detect insider threat behaviors on a Linux system using Auditd for event collection and Splunk for analysis, alerting, and dashboards.
Designed as a beginner SOC Analyst project, it covers monitoring for:

Unauthorized access to sensitive files
Sudo misuse
Local user creation or deletion
Bash history tampering
Log deletion or manipulation
___________________________

📌 1. Why This Project Matters

Insider threats are one of the fastest-growing security risks.
As a beginner, this project helped me learn:

✔ How Linux logs work (auditd)
✔ How endpoint events are collected and forwarded
✔ How SIEM searches, dashboards, and alerts are created
✔ How MITRE ATT&CK techniques map to detections
✔ How SOC teams detect misuse, tampering, unauthorized access

This is a real-world, practical project showcasing SOC monitoring skills.
___________________________________
🏗 2. Project Architecture
Linux Host (Auditd) 
     → (Audit Logs)
         → Splunk Universal Forwarder 
                → Splunk Enterprise (Index=main)
                       → Detection Searches 
                       → Alerts 
                       → Dashboards
______________________________________
⚙️ 3. Auditd Installation (Linux)
sudo apt update
sudo apt install auditd audispd-plugins -y
sudo systemctl enable auditd
sudo systemctl start auditd

Verify:
sudo auditctl -s

_________________________________

🛡 4. Auditd Configuration
4.1 auditd.conf 

Create file:
auditd/auditd.conf

log_file = /var/log/audit/audit.log
log_format = RAW
flush = INCREMENTAL
freq = 20
num_logs = 5
max_log_file = 20
max_log_file_action = ROTATE
space_left = 75
space_left_action = SYSLOG
admin_space_left = 50
admin_space_left_action = SUSPEND
disk_full_action = SUSPEND
disk_error_action = SUSPEND

4.2 insider.rules (GitHub version)
Create file:
auditd/insider.rules

## Sensitive File Monitoring
-w /etc/passwd -p wa -k auth_changes
-w /etc/shadow -p wa -k auth_changes
-w /etc/group  -p wa -k auth_changes

## Sudo Monitoring
-w /etc/sudoers -p wa -k sudo_changes
-w /etc/sudoers.d -p wa -k sudo_changes
-w /usr/bin/sudo -p x -k sudo_exec

## Bash History & Shell Config
-w /home -p wa -k home_mods
-w /root/.bash_history -p wa -k bash_history

## Audit Log Tampering
-w /var/log/audit/ -p wa -k audit_logs

## User Management Commands
-a always,exit -F arch=b64 -S execve -F path=/usr/sbin/useradd -k user_mgmt
-a always,exit -F arch=b64 -S execve -F path=/usr/sbin/userdel -k user_mgmt

## /etc Permission Changes
-w /etc/ -p wa -k etc_mods

Load rules:
sudo augenrules --load
sudo systemctl restart auditd

_______________________
🔄 5. Splunk Universal Forwarder Configuration
inputs.conf

File: splunk/inputs.conf
[monitor:///var/log/audit/audit.log]
disabled = false
sourcetype = linux:audit
index = main

outputs.conf
File: splunk/outputs.conf

[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 127.0.0.1:9997
(Replace IP if Splunk Enterprise is on another system)
_______________________________
🔍 6. Splunk Saved Searches (Detection Rules)
A. Sensitive File Modification
splunk/savedsearches/sensitive_file_modification.spl
index=main sourcetype=linux:audit key=auth_changes

B. Sudo Misuse
splunk/savedsearches/sudo_misuse.spl
index=main sourcetype=linux:audit (exe="/usr/bin/sudo" OR key=sudo_exec)

C. User Creation / Deletion
splunk/savedsearches/user_create-del.spl
index=main sourcetype=linux:audit (exe="/usr/sbin/useradd" OR exe="/usr/sbin/userdel")

D. Bash History Tampering
splunk/savedsearches/bash_history_tamper.spl
index=main sourcetype=linux:audit key=bash_history

E. Audit Log 
splunk/savedsearches/audit_log_deletion.spl
index=main sourcetype=linux:audit key=audit_logs

____________________________________
🧪 7. Attack Test Cases
Folder: /attacks/

✔ useradd_test.txt
sudo useradd test123
Expected detection: auditd rule logs execve on /usr/sbin/useradd
MITRE: T1136.001 — Create Local Account

userdel_test.txt
sudo userdel test123
Expected detection: auditd rule logs execve on /usr/sbin/userdel
MITRE: T1531 — Account Removal

✔ sudo_bruteforce.txt
sudo -k ; for i in {1..5}; do sudo ls; done
Expected detection: sudo_misuse
MITRE: T1078.003 — Privileged Access Abuse

✔ shadow_edit.txt
sudo nano /etc/shadow
Expected detection: auth_changes
MITRE: T1098 — Account Manipulation

✔ bash_history_tamper.txt
echo "" > ~/.bash_history
Expected detection: bash_history
MITRE: T1562.001 — Log Tampering

__________________________
🖥 8. Dashboard
XML dashboard file:
dashboard/insider-threat-overview.xml
Contains panels for: 
Sensitive file modifications
Sudo activity
New user creation
Bash history tampering
Log deletion

_____________________
🧩 9. MITRE ATT&CK Mapping Table
| Detection                   | MITRE Technique                  | ID        |
| --------------------------- | -------------------------------- | --------- |
| Sensitive file modification | Account Manipulation             | T1098     |
| Sudo misuse                 | Valid Accounts: Privileged Abuse | T1078.003 |
| New user creation           | Create Local Account             | T1136.001 |
| User deletion               | Account Removal                  | T1531     |
| Bash history tampering      | Log Tampering                    | T1562.001 |
| Audit log deletion          | Inhibit System Recovery          | T1490     |

🚀 10. What I Learned
How Linux audit logs work
How to write detection rules using auditd
How Splunk forwarders send data
How SIEM searches, dashboards, and alerts are built
MITRE ATT&CK mapping for real SOC workflows
How to simulate attacks safely in a lab

_________________
🧠 11. Performance & Safety Notes
Auditd adds load if too many syscalls monitored
Avoid watching large directories like /home fully
Monitor audit queue depth using:
sudo auditctl -s
If logs drop, tune audit kernel parameters

_________________________
🔮 12. Future Enhancements
Planned improvements:
Integrate Sysmon for Linux
Add behavioural anomaly dashboards
Add machine-learning based insider threat scoring
Send alerts to Slack or Microsoft Teams
Add threat intelligence correlation
Build full MITRE ATT&CK Navigator layer

______________________
👤 Author

Pallabi Poria

Beginner SOC Analyst | Cybersecurity Student
