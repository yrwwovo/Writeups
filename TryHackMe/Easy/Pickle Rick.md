# Pickle Rick

**难度**: Easy  
**机器**: Pickle Rick v2  
**完成日期**: 2026-04-06  

## 目标信息
- **目标**: 帮助 Rick 找到 3 种原料，制作反转药水，让他从泡菜变回人类。
- **最终目标**: 找到 `ingredient1`、`ingredient2`、`ingredient3`

## 1. 信息收集

### 1.1 查看页面源代码
在主页 `http://<MACHINE_IP>` **View Page Source**在 HTML 注释中找到用户名：

```html
<!-- Note to self, remember username! -->
Username: RickRul3s
```

**用户名**: `RickRul3s`

### 1.2 robots.txt 枚举
访问 `http://<MACHINE_IP>/robots.txt`，发现神秘字符：

```
Wubba Lubba Dub Dub
```

**猜测密码**: `Wubba Lubba Dub Dub`

### 1.3 登录后台
访问 `http://<MACHINE_IP>/portal.php`，使用上面凭证登录，进入 **Command Panel**。

## 2. 命令注入 + 黑名单绕过

Command Panel 对 `cat`、`cd` 等常用命令做了黑名单限制，但 **`less`** 命令未被禁用。
（并且sudo -l可以发现无密码可以使用任何指令，除了黑名单）
### 2.1 第一个原料 (ingredient1)
```bash
ls #发现文件
less Sup3rS3cretPickl3Ingr3d.txt
```

### 2.2 第二个原料 (ingredient2)
先列目录找到文件名：
```bash
ls /home/rick
```

发现文件名为 `second ingredients`（带空格），读取命令：
```bash
less "second ingredients"
```

### 2.3 第三个原料 (ingredient3)
直接读取 root 目录下的文件：
```bash
sudo ls /root #发现第三种原料3rd.txt
less /root/3rd.txt
```

**三个原料全部获取成功！**

## 总结
- 通过页面源代码 + robots.txt 完成初始 Recon
- 利用 Command Injection + 黑名单绕过（使用 `less`）成功获取全部 3 个原料
- 重点：**命令绕过和web信息收集**

---

<div align="center">

**Writeup By YRww**

**Keep Hacking**

</div>
