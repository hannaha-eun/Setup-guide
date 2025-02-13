# OpenVPN3 Setup and Usage Guide

## 1. Installation

Switch to the root user:
```bash
sudo -i
```

For Ubuntu, follow these steps:   
```bash
apt install apt-transport-https curl
```
Retrieve the OpenVPN Inc package signing key:   

```bash
mkdir -p /etc/apt/keyrings    ### This might not exist in all distributions
curl -sSfL https://packages.openvpn.net/packages-repo.gpg >/etc/apt/keyrings/openvpn.asc
```
   
   Set up the apt source listing:

   -  replace distribution with your Ubuntu release name (e.g., focal for Ubuntu 20.04):
```bash
echo "deb [signed-by=/etc/apt/keyrings/openvpn.asc] https://packages.openvpn.net/openvpn3/debian DISTRIBUTION main" >>/etc/apt/sources.list.d/openvpn3.list
```
   
   Install OpenVPN3:
   ```bash
   apt update
apt install openvpn3
```
   

---
## 2. Update IP Forwarding Settings

Check the current IP forwarding setting:
```bash
cat /proc/sys/net/ipv4/ip_forward
```
- If the output is `1`, IP forwarding is enabled.
- If the output is `0`, enable IP forwarding:
  1. Edit `/etc/sysctl.conf` and uncomment or add the following line:
     ```
     net.ipv4.ip_forward=1
     ```
  2. Apply the changes:
     ```bash
     sudo sysctl -p
     ```
  3. Verify the change:
     ```bash
     cat /proc/sys/net/ipv4/ip_forward
     ```

---
## 3. Check Firewall NAT Rules

Before connecting to VPN:
```bash
ifconfig
```
Identify:
- Outgoing network (e.g., `eth0`)
- Assigned VPN IP (e.g., `9.9.9.9` )

Update `iptables` rules:
```bash
sudo iptables -t nat -A POSTROUTING -s 9.9.9.9/24 -o eth0 -j MASQUERADE
```
Enable IP forwarding:
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```
Apply changes:
```bash
sudo sysctl -p
```
Restart OpenVPN3:
```bash
sudo systemctl restart openvpn3
```
If the restart fails with `Unit openvpn3.service not found`, proceed to the next steps.

---
## 4. Check Active OpenVPN3 Sessions

List active sessions:
```bash
openvpn3 sessions-list
```
If no sessions are available, disconnect and reconnect:
```bash
openvpn3 session-manage --disconnect --path /net/openvpn/v3/sessions/<SESSION_ID>
openvpn3 config-import --config /path/to/your/config.ovpn
```
Verify OpenVPN3 service is running:
```bash
ps aux | grep openvpn3
```
Check available configurations:
```bash
openvpn3 configs-list
```

---
## 5. Import and Start VPN Session

### Import Configuration
```bash
openvpn3 config-import --config /path/to/your/config.ovpn--persistent
```

### Start VPN Session
```bash
openvpn3 session-start --config /path/to/your/config.ovpn
```
If prompted, enter authentication details.

### Verify Connection
```bash
openvpn3 sessions-list
```
Example Output:
```
Path: /net/openvpn/v3/sessions/48aaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Created: 2025-02-13 10:41:37   PID: 20158
Owner: hanna   Device: tun0
Config name: /path/to/your/config.ovpn
Connected to: tcp:12.12.12.12:8443
Status: Connection, Client connected
```

---
## 6. Terminate the VPN Connection

Stop the session:
```bash
openvpn3 session-stop --config /path/to/your/config.ovpn
```
If this fails, manually get the session path:
```bash
openvpn3 sessions-list
```
Then disconnect:
```bash
openvpn3 session-manage --session-path /net/openvpn/v3/sessions/48aaaaaaaaaaaaaaaaaaaaaaaaaaaa--disconnect
```
Example output after termination:
```
Initiated session shutdown.
Connection statistics:
BYTES_IN.................1000000
BYTES_OUT.................100000
PACKETS_IN...................100
PACKETS_OUT.................1000
TUN_BYTES_IN..............100000
TUN_BYTES_OUT............1000000
TUN_PACKETS_IN..............1000
TUN_PACKETS_OUT.............1000
```

---
### Notes:
- Always check network interfaces (`ifconfig`) before and after VPN connection.
- If experiencing connectivity issues, recheck firewall rules and IP forwarding settings.

This guide ensures proper setup, management, and troubleshooting of OpenVPN3 on Linux.


---
### Helpful links:
- https://www.reddit.com/r/OpenVPN/comments/uodv0n/help_openvpn_kills_my_internet_connection/?rdt=54025



