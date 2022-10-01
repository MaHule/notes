## Python-nmap模块



### 1.安装python-nmap

```python
pip3 install python-nmap
```



### 2. 基本用法

**python-nmap模块中的核心类：**

<u>PortScanner()</u>    <u>PortScannerAsync()</u>   <u>PortScannerError()</u>  <u>PortScannerHostDict()</u>  <u>PortScannerYield()</u>    

#### (1) python-nmap模块类的实例化

> PortScanner类实现Nmap工具功能的封装。

```python
nmap.PortScanner()     #PortScanner()实例化
```

> PortScannerAsync()与PortScanner()类的功能相同，但是这个类可以实现异步扫描

```python
nmap.PortScannerAsync()     #PortScannerAsync()实例化
```

#### (2)PortScanner类中的函数

1. scan()：scan(self,hosts='127.0.0.1', ports=None, arguments= '-sV', sudo=False)，用来对指定目标进行扫描，其中需要设置的3个参数包括hosts、ports和arguments。

- - 参数hosts的值为字符串类型，表示要扫描的主机，形式可以是IP地址，例如“192.168.1.1”，也可以是一个域名，例如“www.nmap.org”。
- - 参数ports的值也是字符串类型，表示要扫描的端口。如果要扫描的是单一端口，形式可以为“80”。如果要扫描的是多个端口，可以用逗号分隔开，形式为“80,443,8080”。如果要扫描的是连续的端口范围，可以用横线，形式为“1-1000”。
- - 参数arguments的值也是字符串类型，这个参数实际上就是Nmap扫描时所使用的参数，例如“-sP”“-PR”“-sS”“-sT”“-O”“-sV”等。其中，“-sP”表示对目标进行Ping主机在线扫描；“-PR”表示对目标进行一个ARP的主机在线扫描；“-sS”表示对目标进行一个TCP半开（SYN）类型的端口扫描；“-sT”表示对目标进行一个TCP全开类型的端口扫描；“-O”表示扫描目标的操作系统类型；“-sV”表示扫描目标上所安装网络服务软件的版本。

```python
>>> import nmap
>>> nm = nmap.PortScanner()
>>> print(nm.scan('127.0.0.1','5432-5434','-sS'))
{'nmap': {'command_line': 'nmap -oX - -p 5432-5434 -sS 127.0.0.1', 'scaninfo': {'tcp': {'method': 'syn', 'services': '5432-5434'}}, 'scanstats': {'timestr': 'Thu Sep 22 23:12:03 2022', 'elapsed': '0.61', 'uphosts': '1', 'downhosts': '0', 'totalhosts': '1'}}, 'scan': {'127.0.0.1': {'hostnames': [{'name': 'kubernetes.docker.internal', 'type': 'PTR'}], 'addresses': {'ipv4': '127.0.0.1'}, 'vendor': {}, 'status': {'state': 'up', 'reason': 'localhost-response'}, 'tcp': {5432: {'state': 'closed', 'reason': 'reset', 'name': 'postgresql', 'product': '', 'version': '', 'extrainfo': '', 'conf': '3', 'cpe': ''}, 5433: {'state': 'closed', 'reason': 'reset', 'name': 'pyrrho', 'product': '', 'version': '', 'extrainfo': '', 'conf': '3', 'cpe': ''}, 5434: {'state': 'closed', 'reason': 'reset', 'name': 'sgi-arrayd', 'product': '', 'version': '', 'extrainfo': '', 'conf': '3', 'cpe': ''}}}}}
```

2. all_hosts() 返回被扫描所有主机列表

```python
>>> nm.all_hosts()
['127.0.0.1']
```

3. command_line()   返回当前扫描中使用的命令行

```python
>>> nm.command_line()
'nmap -oX - -p 5432-5434 -sS 127.0.0.1'
```

4. csv()  返回被扫描的所有主机的CSV格式的文件

```python
>>> print(nm.csv())
host;hostname;hostname_type;protocol;port;name;state;product;extrainfo;reason;version;conf;cpe
127.0.0.1;kubernetes.docker.internal;PTR;tcp;5432;postgresql;closed;;;reset;;3;
127.0.0.1;kubernetes.docker.internal;PTR;tcp;5433;pyrrho;closed;;;reset;;3;
127.0.0.1;kubernetes.docker.internal;PTR;tcp;5434;sgi-arrayd;closed;;;reset;;3;
```

5. has_host(self,host)    检查是否有扫描结果

```python
>>> nm.has_host('127.0.0.1')
True
```

6. scaninfo()   列出扫描信息的结构

```python
>>> nm.scaninfo()
{'tcp': {'method': 'syn', 'services': '5432-5434'}}
```

7. 支持的其他操作

![image-20220922232118381](D:\Gridea\post-images\image-20220922232118381.png)

#### (3) PortScannerAsync()类中的函数

```python
>>> import nmap
>>> nma = nmap.PortScannerAsync()
>>> nma.scan('192.168.0.1/24',arguments='-sP')
```

1. scan()   函数与PortScanner中的scan()中方法一致
2. still_scanning()  返回是否正在扫描

```
>>> nma.still_scanning()
True
```

3. wait(self,timeout=None)  表示等待时间

```python
>>> nma.wait(2)
```

4. stop() 停止当前扫描

 ```python
>>> nma.stop()
 ```



### 3. python-nmap模块编写一个扫描器

```python
import nmap
target = '127.0.0.1'
port = '80'
nm = nmap.PortScanner()
nm.scan(traget,port)
for host in nm.all_hosts():
    print('-----------------------------------------------------')
    print('Host: {0}({1})'.format(host,nm[host].hostname()))
    print('State:{0}'.format(nm[host].state()))
    for proto in nm[host].all_protocols():
        print('----------------------------')
        print('Protocol:{0}'.format(proto))
        Iport = list(nm[host][proto].keys())
        Iport.sort()
        for port in Iport:
            print('port:{0}\tstate:{1}'.format(port,nm[host][proto][port]['state']))
        
```



**只扫描活跃主机：**

```python
import nmap 
nm = nmap.PortScanner()
nm.scan(hosts='192.168.0.0/24',arguements='-sP')
hosts_list = [(x,nm[x]['status']['state']) for x in nm.all_hosts()]
for host,status in hosts_list:
    print(host + "is" + status)
```

