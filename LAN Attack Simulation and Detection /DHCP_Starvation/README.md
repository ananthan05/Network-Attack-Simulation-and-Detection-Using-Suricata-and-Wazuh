For DHCP starvation attack we need to install a tool called yersinia
```
sudo apt-get install yersinia
```
After install we can start it using

```
sudo yersinia -I
```
![image](https://github.com/user-attachments/assets/6cc170aa-46b9-43cf-ac1c-9f19558abe9e)

Press enter and then press `g` to select the protocol for attack

![image](https://github.com/user-attachments/assets/24754b19-852e-487d-be48-398d32aca33c)

Select the DHCP protocol in it. Now press x to bring up the attack screen and we will select `1` for our attack

![image](https://github.com/user-attachments/assets/4e5e36f2-f9cf-4c2c-8086-d786858b99ac)

Now the attack will start.Sending many DHCP Discover packets rapidly (especially with spoofed MAC addresses) can exhaust the DHCP server's IP pool (leading to DHCP starvation)

![image](https://github.com/user-attachments/assets/08501e54-6920-4de7-9ca9-adcaf84e7d6e)

We can see the attack happening.We can press `q` to stop the attack

Now to detect the attack

In Suricata we need to add a custom script rule for it to detect the attack

```
alert udp any any -> any 67 \
(msg:"DHCP Starvation Attack"; \
content:"|01|"; offset:242; depth:1; \
flow:to_server; \
classtype:attempted-dos; \
sid:1000002; rev:12;)
```
save this under `/etc/suricata/rules` as a .rule file

Now run these commands

```
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
```
sudo systemctl restart suricata
```
We can see the fast.log with this command 

```
sudo tail -f /var/log/suricata/fast.log
```

![Screenshot 2025-07-01 143635](https://github.com/user-attachments/assets/cd19536f-4192-4e33-9760-46aa4a57c366)

We can see that suricata has succesfully detected the dhcp starvation attack
The rule was matched when a UDP packet was observed being sent from IP 0.0.0.0, port 68 (a DHCP client) to broadcast IP 255.255.255.255, port 67 (used by DHCP servers).
Yersinia simulates many fake clients,all pretending to be new devices so they all send from 0.0.0.0.The 255.255.255.255  is the broadcast address for the local network,since the fake clients don't know the DHCP server's IP they send the request to the broadcast address.

In wazuh dashboard also we can see the log

![image](https://github.com/user-attachments/assets/c442c684-1ab5-4384-8400-e51b95d997ba)

