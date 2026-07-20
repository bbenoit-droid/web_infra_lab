# Report: UFW HTTP Rule for web-01 and web-02 from lb-01

# Objective
I attempted to configure a UFW rule on the backend web servers so that they would accept plain-text HTTP traffic only from the load balancer, while keeping the rest of inbound traffic restricted.

## Environment
The lab consists of three containers:
- web-01: 172.25.0.11
- web-02: 172.25.0.12
- lb-01: 172.25.0.10

The containers were accessed through the lab SSH ports:
- `ssh ubuntu@localhost -p 2211` → web-01
- `ssh ubuntu@localhost -p 2212` → web-02
- `ssh ubuntu@localhost -p 2210` → lb-01

## Steps I performed

### 1. Checked the firewall state on each host
I tried to inspect UFW status inside each container.

#### On web-01
Command:
```bash
sudo ufw status numbered
```
Response observed:
```text
ERROR: problem running iptables: iptables v1.8.10 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)
```

#### On web-02
Command:
```bash
sudo ufw status numbered
```
Response observed:
```text
ERROR: problem running iptables: iptables v1.8.10 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)
```

#### On lb-01
Command:
```bash
sudo ufw status numbered
```
Response observed:
```text
sudo: ufw: command not found
```

### 2. Intended UFW rule for the requirement
The rule needed to satisfy the task was the following on web-01 and web-02:
```bash
sudo ufw allow from 172.25.0.10 to any port 80 proto tcp
```
This would allow only the load balancer at `172.25.0.10` to reach TCP port 80 over plain HTTP.

In a normal Ubuntu VM, the full sequence would be:
```bash
sudo apt update
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 172.25.0.10 to any port 80 proto tcp
sudo ufw enable
sudo ufw status numbered
```

## Result
The requested UFW rule could not be successfully applied in this lab setup because the containers do not have the kernel-level firewall permissions required by UFW/iptables. The responses showed that:
- web-01 and web-02 could not run `ufw` properly because `iptables` reported a permission error.
- lb-01 did not even have the `ufw` package available (`command not found`).

## Conclusion
The task was understood and the required rule was identified, but the environment prevented the rule from being installed and enabled inside the Docker containers. In a real Ubuntu server environment, the above UFW commands would be the correct way to enforce the requirement that only the load balancer can reach plain HTTP on web-01 and web-02.
