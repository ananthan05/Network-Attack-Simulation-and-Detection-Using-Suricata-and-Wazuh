## Simulate DOS from Attacker VM

###  Step 1: Identify Target (Victim) IP Address

From the **Kali Attacker**, scan the local network:

```bash
nmap -sn 10.0.2.0/24
```

![image](https://github.com/user-attachments/assets/2c227bcb-c4d2-42d7-b654-8785532b0bca)

### Step 2: Run TCP SYN Flood Attack (Kali Attacker)

Send a SYN flood to the victim:

```bash
sudo hping3 -S --flood -V -p ++10 10.0.2.15
```
-S: SYN flag
--flood: send packets as fast as possible
 -p ++10 : Increment the destination port number starting from port 10 (i.e., send to 10, 11, 12, ..., etc.)
10.0.2.15: victim IP

![image](https://github.com/user-attachments/assets/fcebde83-e103-4422-ab63-0d30c80a273f)

Now go and check the victim PC by accessing any webpage — it will load slowly due to the DoS attack, indicating that the attack was successful.

![image](https://github.com/user-attachments/assets/d5e13fb4-e6f8-471d-a293-b81e9cd29acd)

Attack was successful.

## Detect DoS attacks using Suricata and Wazuh.

### Step 1 : Write Custom Rule to Detect SYN Flood on Victim.

Create a file or add to

```bash
sudo gedit /etc/suricata/rules/custom.rules
```
rule-

```rule
alert tcp any any -> any any (msg:"[DoS] TCP SYN Flood Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 3; sid:1000010; rev:3;)
```

![image](https://github.com/user-attachments/assets/dd045882-4cff-42d6-9a5e-ca5cca0a0664)

We have already updated the suricata.yaml file to include custom.rules in the initial setup, so we don't need to add it manually again. Any rules we write will be automatically included.

Test and restart Suricata:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

![image](https://github.com/user-attachments/assets/8d697932-397c-4645-ae91-a1ffd2f7b480)

```bash
sudo systemctl restart suricata
```
![image](https://github.com/user-attachments/assets/f107c4ea-4caa-4b7e-a693-41128c3cfacf)

### Step 2: Run TCP SYN Flood Attack (Kali Attacker)

```bash
sudo hping3 -S --flood -V -p ++10 10.0.2.15
```

### Step 3: Monitor Alerts via Suricata (Victim)

On the victim, monitor alert logs:

```bash
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```

![image](https://github.com/user-attachments/assets/62685b3f-cba9-47a3-91b2-c9b1c9cb8259)

Attack detected successfully.


## View Alerts in Wazuh Dashboard

In Wazuh :

Go to Discover or wazuh alerts

Search for:`

```
data.alert.signature: [DoS] TCP SYN Flood Detected
```

![image](https://github.com/user-attachments/assets/4b7c2c2e-d077-4258-934f-6c88efe7bc25)


### Screenshot Summary

- **Signature ID**: 1000010
- **Time Range**: Jul 1, 2025 — 11:30:00 to 12:00:00 UTC
- **Alerts Count**: 13
- **Attacker IP**: 10.0.2.4
- **Victim IP**: 10.0.2.15

### 📈 Graph Analysis

- The histogram shows spikes of alert activity, aligned with TCP SYN flood bursts.
- Helps identify attack timeframes and correlate with potential system impact.

### Detailed Alert Fields

| Field | Value |
|-------|-------|
| Signature | [DoS] TCP SYN Flood Detected |
| Severity | 3 |
| Source IP | 10.0.2.4 |
| Destination IP | 10.0.2.15 |
| Packet Source | wire/pcap |
| Status | Allowed |
| Timestamps | Precise attack times logged |
