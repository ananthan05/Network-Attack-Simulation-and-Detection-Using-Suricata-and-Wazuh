DHCP spoofing is a network attack where a rogue (unauthorized) DHCP server responds to DHCP requests with fake configuration data. This can mislead devices into using a malicious gateway or DNS server, enabling man-in-the-middle attacks or denial of service.

We will be using the tool yersinia for this attack

Before that we need to change the vm configurations, this atack will only work on host-only network on virtualbox. So we need to add a host-only adapter if it is not there

![image](https://github.com/user-attachments/assets/e0034956-d324-4014-9f89-ebb7f3d42d43)

Make sure to turn off the DHCP server option

![image](https://github.com/user-attachments/assets/c49e4181-423c-4896-bc5e-3b5d12ec555d)

Now change the network for both vms to `host-only` network and restart the vms.

Initialy both vm will not have any ips as we disabled dhcp server, so we will set ip for attacker manually.

```bash
sudo ip addr add 10.0.2.5/24 dev eth0
sudo ip link set eth0 up
```
## Attacking
Now we will open the tool

```
sudo yersinia -I
```

Now pressing `g` will bring up protocol list and select DHCP protocol in it

After that pressing `x` button will bring up attack screen on that select number 2 for configuring the dhcp properties

![image](https://github.com/user-attachments/assets/f1ee690c-2be5-43db-add0-3df82de9a982)

we will configure the values in yersinia tool. The router and dns server is given same ip as attacker so any traffic that goes from the victim will go through the attacker.

Let the attack start then initialy if we try to run command `ifconfig` it will show vicitm has no ip address

![Screenshot 2025-07-02 144813](https://github.com/user-attachments/assets/235b28f8-27b1-45d7-9e6e-1cd2e2db0c29)

now run the command

```bash
sudo dhclient -r && sudo dhclient
```
The client will make a request to the attacker for an ip address

![Screenshot 2025-07-02 144746](https://github.com/user-attachments/assets/9e3ce4ce-0904-4c77-8617-918135c8dd4f)

now we can see that the client has a new ip assained from the attacker

![Screenshot 2025-07-02 144802](https://github.com/user-attachments/assets/9620a012-9b07-43d2-988b-eaab89643b1f)

Attack was successfull

## Detecting on Suricata logs

In suricata we need to add a custom .rules file

```
alert udp any 67 -> any 68 (msg:"[ALERT] DHCP Offer Detected"; content:"|35 01 02|"; offset:240; depth:10; sid:1000003; rev:1;)
```

This rule will alert whenever a dhcp request is being made and it will aslo list out the ip of the server

After saving it we need to load it in `/etc/suricata/suricata.yaml` file.Sometimes the custom rules we add will not load automatically in that cases we need to explicitly add it in the file

![image](https://github.com/user-attachments/assets/e73ab470-7ccb-46ea-b221-2106ab959651)

The rest of the rules could be commented out if it is causing errors

now run the codes

```
sudo suricata -T -c /etc/suricata/suricata.yaml -v
sudo systemctl restart suricata
```

now during attack if we tail the /var/log/suricata/fast.log we can see the alert

```
sudo tail -f fast.log
```

![Screenshot 2025-07-02 144929](https://github.com/user-attachments/assets/4c94db32-41e2-47ce-b68d-48dd163d7adc)

