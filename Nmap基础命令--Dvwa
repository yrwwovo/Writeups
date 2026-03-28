# Nmap 基础指令实战 - DVWA 靶场

**作者**：YRwwovo  
**日期**：2026-03-28  
**目标**：DVWA 靶场（XAMPP 环境）  
**目标 IP**：192.168.3.12  
**攻击机**：Kali Linux（虚拟机）

## 1. 实验环境
- 攻击机：Kali Linux
- 靶机：Windows + XAMPP 运行 DVWA
- 工具：Nmap 7.98
- 端口：8080

## 2. 扫描目的
掌握 Nmap 核心参数，理解端口扫描、服务版本识别、OS 指纹、脚本扫描在红队信息收集中的作用。

## 3. 扫描过程与结果

### 命令1：基础 SYN 扫描
```bash
nmap 192.168.3.12 -p 8080
```
**结果**：8080/tcp open http-proxy  

（插入）

### 命令2：服务版本探测（-sV）
```bash
nmap -sV 192.168.3.12 -p 8080
```
**结果**：Apache httpd 2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12  

（插入）

### 命令3：版本 + OS 探测（-sV -O）
```bash
nmap -sV -O 192.168.3.12 -p 8080
```
**结果**：  
- 服务：Apache 2.4.58  
- OS 猜测：Microsoft Windows XP/7/2012 + VMware Player（有 warning）  
**原因**：只扫描单个端口导致 OS 指纹不准  

（插入）

### 命令4：默认脚本扫描（-sC）
```bash
nmap -sC -sV 192.168.3.12 -p 8080
```
**结果**：  
- http-title: Welcome to XAMPP  
- 识别出 XAMPP 默认页面  

（插入）

### 命令5：漏洞脚本扫描（--script vuln）
```bash
nmap --script vuln 192.168.3.12 -p 8080
```
**结果**：  
- 发现 Slowloris DOS 攻击风险（CVE-2007-6750）  
- http-enum 枚举出多个目录（/phpmyadmin、/icons、/img 等）  

（插入）

## 4. 学到的知识点
- Nmap 默认使用 TCP SYN 扫描，速度快且隐蔽性较高。
- `-sV` 可以准确识别服务版本，是后续查找 CVE 的重要依据。
- `-O` 做 OS 指纹识别时，只扫单个端口结果容易不准。
- `-sC` 默认脚本能自动读取网页标题、枚举目录等信息。
- `--script vuln` 可以检测已知漏洞，但实验靶场通常扫不出高危 CVE。

## 5. 红队应用总结
在红队渗透中，Nmap 是信息收集的第一步。通过端口扫描和服务版本识别，能快速定位攻击面；脚本扫描可以自动发现很多人工容易忽略的信息，为后续漏洞利用提供可靠情报。
