`docker run -d --name eg_plc --net plc-macvlan --ip 192.168.40.202 --privileged openplc:v3`
 
ubuntu@pgt:~$ ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:62:7e:2a brd ff:ff:ff:ff:ff:ff
    inet 192.168.30.60/24 brd 192.168.30.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe62:7e2a/64 scope link
       valid_lft forever preferred_lft forever
 
Perfect 👍 — your VM’s bridged IP is 192.168.30.60, so that’s exactly the address to use in your forwarding rules.
Here’s the exact set of commands you can run inside your Ubuntu VM to make your 4 PLC Docker containers accessible from:

- the Remote Desktop host where the VM runs, and
- your own machine (the one you use to SSH into the VM).

🧩 Step 1 — Run these commands inside the Ubuntu VM
```
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.30.60 --dport 8081 -j DNAT --to-destination 192.168.40.201:8080
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.30.60 --dport 8082 -j DNAT --to-destination 192.168.40.202:8080
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.30.60 --dport 8083 -j DNAT --to-destination 192.168.40.203:8080
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.30.60 --dport 8084 -j DNAT --to-destination 192.168.40.204:8080
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```
These rules tell Ubuntu:
"When traffic comes to 192.168.30.60:8081, forward it internally to 192.168.40.201:8080 (and so on for the others)."
🧩 Step 2 — Verify it’s working
From your Remote Desktop host, open:
http://192.168.30.60:8081/login → PLC1
http://192.168.30.60:8082/login → PLC2
http://192.168.30.60:8083/login → PLC3
http://192.168.30.60:8084/login → PLC4
From your own computer (the one you use to SSH in), you can use the same URLs — as long as you can ping 192.168.30.60.
You do not need to use localhost or SSH port forwarding anymore.
🧩 Step 3 — (Optional but recommended) Make the rules persistent
To make sure the forwarding stays active after reboot:
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
sudo netfilter-persistent enable 
Now the rules you just added will automatically reapply every time the VM restarts.
✅ Quick access summary
PLC	Container IP	Browser Access URL	Target (inside VM)
PLC1	192.168.40.201	http://192.168.30.60:8081/login	192.168.40.201:8080
PLC2	192.168.40.202	http://192.168.30.60:8082/login	192.168.40.202:8080
PLC3	192.168.40.203	http://192.168.30.60:8083/login	192.168.40.203:8080
PLC4	192.168.40.204	http://192.168.30.60:8084/login	192.168.40.204:8080
Would you like me to modify your existing /usr/local/sbin/setup-macvlan.sh so it automatically sets up these forwarding rules too, so you only manage one script?
 
 
TO DO: need to continue for other 3 plc containers, persist the rules
 
make sure to docker start [plc_name]
 
st_plc -> http://192.168.30.60:8081/login 
 
Can be accessed on remote desktop (base machine of cyberX) or on your own host machine where you login to the VM
 
After setting the DNAT rule:

```
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.30.60 --dport 8081 -j DNAT --to-destination 192.168.40.201:8080
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```


Able to access http://192.168.30.60:8081/dashboard from the host machine (host machine that has ssh connection to ubuntu VM on remote host & docker container is on this ubuntu vm) but NOT able to ping 192.168.30.60 
 
⚙️ Why http works but ping doesn’t
 
`-p tcp --dport 8081`

So it matches only TCP packets going to port 8081.
**ping** uses **ICMP**, not TCP.
Therefore, your **DNAT rule** does not apply to ICMP at all.
The host (192.168.30.60) has no direct route to 192.168.40.0/24
The internal Docker network (192.168.40.0/24) is isolated by default.
Containers can reach out to the internet via NAT, but the host or external machines cannot directly reach them unless:
You create a bridge network and expose it,
Or add specific iptables / routing rules to forward ICMP (rarely done).
HTTP works because of port forwarding (DNAT)
When your browser connects to port 8081 on 192.168.30.60, iptables rewrites the packet’s destination to 192.168.40.201:8080.
Docker or the VM’s networking stack then handles the return packets (via MASQUERADE).
So HTTP works perfectly through this NAT.
