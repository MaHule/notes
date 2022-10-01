## Socket模块

> 提供两个基本的socket模块：服务端Socket和客户端Socket。当创建一个服务器端Socket之后，这个Socket会在本机的一个端口上等待连接，客户端Socket访问这个端口后，两者完成连接就可以进行交互。

**Socket服务端建立：**  首先实例化一个Socket类，然后开始循环监听，一直等待接收来自客户端的连接。成功建立连接之后，接收客户端发来的数据，并向客户端发送数据，传输完毕之后，关闭这次连接。

**Socket客户端建立：** 实例化一个Socket类之后连接一个远程的地址，这个地址由IP和端口组成。成功建立连接之后，开始发送和接收数据，传输完毕之后，关闭这次连接。

### 1. Socket实例化

```python
s=socket.socket(family,type[,protocal])
```

- family : 使用的地址族。常用的地址族有：AF_INET、AF_INET6、AF_LOCAL（或称AF_UNIX   UNIX域Socket）、AF_ROUTE等。默认值为socket.AF_INET。

- type ： 指明Socket类型。使用值有三个（默认为sock_STREAM）：
- - sock_STREAM，TCP类型，保证数据顺序及可靠性。
  - sock_DGRAM，UCP类型，不保证数据接收的顺序，非可靠连接。
  - sock_RAM，原始类型，允许对底层协议如IP和ICMP进行直接访问，基本不会用到
- protocal：指使用的协议，参数可选，通常赋值为“ 0 ”，由系统自动选择

```python
#常用实例
s = socket.socket()   #使用默认值，为TCP连接
s = socket.socket(socket.AF_INET,socket.sock_DGRAM)   #建立一个UDP类型socket
```



### 2.Socket常用函数

#### （1）服务端函数

1. bind()：由服务器端Socket调用，会将之前创建的Socket与指定的IP地址和端口绑定。如果之前使用AF_INET初始化Socket，那么这里可以使用元组（host，port）的形式表示地址。

```python
s.bind(('127.0.0.1',2345))
```

2. listen()：用于在使用TCP的服务器端开启监听模式，可以使用一个参数来指定挂起的最大连接数量。这个参数的值最小为1，一般设置为5。

```python
s.listen(5)
```

3. accept():用于在使用TCP的服务器端接收连接，一般是阻塞态。接收TCP连接并返回（conn，address），其中，conn是新的套接字对象，可以用来接收和发送数据；address是连接客户端的地址。

#### （2）客户端函数

1. connect()：这个函数用在使用TCP的客户端去连接服务器端时，使用的参数是一个元组，形式为（hostname，port）。

```python
s.connect(("127.0.0.1",2345))   #连接本机2345端口
```

#### （3）通用函数

1. send()：这个函数在使用TCP时用于发送数据，完整的形式为send(string[,flag])，利用这个函数可以将string代表的数据发送到已经连接的Socket，返回值是发送字节的数量，但是可能未全部发送指定的内容。

2. sendall()：这个函数与send()相似，也在使用TCP时用于发送数据，完整的形式为sendall(string[,flag])，与send()的区别是它可以完整发送TCP数据。将string中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据。成功返回None，失败则抛出异常。

```python
s.sendall(bytes("Hello,My Friend!",encoding="utf-8"))  #发送一段字符到连接的socket
```

3. recv()：这个函数在使用TCP时用于接收数据，完整的形式为recv(bufsize[,flag])，接收Socket的数据。数据以字符串形式返回，bufsize指定最多可以接收的数量。一般不会使用参数flag。

```python
obj.recv(1024)   #接收一段长度为1024字符的Socket
```

4. sendto()：这个函数用于在使用UDP时发送数据，完整的形式为sendto(string [,flag],address)，返回值是发送的字节数。address是形式为（ipaddr，port）的元组，指定远程地址。

5. recvfrom()：UDP专用，接收数据，返回数据远端的IP地址和端口，但返回值是（data，address）。其中，data是包含接收数据的字符串，address是发送数据的套接字地址。

6. close()：关闭Socket。



### 3.Socket简单通信服务器和客户端

```python
#Socket服务端
import socket

s1 = socket.socket()
s1.bind(('127.0.0.1',2345))
s1.listen(5)
str = "Hello World"
while 1:
    conn,address = s1.accept()
    print("a new connect from",address)
    conn.send(str.encode())
    conn.close
```

```python
import socket

s2 = socket.socket()
s2.connect(('127.0.0.1',2345))
#对传输数据使用encode()函数处理，Python3不再支持str类型传输，需要转换为bytes类型
data = bytes.decode(s2.recv(1024))
s2.close()
print(data)
```

