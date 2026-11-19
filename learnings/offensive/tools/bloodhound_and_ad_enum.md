# BloodHound & AD Enumeration (`bloodhound-python`, `kerbrute`, `ldapsearch`)

> **TL;DR:** **BloodHound** maps Active Directory relationships (users, groups, sessions, ACLs, GPOs) into a visual graph to reveal hidden attack paths to **Domain Admin**. Pair it with **`bloodhound-python`** (runs directly from Kali) and **`kerbrute`** for pre-auth AD user enumeration.

---

## Installation & Setup in Kali Linux
```bash
sudo apt install -y bloodhound neo4j bloodhound.py
# Start Neo4j graph database (default creds: neo4j:neo4j -> change on first login)
sudo neo4j start
# Launch BloodHound GUI
bloodhound
```

---

## 1. Collecting AD Data from Kali (`bloodhound-python`)

When you have valid domain credentials (`alice:Pass123!`) and network access to the Domain Controller (`10.10.10.10`):
```bash
mkdir ~/bh_data && cd ~/bh_data
bloodhound-python -u alice -p 'Pass123!' -d corp.local -ns 10.10.10.10 -c All
```
| Flag | Purpose |
|------|---------|
| `-u` / `-p` | Domain username & password (or `--hashes :<NTLM>` for Pass-the-Hash) |
| `-d` | Target AD domain (`corp.local`) |
| `-ns` | Domain Controller IP (DNS server) |
| `-c All` | Collection method (`All`, `DCOnly`, `Session`, `ACL`, `Group`) |

*(If running on a compromised Windows host, use `SharpHound.exe -c All` instead, then transfer the `.zip` back to Kali.)*

### Analyzing in BloodHound GUI
1. Drag & drop the generated `.json` or `.zip` files into the BloodHound window.
2. Open the **Analysis** tab -> run built-in queries:
   - **Find Shortest Paths to Domain Admins**
   - **Find Principals with DCSync Rights**
   - **Find AS-REP Roastable Users**
   - **Find Kerberoastable Users**

---

## 2. `kerbrute` — Fast AD Username Enumeration & Spraying

`kerbrute` abuses Kerberos pre-authentication (`AS-REQ`) to validate usernames **without generating Windows `4625` failed-logon events**:
```bash
# Enumerate valid AD usernames
kerbrute userenum --dc 10.10.10.10 -d corp.local /usr/share/seclists/Usernames/Names/names.txt

# Safe password spraying (1 password across valid users)
kerbrute passwordspray --dc 10.10.10.10 -d corp.local valid_users.txt 'Winter2026!'
```

---

## 3. `ldapsearch` — Direct LDAP Querying from Kali
```bash
# Query AD users & descriptions (admins sometimes leave passwords in description fields!)
ldapsearch -x -H ldap://10.10.10.10 -D "alice@corp.local" -w 'Pass123!' \
  -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName description
```

---

## Common Pitfalls & Defensive Notes
- **DNS Resolution Errors in `bloodhound-python`:** Add the DC and domain (`10.10.10.10 dc01.corp.local corp.local`) to `/etc/hosts` and always pass `-ns <DC_IP>`.
- **Defensive Tip:** Run **BloodHound** defensively on your own AD to find and break unintended ACL/AdminSDHolder/GPO attack paths before attackers do. Monitor for `Event ID 4662` / high-volume LDAP queries (`T1087.002`).
