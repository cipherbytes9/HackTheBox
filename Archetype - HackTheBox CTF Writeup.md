# Archetype - HackTheBox CTF Writeup

## Target Information
- **IP Address:** 10.129.60.159
- **Operating System:** Windows Server 2019 Standard
- **Hostname:** ARCHETYPE

## Reconnaissance

### Port Scan Results
| Port | Service     | Version                        |
|------|-------------|--------------------------------|
| 135  | msrpc       | Microsoft Windows RPC          |
| 139  | netbios-ssn | Microsoft Windows netbios-ssn  |
| 445  | microsoft-ds| Windows Server 2019           |
| 1433 | ms-sql-s    | Microsoft SQL Server 2017      |
| 5985 | http        | Microsoft HTTPAPI 2.0 (WinRM)  |

## Initial Access

### SMB Enumeration
- Anonymous access to `backups` share was allowed
- Found configuration file: `prod.dtsConfig`

### Credentials Discovered
- **Username:** ARCHETYPE\sql_svc
- **Password:** [REDACTED - from config file]

### MSSQL Exploitation
Used `mssqlclient.py` (Impacket) to connect to MSSQL:
```
mssqlclient.py ARCHETYPE/sql_svc:[REDACTED]@10.129.60.159 -windows-auth
```

Enabled xp_cmdshell for code execution:
```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

## Privilege Escalation

### User Flag
- Accessed: C:\Users\sql_svc\Desktop\user.txt
- Obtained user credentials

### Horizontal Privilege Escalation
- Discovered PowerShell history file containing admin password
- File location: `C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
- **Administrator password:** [REDACTED]

Used Impacket's `psexec.py` and `wmiexec.py` to obtain SYSTEM access:
```
wmiexec.py 'ARCHETYPE/Administrator:[REDACTED]'@10.129.60.159
```

### Root Flag
- Accessed: C:\Users\Administrator\Desktop\root.txt

## Tools Used
- nmap
- smbclient
- mssqlclient.py (Impacket)
- psexec.py (Impacket)
- wmiexec.py (Impacket)

## Vulnerabilities Exploited
1. Anonymous SMB access to backups share
2. Cleartext credentials in configuration file
3. MSSQL xp_cmdshell enabled
4. PowerShell history file containing credentials
5. Weak administrative password

## Remediation
- Disable anonymous SMB access
- Use strong, unique passwords for service accounts
- Restrict MSSQL permissions - avoid running with sysadmin privileges
- Implement least privilege principle
- Exclude sensitive paths from PowerShell history logging
- Use credential managers instead of plaintext passwords in configs
