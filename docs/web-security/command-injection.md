# 命令注入 (Command Injection)

## 什么是命令注入？

命令注入是一种安全漏洞，攻击者通过在Web应用程序中插入恶意命令，使服务器执行这些命令。这种漏洞通常发生在应用程序调用系统命令来处理用户输入，但没有正确验证或过滤这些输入的情况下。

命令注入攻击可能导致服务器被完全控制，包括数据泄露、服务中断、权限提升等严重后果。

## 命令注入的工作原理

命令注入的核心原理是应用程序将用户输入直接传递给系统命令执行函数。攻击者可以通过在输入中添加特殊字符（如分号、管道符等）来终止原始命令并执行自己的命令。

## 常见的命令注入场景

1. **操作系统命令执行**：应用程序调用`system()`、`exec()`等函数执行系统命令
2. **数据库命令执行**：通过数据库执行操作系统命令（如MySQL的`xp_cmdshell`）
3. **网络命令执行**：应用程序调用网络相关命令处理用户输入
4. **文件操作命令执行**：应用程序使用命令行工具进行文件操作

## 命令注入示例

### Python中的命令注入

假设有以下Python代码，用于ping一个IP地址：

```python
# 不安全的Python代码示例
import os

ip_address = request.form['ip_address']
command = f"ping -c 4 {ip_address}"
result = os.system(command)  # 危险：直接执行用户输入的命令
```

攻击者可以输入以下内容来执行额外的命令：

```
127.0.0.1; cat /etc/passwd
```

生成的命令将变成：
```
ping -c 4 127.0.0.1; cat /etc/passwd
```

这将首先ping本地主机，然后显示系统的密码文件。

### PHP中的命令注入

假设有以下PHP代码：

```php
<?php
// 不安全的PHP代码示例
$filename = $_GET['filename'];
$result = shell_exec("cat /var/www/uploads/" . $filename);
?>
```

攻击者可以输入：
```
; ls -la /etc/
```

生成的命令将变成：
```
cat /var/www/uploads/; ls -la /etc/
```

这将显示`/etc/`目录中的所有文件。

### 常见的命令分隔符

不同操作系统中常用的命令分隔符：

| 分隔符 | 作用 | 适用操作系统 |
|--------|------|------------|
| ; | 顺序执行多个命令 | Unix/Linux, Windows |
| & | 后台执行命令 | Unix/Linux, Windows |
| && | 只有前一个命令成功执行才执行下一个命令 | Unix/Linux, Windows |
| \| | 管道，将前一个命令的输出作为后一个命令的输入 | Unix/Linux, Windows |
| \|\| | 只有前一个命令执行失败才执行下一个命令 | Unix/Linux, Windows |
| $() | 命令替换，执行括号内的命令并返回结果 | Unix/Linux |
| `` ` `` | 命令替换，执行反引号内的命令并返回结果 | Unix/Linux |
| \ | 命令换行符 | Unix/Linux, Windows |

## 命令注入的防御措施

### 1. 避免直接执行系统命令

尽可能避免在应用程序中执行系统命令。考虑使用编程语言提供的API来完成相同的任务。

```python
# 使用Python的subprocess模块的安全方式
import subprocess

def ping_host(ip_address):
    # 只允许IP地址格式的输入
    import re
    if not re.match(r'^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$', ip_address):
        raise ValueError("Invalid IP address format")
    
    # 使用参数列表而不是字符串
    result = subprocess.run(['ping', '-c', '4', ip_address], 
                          stdout=subprocess.PIPE, 
                          stderr=subprocess.PIPE,
                          text=True)
    return result.stdout
```

### 2. 使用白名单验证

对用户输入进行严格的白名单验证，只允许预期的字符或格式。

```python
# 白名单验证示例
import re

def validate_input(input_str):
    # 只允许字母、数字、点和连字符
    if not re.match(r'^[a-zA-Z0-9.-]+$', input_str):
        raise ValueError("Invalid input format")
    return input_str

ip_address = request.form['ip_address']
valid_ip = validate_input(ip_address)
# 现在可以安全地使用valid_ip
```

### 3. 使用安全的函数和API

使用编程语言提供的安全函数，避免使用容易导致命令注入的函数：

- **不安全的函数**：`system()`, `exec()`, `` ` `` (反引号), `shell_exec()`等
- **安全的函数**：
  - Python: `subprocess.run()`（带参数列表）
  - PHP: `exec()`（带数组参数）, `proc_open()`（带描述符）
  - Node.js: `child_process.execFile()`

### 4. 最小权限原则

应用程序运行时使用的账户应该只被授予完成其工作所需的最小权限。避免使用root或管理员账户运行Web应用程序。

### 5. 输入清理和转义

在无法避免执行系统命令的情况下，对用户输入进行适当的清理和转义。

```python
# 使用shlex.quote进行命令转义
import shlex

user_input = request.form['input']
safe_input = shlex.quote(user_input)
command = f"safe_command {safe_input}"
```

### 6. 使用容器化或沙箱环境

将可能执行系统命令的代码放在容器或沙箱环境中运行，限制其对系统的访问权限。

## 安全编码最佳实践

1. **永远不要直接执行用户输入**：避免将用户输入直接传递给命令执行函数
2. **使用白名单验证**：基于白名单验证输入，而不是黑名单
3. **使用参数化函数**：使用接受参数列表而不是字符串的函数来执行系统命令
4. **实施最小权限原则**：以最小必要权限运行应用程序
5. **使用应用级防火墙**：配置Web应用防火墙(WAF)来检测和阻止命令注入攻击
6. **定期进行安全测试**：使用工具如OWASP ZAP或手动测试来发现命令注入漏洞

## 总结

命令注入是一种严重的安全漏洞，可能导致服务器完全被控制。通过采用适当的防御措施，如避免直接执行系统命令、使用白名单验证、选择安全的函数和API等，可以有效地防止大多数命令注入攻击。在开发过程中，应当始终将安全放在首位，对可能导致安全问题的操作保持警惕。