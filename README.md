# Monitoring SNMPv3 with Zabbix

This guide walks through configuring SNMPv3 on a Linux server for integration with Zabbix monitoring.

---

## 📆 Install SNMP and SNMPD

```bash
sudo apt update
sudo apt install snmp snmpd
```

Start and enable the SNMP daemon:

```bash
sudo systemctl start snmpd
sudo systemctl enable snmpd
```

---

## ⚙️ Configure SNMP Daemon

Edit the SNMP configuration file:

```bash
sudo vim /etc/snmp/snmpd.conf
```

Make the following changes:
- Set the `agentAddress` to listen on UDP port 161:
  ```conf
  agentAddress udp:161
  ```
- Remove or comment default community settings.
- Add user-based security by creating an SNMPv3 user (see below).

Restart the SNMP daemon:

```bash
sudo systemctl restart snmpd
```

---

## 👤 Create SNMPv3 User

Install required development library:

```bash
sudo apt install libsnmp-dev
```

Stop the SNMP service temporarily:

```bash
sudo systemctl stop snmpd
```

Create a new SNMPv3 user with authentication and privacy (encryption):

```bash
sudo net-snmp-config --create-snmpv3-user -ro -a SHA-512 -A "myauthpass" -x AES -X "myprivpass" AuthPrivUser
```

> You can verify the user entry in:
```bash
sudo vim /var/lib/snmp/snmpd.conf
```

Start SNMPD again:

```bash
sudo systemctl start snmpd
sudo systemctl status snmpd
```

---

## 🔐 Configure IPTables (Optional but Recommended)

Install `iptables` if not already installed:

```bash
sudo apt install iptables
```

Allow Zabbix server to query SNMP (replace `<Zabbix-IP>`):

```bash
sudo iptables -A INPUT -p udp --dport 161 -s <Zabbix-IP> -j ACCEPT
sudo iptables -A INPUT -p udp --dport 161 -j DROP
```

Check active rules:

```bash
sudo iptables -L -v
```

---

## 📂 Save IPTables Rules

Install iptables-persistent:

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

Or manually save the rules:

```bash
# IPv4
sudo sh -c '/sbin/iptables-save > /etc/iptables/rules.v4'

# IPv6
sudo sh -c '/sbin/ip6tables-save > /etc/iptables/rules.v6'
```

---

## 🧰 Test SNMPv3 Access

Use `snmpget` to test:

```bash
sudo snmpget -v3 -u AuthPrivUser -l authpriv -a SHA-512 -A myauthpass -x AES -X myprivpass <Target-IP> 1.3.6.1.2.1.1.1.0
```

Or use `snmpwalk`:

```bash
sudo snmpwalk -v3 -u AuthPrivUser -l authpriv -a SHA-512 -A myauthpass -x AES -X myprivpass <Target-IP> | head -10
```

---

## ✅ Done!

You can now add this host to your Zabbix frontend using SNMPv3 and start monitoring!

---

> Created by [Mohammad Talebi](https://linkedin.com/in/mtlbd) – DevOps Engineer 👨‍💻
