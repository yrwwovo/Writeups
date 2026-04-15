# Silver Platter Writeup
**平台**：TryHackMe  
**难度**：Easy  
**操作系统**：Linux  
**IP**：10.146.143.209  
**完成日期**：2026/4/15  

**Tasks**：Gain a shell and escalate privileges to root.

---

## 1. 信息收集 / 枚举 (Enumeration)

### Nmap 扫描
```bash
nmap -sV -Pn -T4 10.146.143.209
```
结果：

22/tcp   open  ssh
80/tcp   open  http (nginx)
8080/tcp open http-proxy

### Web 目录枚举
正常访问域名80端口，提取敏感信息
/silverpeas（Silverpeas 内部协作系统）

关键发现
可进入http:10.146.143.209:8080/silverpeas
项目经理用户名提示：scr1ptkiddy
系统存在认证绕过漏洞
使用 Burp Suite 拦截登录请求，删除 Password 参数后成功登录


## 2. 初始访问 (Initial Foothold)
漏洞类型：认证绕过 + IDOR（不安全的直接对象引用）
利用过程：

Burp Suite 拦截登录请求，删除 password 参数，实现认证绕过。
登录后台后发现消息阅读功能 ReadMessage.jsp。
通过修改 URL 参数 ID=5 → ID=6（IDOR），读取到其他用户的私信。

获得凭证：

Username: tim
Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol

权限：普通用户 tim

## 3. 权限提升 (Privilege Escalation)
枚举信息：
```Bash
id
```
发现 tim 用户属于 adm 组（groups=...,4(adm)）。
提权路径：

adm 组默认拥有读取 /var/log/ 下日志文件的权限。
在日志中搜索用户 tyler：
```bash
grep -r "tyler" /var/log/ 2>/dev/null
```
发现 tyler 的 SSH 登录记录，提取密码。
使用 su tyler 切换到 tyler 用户，最终获得 root 权限。

最终权限：root

## 4. Flag

User Flag：THM{c4ca4238a0b923820dcc509a6f75849b}
Root Flag：THM{098f6bcd4621d373cade4e832627b4f6}


## 5. 关键命令 & 技巧记录
### 认证绕过
Burp Suite 删除 password 参数

### IDOR 利用
http://IP:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=6

### 查看日志（adm 组权限）
grep -r "tyler" /var/log/ 2>/dev/null

### 切换用户
su tyler

## 6. 学到的知识 / 新技巧

认证绕过有时只需要删除登录请求中的 password 参数即可。
IDOR 漏洞常出现在 URL 参数（如 ID=）中，修改数字可能访问其他用户数据。
adm 组权限非常重要，默认可读取系统日志，这是常见的隐蔽提权点。
日志信息泄露（尤其是 SSH 登录日志）经常包含明文或弱密码。
不能只盯着 SUID、Capabilities 等常规提权点，要学会检查用户组权限和系统日志。


## 7. 心得体会

Easy 难度不代表思路简单。很多 Easy 机器的提权点其实藏在“非主流”位置（日志、组权限、IDOR）。
我一直习惯性去找 SUID / pkexec，结果绕了很大弯路。枚举时一定要全面，不能只盯着常见提权手法。
以后看到特殊用户组（adm、sudo 等）要第一时间检查其权限。

---

<div align="center">

**Writeup By YRww**

**Keep Hacking**

</div>
