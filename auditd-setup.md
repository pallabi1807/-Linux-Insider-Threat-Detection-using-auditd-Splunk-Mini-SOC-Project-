Auditd Installation & Setup (Ubuntu/Debian)
1. Install Auditd
sudo apt update
sudo apt install auditd audispd-plugins -y

2. Enable & Start Auditd
sudo systemctl enable auditd
sudo systemctl start auditd
sudo systemctl status auditd

3. Copy custom rules

Place your rules file:
sudo cp auditd/insider.rules /etc/audit/rules.d/

4. Load rules
sudo augenrules --load

5. Verify Active Rules
sudo auditctl -l

6. Check audit log
sudo tail -n 50 /var/log/audit/audit.log
