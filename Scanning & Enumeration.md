# Scanning & Enumeration

Identify every running service, version, and misconfiguration. This is where most OSCP boxes are won or lost. **enum is the single most-practiced skill in your tracker.**

> General enumeration practice: [Heist](https://app.notion.com/p/Heist-316c3dbf5cad8036adfcf9238dee2c45?pvs=21) (HTB) · [Love](https://app.notion.com/p/Love-316c3dbf5cad8046ad37dfd47f2e844b?pvs=21) (HTB) · [Remote](https://app.notion.com/p/Remote-316c3dbf5cad80fa8dd1f68ff568364d?pvs=21) (HTB) · [Servmon](https://app.notion.com/p/Servmon-316c3dbf5cad8014af0deed832cde2c6?pvs=21) (HTB) · [Netmon](https://app.notion.com/p/Netmon-316c3dbf5cad80c89e6ece01df28aed5?pvs=21) (HTB) · [Blackfield](https://app.notion.com/p/Blackfield-316c3dbf5cad80169dbeeb5533bdf461?pvs=21) (HTB) · [Cicada](https://app.notion.com/p/Cicada-316c3dbf5cad807fb642fe2a65e5fccf?pvs=21) (HTB) · [Sauna](https://app.notion.com/p/Sauna-316c3dbf5cad8059a7b8e1574eb92d5e?pvs=21) (HTB) · [Active](https://app.notion.com/p/Active-316c3dbf5cad80cfa9abdafa8999bbec?pvs=21) (HTB) · [Busqueda](https://app.notion.com/p/Busqueda-316c3dbf5cad8074935bdcab3b666121?pvs=21) (HTB) · [Precious](https://app.notion.com/p/Precious-316c3dbf5cad80f3b4d5f758a57f1268?pvs=21) (HTB) · [Mentor](https://app.notion.com/p/Mentor-316c3dbf5cad80839dd6ee7ae0d117e8?pvs=21) (HTB) · [Poison](https://app.notion.com/p/Poison-316c3dbf5cad80a1a3c0f5e4cc69094e?pvs=21) (HTB) · [Solidstate](https://app.notion.com/p/Solidstate-316c3dbf5cad80f0b710fcec47dae480?pvs=21) (HTB) · [SkillForge](https://app.notion.com/p/SkillForge-2aec3dbf5cad80bbb4e6e56d5109da5e?pvs=21) (PG) · [Fired](https://app.notion.com/p/Fired-2aec3dbf5cad8013a92def91659e587a?pvs=21) (PG) · [Mzeeav](https://app.notion.com/p/Mzeeav-2aec3dbf5cad808c80f6c532313173a9?pvs=21) (PG)
> 

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
