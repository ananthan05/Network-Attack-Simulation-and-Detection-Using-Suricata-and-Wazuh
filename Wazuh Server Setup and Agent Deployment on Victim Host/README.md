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

## 🖥️ Installing Wazuh Agent on Victim VM

In the Wazuh Dashboard, go to the **"Agents"** tab and click on **"Add agent"**.

![Add Agent](https://github.com/user-attachments/assets/f2a762bc-6656-4002-b82b-834dadd8b7ce)

### 1️⃣ Configure the Agent

Fill in the required details as shown below:

- **Operating system:** Linux DEB (amd64)
- **Wazuh Manager IP:** *IP address of your Wazuh Server VM*
- **Agent Name:** e.g., `victim` or any name you prefer

Leave the other settings as default, then click **Next**.

![Agent Config](https://github.com/user-attachments/assets/e9d8c9bb-99b6-4ee8-866d-fa80fd2036c4)

Wazuh will provide you with a command to run on the victim machine. Copy that command.

![Generated Command](https://github.com/user-attachments/assets/5d2333b8-953c-402c-8d90-16d0c733144a)

### 2️⃣ Run the Command on the Victim VM

Paste and run the command in your **victim VM** terminal to install and configure the Wazuh agent.

![Running on Victim](https://github.com/user-attachments/assets/1b7d0f0f-1db9-4ed8-88a4-6f5b39583dc4)

### 3️⃣ Start the Wazuh Agent

Now enable and start the agent service with the following commands:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

```
Once the agent starts, you will see it successfully registered and connected in the Wazuh Dashboard. ✅

