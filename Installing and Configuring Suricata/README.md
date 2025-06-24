### Installing Suricata IDS on victim VM

🔧 Step 1: Install Suricata
Run the following commands to install Suricata from the official stable PPA:

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```

📥 Step 2: Download and Apply Emerging Threats Rules

Fisrt make a directory for rules inside `/etc/suricata` 

```
sudo mkdir /etc/suricata/rules
```

Download the official open-source rule set and move it to the Suricata rules directory:

```bash
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz
sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

⚙️ Step 3: Configure Suricata

Edit the Suricata configuration file to set your local network and rule paths:

```yaml
# /etc/suricata/suricata.yaml

vars:
  HOME_NET: "<UBUNTU_IP>"         # Replace with your VM's IP
  EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"

stats:
  enabled: yes
 
pcap-log:
      enabled: yes

af-packet:
  - interface: eth0               # Use your actual network interface (e.g., enp0s3 if on VirtualBox)
```

💡 Tip: Use ip a to check your interface name.

🔁 Step 4: Restart Suricata
Apply the changes by restarting the Suricata service:

```bash
sudo systemctl restart suricata
```
Once installed and configured, Suricata will begin monitoring traffic and generating alerts based on the loaded rules.

## 🔗 Integrating Suricata with Wazuh

To allow Wazuh to monitor and analyze Suricata alerts, follow these steps:

---

### 👨‍💻 Step 1: Switch to Root User

```bash
sudo su
```

📂 Step 2: Navigate to Wazuh Agent Configuration Directory

```bash
cd /var/ossec/etc/
```

![image](https://github.com/user-attachments/assets/f090d245-03fb-4eca-b5a3-72ba74bcbe98)

📝 Step 3: Edit the ossec.conf File

Open the ossec.conf file in a text editor (e.g., nano or vim):

```bash
nano ossec.conf
```
Scroll to the bottom and add the following configuration block inside the <ossec_config> section to include Suricata’s JSON alert log:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

![image](https://github.com/user-attachments/assets/f9391de0-adb0-439a-a27b-e4387ca94443)

💡 Make sure this block is placed inside the <ossec_config> root element and not within any comments.

🔁 Step 4: Restart the Wazuh Agent

Apply the changes by restarting the Wazuh agent:

```bash
sudo systemctl restart wazuh-agent
```

✅ Result
After restarting, Wazuh will begin ingesting Suricata alerts from the eve.json log file. You should start seeing Suricata detections in the Wazuh Dashboard under Security Events.


🛡️ Congratulations! Suricata is now successfully integrated with Wazuh for enhanced real-time intrusion detection.




