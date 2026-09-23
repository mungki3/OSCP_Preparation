# Scanning & Enumeration

Identify every running service, version, and misconfiguration. This is where most OSCP boxes are won or lost. **enum is the single most-practiced skill in your tracker.**

## Comprehensive port scanning

Never assume only 22/80/443 exist.

```bash
nmap -p- target.com -v            # ALL 65535 TCP ports
nmap -sV -sC -O -A target.com     # versions + default scripts + OS
nmap -sU -p 53,67,123,161,162 target.com   # critical UDP
```

## Service-specific enumeration

### SMB (139/445) - almost always misconfigured

```bash
enum4linux-ng target.com
nmap -p 139,445 --script smb-* target.com
```

### SNMP (161/UDP) - default community strings grant access

```bash
snmpwalk -c public -v1 target.com
snmp-check target.com -c public
```

### LDAP (389/636) - unauth queries dump the directory

```bash
ldapsearch -h target.com -x -b "dc=target,dc=com"
```

### PRTG / monitoring web apps

> Practice: [Netmon](https://app.notion.com/p/Netmon-316c3dbf5cad80c89e6ece01df28aed5?pvs=21) (HTB)
> 

> Path traversal practice: [Servmon](https://app.notion.com/p/Servmon-316c3dbf5cad8014af0deed832cde2c6?pvs=21) (HTB)
> 

## Directory & file discovery

Do not stop at root. Check /admin, /api, /backup, /upload, /.git, /.env.

```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/common.txt
ffuf -u http://target.com/FUZZ -w wordlist.txt -fs 1024 -fw 100
```

> API enumeration practice: [Mentor](https://app.notion.com/p/Mentor-316c3dbf5cad80839dd6ee7ae0d117e8?pvs=21) (HTB) · [Xposedapi](https://app.notion.com/p/Xposedapi-2aec3dbf5cad80a8a59fd02631469d4c?pvs=21) (PG)
> 

## Gap fixes

- **Gap 4 - only common ports:** always `-p-`.
- **Gap 5 - UDP skipped:** SNMP/DNS/NTP live on UDP.
- **Gap 7 - shallow web enum:** enumerate /admin, /backup, /.git, /web.config.
- **Gap 8 - default creds never tried:** admin/admin on Tomcat, Jenkins, MySQL, MongoDB, Redis.
- **Gap 9 - creds from configs not reused:** test found passwords against SSH/RDP/DB/domain.

# --- SUPPLEMENT: real tools from your write-ups ---

## Web content discovery (feroxbuster is your actual go-to, 26 boxes)

```bash
feroxbuster -u http://target -x php,txt,html -s 200,301,302,403 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
ffuf -u http://target/FUZZ -w wordlist -mc 200,301,302,403
ffuf -u http://target -H "Host: FUZZ.target" -w subdomains.txt -fs <baseline>   # vhost
whatweb -a3 http://target && nikto -h http://target
```

## SMB / AD recon with netexec (nxc) - used on 15 boxes

```bash
nxc smb <ip>                         # host + domain + signing
nxc smb <ip> -u '' -p '' --shares    # null session
nxc smb <ip> -u user -p 'Pass' --rid-brute
nxc smb <subnet> -u user -p 'Pass' --continue-on-success   # spray
```

## SNMP / RPC deep

```bash
onesixtyone -c community.txt <ip>
snmpwalk -c public -v2c <ip> .1        # full walk
rpcclient -U '' -N <ip> -c 'enumdomusers;querydispinfo'
```
