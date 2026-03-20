## How to monitor usage of a node

Disk space usage:

- df -h 

CPU usage (look at %CPU used and load average):

- top 
- htop 
- uptime

Memory (RAM) Usage (if RAM is full, the system slows down or starts swapping):

- free -h 

Disk I/O (even if disk space is fine, disk read/write operations might be overloaded):

- iostat-x 1
- iotop

Network usage:

- iftop
- nload 

Running processes (check if something is consuming resources):

- top 

For monitoring a node's usage, a typical engineer might check something like:

- uptime
- free -h 
- df -h 
- top 


### To run all commands at once as a script

Create a temporary script file:

nano monitor_node72.sh 

```
#!/bin/bash

echo "===== UPTIME / LOAD ====="
uptime

echo -e "\n===== MEMORY USAGE ====="
free -h

echo -e "\n===== TOP PROCESSES (CPU) ====="
top -b -n 1 | head -20

echo -e "\n===== DISK USAGE ====="
df -h

echo -e "\n===== DISK I/O (optional) ====="
iostat -x 1 3

echo -e "\n===== NETWORK USAGE (optional) ====="
ifstat 1 3
```

