# 手机中国 product.cnmo.com 反射型 XSS 漏洞（补天真实提交）

## 基本信息
- **平台**：补天平台  
- **厂商**：手机中国（CNMO）  
- **漏洞编号**：QTVA-2026-10493798  
- **漏洞类型**：反射型 XSS（Reflected XSS）  
- **提交时间**：2026-03-28  
- **当前状态**：已提交，待审核

## 环境
- **目标**：https://product.cnmo.com  
- **工具**：Burp Suite（Proxy + Repeater）  
- **测试方法**：手动构造 payload

## 漏洞描述
product.cnmo.com 搜索接口及产品详情页存在反射型 XSS 漏洞。攻击者可构造恶意链接，诱导用户点击后在浏览器中执行任意 JavaScript 代码。

## 复现过程
1. 访问搜索页面：https://product.cnmo.com/search  
2. 在参数 `s=` 中输入 payload：`<script>alert(1)</script>`  
3. 点击搜索后，payload 被原样反射到页面并成功执行，弹出 `alert(1)` 对话框。  
4. 在搜索结果中点击任意产品进入详情页，同一 payload 依然触发（影响范围更大）。

**关键 Burp Repeater 截图**（请替换为你的实际图片）：
- 请求与响应（payload 被反射到 HTML）
- 搜索页弹窗
- 产品详情页弹窗（OPPO Find N6 示例）

## 知识点总结
- 反射型 XSS 常见于搜索框、参数直接拼接输出到页面的场景
- 即使网站有基础过滤，仍可通过 Burp Repeater 反复测试不同上下文（属性、标签、事件）
- 子域名（product.cnmo.com）同样在厂商资产范围内

## 红队实战Tips（简历最爱看这段）
- 真实 SRC 中搜索接口是最常见的低危入口之一，优先测 s=、q=、keyword= 等参数
- 发现反射点后立刻验证影响范围（搜索页 → 详情页 → 其他页面），一份报告可覆盖多个触发点
- 此类 XSS 可用于钓鱼、偷 Cookie、劫持会话，后续常配合 C2 实现进一步利用
- 防御端建议使用预编译输出或严格的 HTML 实体编码

## 修复建议
- 对用户输入进行 HTML 实体编码（htmlspecialchars 或等价函数）
- 输出前对特殊字符转义
- 添加响应头：`X-XSS-Protection: 1; mode=block`

## 参考
- 补天报告：QTVA-2026-10493798
