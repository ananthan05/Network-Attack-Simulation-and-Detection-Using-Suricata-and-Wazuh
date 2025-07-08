##  SSH Brute Force Remote Attack Detection (Suricata + Wazuh)

Simulate a **remote SSH brute-force attack** using Hydra and detect the malicious behavior using **Suricata** and **Wazuh** (deployed on the victim).

---

## 🧪 Lab Setup

| Role     | Machine | IP Address         | Network Mode     |
|----------|---------|--------------------|------------------|
| Attacker | Kali    | `172.17.128.157`   | Bridged Adapter  |
| Victim   | Ubuntu  | `10.0.2.15`        | NAT Network      |

> 💡 The victim runs an SSH server on port `22`, which is forwarded to host port `2222` so the attacker (on a different subnet) can access it using the host IP.

---

## ⚙️ Step 1: Set Up SSH Server on Victim (Ubuntu)

### 1.1 Install SSH Server

```bash
sudo apt update
sudo apt install openssh-server -y
```
### 1.2 Enable and Start SSH

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```
### 1.3 Check SSH Status

```bash
sudo systemctl status ssh
```

![image](https://github.com/user-attachments/assets/b5a418c1-3780-4fd0-a93c-a778b7b9dd57)

 Output should show "active (running)".

##  Step 2: Port Forwarding for Victim SSH

On VirtualBox (Victim):

- Go to **Tools > Network >  NAT Network > Port Forwarding**
- Add a rule:
  - **Protocol**: TCP
  - **Host IP**: blank
  - **Host Port**: 2222
  - **Guest IP**: 10.0.2.15
  - **Guest Port**: 22

---

![image](https://github.com/user-attachments/assets/4f202f67-377b-482f-8f3d-be6d6f2fbe9a)


##  Step 3.1 : Hydra Brute Force from Attacker (Kali)

We will create a small custom password list that includes the victim's actual password to speed up the brute-force simulation.
This is because the default rockyou.txt wordlist, which comes preinstalled on Kali Linux, contains over 14 million passwords, making the attack slow and inefficient for testing.
By using a short list, we can simulate the attack faster and still achieve successful detection.:

```bash
echo -e "123\n1234\nadmin\npassword" > ~/fast-pass.txt
```

![image](https://github.com/user-attachments/assets/f3f1a556-0819-478e-a12e-48114e11ea93)

### 3.2 Find Host IP (Used by Kali to Reach Victim via Port Forward)

Run on your host OS:

```bash
ipconfig      # on Windows
```

![image](https://github.com/user-attachments/assets/4d305c8b-d950-4506-ade1-d04bd67b47f5)


## Step 4: Launch Brute Force Attack (Hydra)

Run from Kali:

```bash
hydra -l victim -P ~/fast-pass.txt ssh://172.17.128.27:2222
```

![image](https://github.com/user-attachments/assets/a63a70ba-4b19-41e4-9cfc-1561efbb22ca)

The attack was successful — we obtained the credentials.

## Step 5: Monitor Suricata Alerts on Victim

```
sudo tail -f /var/log/suricata/fast.log
```
![image](https://github.com/user-attachments/assets/37d5f35a-aad7-43d8-93c4-ed6195b0a981)

The attack was successfully detected.

## Step 6: View Detection in Wazuh Dashboard
In Wazuh :

Go to Discover or wazuh alerts

filter for for: fastlog path.

![image](https://github.com/user-attachments/assets/40a6d503-dc76-4ec0-ac36-78c95fe474f3)
