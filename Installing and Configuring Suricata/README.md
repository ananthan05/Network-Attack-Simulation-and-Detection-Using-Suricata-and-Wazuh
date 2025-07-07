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

```
sudo gedit  /etc/suricata/suricata.yaml
```

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
![image](https://github.com/user-attachments/assets/93b8b295-f691-47d8-97c5-77c6d1ab9f1d)

![image](https://github.com/user-attachments/assets/9683488a-2da3-4941-a11c-6c1da08d1d2a)

![image](https://github.com/user-attachments/assets/964d4fa3-e68c-43ea-b469-451d407ca5c1)

add the rules we downloaded and the path `/etc/suricata/rules` also like this 
```
"*.rules"
```
![image](https://github.com/user-attachments/assets/0a30a610-8658-4836-80ce-b8f297071de8)

And update all the configurations as shown in the screenshot below.

![image](https://github.com/user-attachments/assets/1c4f807d-b575-4ea9-ae0e-10dbf5e19d3a)

![image](https://github.com/user-attachments/assets/2984a71c-a0fe-4f84-9288-4d0ce053d025)

![image](https://github.com/user-attachments/assets/06b448b6-ad4d-4b91-8c7c-a177a59fcc69)

![image](https://github.com/user-attachments/assets/ddde1923-8ad2-4337-9208-94bcc556e9ea)

![image](https://github.com/user-attachments/assets/a9a7baed-ea0f-4fb4-ad38-b1d936a0c311)


💡 Tip: Use ip a to check your interface name.

checks whether Suricata’s configuration file (suricata.yaml) and all related rule/log files are valid and ready to run 

```
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

![image](https://github.com/user-attachments/assets/540d31a9-0cf9-47f1-a611-24168fb20d57)

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
```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/suricata/fast.log</location>
</localfile>
```

![image](https://github.com/user-attachments/assets/01d3c705-a8ac-40d3-96c9-8b8eee3221d7)


💡 Make sure this block is placed inside the <ossec_config> root element and not within any comments.

🔁 Step 4: Restart the Wazuh Agent

Apply the changes by restarting the Wazuh agent:

```bash
sudo systemctl restart wazuh-agent
```

✅ Result
After restarting, Wazuh will begin ingesting Suricata alerts from the eve.json log file. You should start seeing Suricata detections in the Wazuh Dashboard under Security Events.


🛡️ Congratulations! Suricata is now successfully integrated with Wazuh for enhanced real-time intrusion detection.




