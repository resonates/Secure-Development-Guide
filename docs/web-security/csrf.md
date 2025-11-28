# CSRF跨站请求伪造 (Cross-Site Request Forgery)

## 什么是CSRF？

跨站请求伪造（Cross-Site Request Forgery，简称CSRF）是一种常见的Web安全漏洞，攻击者诱导用户在已认证的Web应用程序上执行非预期的操作。攻击者通过欺骗用户的浏览器向目标站点发送请求，利用用户的已认证状态来执行未授权的操作。

## CSRF的工作原理

CSRF攻击的核心是利用用户的已认证会话。当用户登录到一个受信任的网站后，其浏览器中会保存该网站的认证Cookie。如果该网站没有实施适当的CSRF防护措施，攻击者可以通过欺骗用户访问一个包含精心构造请求的页面，让浏览器在用户不知情的情况下使用保存的认证Cookie向目标网站发送请求。

## CSRF攻击的特点

1. **攻击者无法直接获取用户的Cookie**：CSRF攻击不会窃取用户的数据，而是利用用户的已认证状态执行操作。
2. **需要用户主动访问攻击页面**：攻击者需要诱导用户点击链接或访问包含恶意代码的页面。
3. **攻击成功依赖于目标站点缺乏CSRF保护**：如果目标站点实施了适当的CSRF保护措施，攻击将无法成功。

## CSRF攻击示例

### 银行转账场景

假设一个银行网站的转账功能使用以下HTTP请求：

```http
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: sessionid=abc123

amount=1000&destination=attacker_account
```

攻击者可以创建一个包含以下内容的恶意页面：

```html
<img src="http://bank.example.com/transfer?amount=1000&destination=attacker_account" style="display:none">
```

当已登录银行网站的用户访问这个恶意页面时，浏览器会自动向银行网站发送请求，由于请求包含用户的认证Cookie，银行网站会认为这是用户的合法操作并执行转账。

### 更改密码场景

攻击者可以构造一个自动提交的表单，修改用户的密码：

```html
<form id="csrf-form" action="http://example.com/change-password" method="POST">
    <input type="hidden" name="new_password" value="attacker_password">
    <input type="hidden" name="confirm_password" value="attacker_password">
</form>
<script>
    document.getElementById('csrf-form').submit();
</script>
```

## CSRF的防御措施

### 1. 实施CSRF令牌（Token）

这是防御CSRF最有效的方法之一。服务器为每个用户会话生成一个唯一的令牌，并将其嵌入到页面中或存储在Cookie中。当用户提交表单或发送请求时，需要同时提交这个令牌。服务器验证令牌的有效性，只有令牌有效才处理请求。

**实现示例（使用Flask）：**

```python
from flask import Flask, render_template, request, session
import secrets

app = Flask(__name__)
app.secret_key = 'your_secret_key'

def generate_csrf_token():
    if '_csrf_token' not in session:
        session['_csrf_token'] = secrets.token_hex(16)
    return session['_csrf_token']

app.jinja_env.globals['csrf_token'] = generate_csrf_token

@app.route('/transfer', methods=['POST'])
def transfer():
    token = request.form.get('csrf_token')
    if not token or token != session.get('_csrf_token'):
        return 'CSRF token validation failed', 403
    # 处理转账逻辑
    return 'Transfer successful'
```

模板中的表单：

```html
<form action="/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
    <input type="text" name="amount" placeholder="Amount">
    <input type="text" name="destination" placeholder="Destination">
    <button type="submit">Transfer</button>
</form>
```

### 2. 验证Referer和Origin头

服务器可以检查请求的Referer和Origin头，确保请求来自受信任的域。

```python
@app.route('/transfer', methods=['POST'])
def transfer():
    referer = request.headers.get('Referer')
    origin = request.headers.get('Origin')
    
    # 检查Referer或Origin是否来自信任的域
    trusted_domain = 'http://example.com'
    if not (referer and referer.startswith(trusted_domain)) and not (origin and origin == trusted_domain):
        return 'Invalid referer/origin', 403
    
    # 处理转账逻辑
    return 'Transfer successful'
```

### 3. 实施SameSite Cookie属性

设置Cookie的SameSite属性可以限制Cookie只在同源请求中发送，有效防止CSRF攻击。

```
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict
```

SameSite属性有三个可能的值：
- **Strict**：Cookie只在同源请求中发送
- **Lax**：Cookie在导航到目标URL的GET请求中发送（默认值）
- **None**：Cookie在所有请求中发送，但需要设置Secure属性

### 4. 要求用户交互

对于敏感操作，可以要求用户重新输入密码或提供额外的确认步骤。

### 5. 使用自定义HTTP头

对于AJAX请求，可以使用自定义HTTP头，如`X-Requested-With: XMLHttpRequest`，服务器验证该头的存在。

## 安全编码最佳实践

1. **为所有状态改变的操作实施CSRF保护**：特别是涉及数据修改、资金转账、密码更改等敏感操作。
2. **使用双重提交Cookie模式**：将CSRF令牌同时存储在Cookie和表单中，服务器验证两者是否匹配。
3. **保持令牌的唯一性和随机性**：每个会话生成不同的令牌，使用强随机数生成器。
4. **设置适当的Cookie属性**：使用HttpOnly、Secure和SameSite属性增强Cookie安全性。
5. **实施多层次防御**：结合多种CSRF防御措施，提高安全性。

## 总结

CSRF是一种常见的Web安全漏洞，可以通过多种方式进行防御。实施CSRF令牌、验证Referer和Origin头、设置SameSite Cookie属性等措施可以有效地保护Web应用程序免受CSRF攻击。在开发过程中，应当对所有可能改变应用状态的操作实施适当的CSRF保护。