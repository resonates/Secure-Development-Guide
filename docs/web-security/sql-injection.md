# SQL注入 (SQL Injection)

## 什么是SQL注入？

SQL注入是一种常见的Web安全漏洞，攻击者通过在用户输入中插入恶意SQL代码，使应用程序执行非预期的数据库操作。这可能导致数据泄露、数据篡改、绕过认证甚至服务器被接管等严重后果。

## SQL注入的工作原理

当Web应用程序直接将用户输入拼接到SQL查询语句中，而没有进行适当的验证或转义时，就可能发生SQL注入。攻击者可以利用这一点修改SQL查询的逻辑，执行自己想要的操作。

## 常见的SQL注入类型

### 1. 基于错误的SQL注入

攻击者通过构造特殊输入，使应用程序返回数据库错误信息，从而获取数据库结构等敏感信息。

### 2. 基于布尔的盲注

攻击者通过发送不同的查询，观察应用程序的响应（真/假）来推断数据库中的信息。

### 3. 基于时间的盲注

攻击者通过构造使数据库执行延迟的查询，根据响应时间来判断查询结果。

### 4. 堆叠查询注入

攻击者在一个请求中执行多个SQL语句，通过分号(;)分隔。

## SQL注入示例

### 简单的登录绕过

假设有以下登录验证代码：

```python
# 不安全的Python代码示例
username = request.form['username']
password = request.form['password']
query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
cursor.execute(query)
```

攻击者可以使用以下输入绕过登录：
- 用户名: `admin' --`
- 密码: 任意值

生成的SQL将变成：
```sql
SELECT * FROM users WHERE username='admin' --' AND password='任意值'
```

由于`--`在SQL中表示注释，后面的条件被忽略，只要存在用户名为'admin'的用户，攻击者就能成功登录。

### 数据提取示例

攻击者可以使用UNION查询注入来提取数据库中的其他信息：

```
' UNION SELECT username, password FROM users --
```

## SQL注入的防御措施

### 1. 参数化查询（预编译语句）

使用参数化查询是防御SQL注入最有效的方法之一。在参数化查询中，SQL语句结构和数据是分开处理的。

**Python (使用psycopg2) 示例：**

```python
# 安全的参数化查询示例
query = "SELECT * FROM users WHERE username = %s AND password = %s"
cursor.execute(query, (username, password))  # 参数作为元组传递
```

**Node.js (使用mysql2) 示例：**

```javascript
// 安全的参数化查询示例
const [rows] = await connection.execute(
  'SELECT * FROM users WHERE username = ? AND password = ?',
  [username, password]
);
```

### 2. 使用ORM框架

ORM（对象关系映射）框架通常内置了防SQL注入的机制，如Django ORM、SQLAlchemy、Hibernate等。

**Django ORM示例：**

```python
# 安全的Django ORM查询
user = User.objects.filter(username=username, password=password).first()
```

### 3. 输入验证和过滤

对用户输入进行严格的验证和过滤，确保输入符合预期的格式和长度。

```python
# 简单的输入验证示例
import re

def validate_input(input_str):
    # 只允许字母、数字和下划线
    if not re.match(r'^\w+$', input_str):
        raise ValueError("Invalid input format")
    return input_str
```

### 4. 最小权限原则

数据库用户应该只被授予完成其工作所需的最小权限，避免使用具有高权限的数据库账户。

### 5. 使用存储过程

合理使用存储过程可以减少SQL注入的风险，因为存储过程的参数会被自动转义。

### 6. 转义特殊字符

在无法使用参数化查询的情况下，可以对特殊字符进行转义，但这不是最安全的方法。

```python
# 使用连接池的转义功能
username_escaped = connection.escape_string(username)
password_escaped = connection.escape_string(password)
query = f"SELECT * FROM users WHERE username='{username_escaped}' AND password='{password_escaped}'"
```

## 安全编码最佳实践

1. **永远不要信任用户输入**：对所有用户输入进行验证和清理
2. **使用参数化查询**：始终使用参数化查询而不是字符串拼接
3. **实现输入长度限制**：限制输入的长度，减少攻击面
4. **使用白名单过滤**：基于白名单验证输入，而不是黑名单
5. **对数据库错误进行处理**：避免向用户显示详细的数据库错误信息
6. **定期进行安全测试**：使用工具如SQLmap进行SQL注入测试

## 总结

SQL注入是一种严重的安全漏洞，但通过采用适当的防御措施，特别是使用参数化查询和ORM框架，可以有效地防止大多数SQL注入攻击。在开发过程中，遵循安全编码规范，定期进行安全测试是确保应用程序安全的关键。