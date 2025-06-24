## 🛡️ Wazuh Server VM Installation

Download the official Wazuh Server Virtual Machine from the Wazuh documentation site:  
🔗 [https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html)

![Wazuh VM Import](https://github.com/user-attachments/assets/6df40ea3-2cb7-4987-965f-37e624ee4101)

### 📥 Import VM into VirtualBox

1. Open **VirtualBox**.
2. Import the downloaded `.ova` file using `File → Import Appliance`.
3. Follow the prompts to finish the import process.

---

### 🔐 Default Login Credentials

- **Username:** `wazuh-user`  
- **Password:** `wazuh`

---

### 🚀 Start Wazuh Services

After logging into the Wazuh VM, run the following commands to start the required services:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-dashboard
sudo systemctl daemon-reexec
sudo systemctl start wazuh-indexer
```

📝 You can check the status of each service by replacing start with status in the above commands.

Example:
```
sudo systemctl status wazuh-manager
```


### Access the Wazuh Web Interface
  Find the VM’s IP address:

```
ip a
```
On your attacker VM (or any other VM in the same network), open Firefox and enter the Wazuh server IP in the browser:

```cpp
http://<wazuh-vm-ip>
```


✅  Now you can log into the Wazuh Dashboard using the credentials:

- **Username:** `admin`  
- **Password:** `admin`

and start monitoring!





