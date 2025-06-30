
## 📥 Installing Zeek 7.0 LTS (Ubuntu 22.04)

To install the latest **Zeek 7.0 LTS** on Ubuntu 22.04, follow these steps:

### 🧱 Step-by-Step Installation

```bash
# Add the official Zeek repository
echo 'deb https://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/ /' | sudo tee /etc/apt/sources.list.d/security:zeek.list

# Import the repository GPG key
curl -fsSL https://download.opensuse.org/repositories/security:zeek/xUbuntu_22.04/Release.key | \
  gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/security_zeek.gpg > /dev/null

# Update the package list
sudo apt update

# Install Zeek 7.0
sudo apt install zeek-7.0
```

✅ Post-installation
Add Zeek to your system PATH:

```bash
echo 'export PATH=/usr/local/zeek/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

✅ Verify the Installation

```bash
zeek --version
```
![image](https://github.com/user-attachments/assets/8734ccff-eb1b-44f2-97df-d3967ec20d69)


## ⚙️ Post-Installation Configuration for Zeek

Once Zeek 7.0 is installed, follow these steps to configure it for monitoring:

Edit node.cfg to define which interface Zeek should monitor:

```
gedit /opt/zeek/etc/node.cfg
```

change the inferface of your sysytem by checking `ifconfig`

![image](https://github.com/user-attachments/assets/b6bc2c3c-66c3-4983-81e6-4cef23826049)

![image](https://github.com/user-attachments/assets/29f1054d-80df-42fa-a6f4-e3c1b18e047d)

📊 View Logs:

Zeek logs are usually stored in:
`
```
/opt/zeek/logs/current
```

![image](https://github.com/user-attachments/assets/9410fedb-1175-4040-87a9-0cd347c499bd)

conn.log – Connection logs

dns.log – DNS queries

http.log – HTTP traffic

notice.log – Security alerts


### Initialize the Zeek control framework:

```bash
sudo ./zeekctl check         # Check config validity
sudo ./zeekctl install       # Install runtime environment
sudo ./zeekctl start         # Start Zeek monitoring
sudo ./zeekctl deploy
```
go to `
```
cd /opt/zeek/bin
```
![image](https://github.com/user-attachments/assets/055bfa37-95c6-4851-9c70-1e36d4461cee)

![image](https://github.com/user-attachments/assets/6f6bb6bc-7ec5-469f-be4f-9d96e22750c1)
