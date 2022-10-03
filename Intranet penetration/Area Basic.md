# 内网渗透|初始域基础

> 

## 第一部分 内网基础知识点

> **内网**也指局域网，是指在某一区域由多台计算机互连而成的计算机组或域。

### 1.工作组

​		对局域网中的计算机进行分类，使得网络更有序。计算机的管理依然是各自为政，所有计算机依然是对等的，松散会员制，可以随意加入和退出，且不同工作组之间的共享资源可以相互访问。

### 2. 域

> 分类：单域、子域、父域、域树、域森林、DNS域名服务器

> 有安全边界的计算机组合（一个域中的用户无法访问另一个域中的资源），域内资源由一台域控制器（Domain Controller，DC）集中管理，用户名和密码是放在域控制器去验证的。域内主机可以通过组策略统一管理

> **域控制器：** 存在由这个域账户，密码，属于这个域的计算机等信息构成的数据库。当计算机连接到域时，域控制器首先级别这台计算机是否属于这个域，以及用户使用的登录账号是否存在，密码是否正确。域控制器是整个域的通信枢纽，所有的权限身份验证都在域控制器上进行。

（1）单域

​		即只有一个域的网络环境，一般需要两台DC，一台DC，另一台备用DC（容灾）

（2）父子域

​		类比公司总部和公司分部的关系，总部的域称为父域，各分部的域称为该域的子域。使用父子域的好处：

- 减小了域之间信息交互的压力（域内信息交互不会压缩，域间信息交互可压缩）
- 不同的子域可以指定特定的安全策略

（3）域树

​		多个域通过建立信任关系组成的集合。若两个域之间需要相互访问，需要建立信任关系（Trust Relation），通过**信任关系可以将父子域连接成树状结构**。

（4）域森林

​		多个域树通过建立信任关系组成的集合。

（5）域名服务器

​		实现域名到IP地址的转换。由于域中计算机使用DNS来定位DC、服务器和其他计算机的，所以域的名字就是DNS域的名字。

​		内网渗透中，大都是通过寻找DNS服务器来确定域控制器位置（因为DNS服务器和域控制器通常配置在一台机器上）



### 3.活动目录

> 指域环境中提供目录服务的组件

​		用于存储有关网络对象（用户、组、计算机、共享资源、联系人）的信息。基于活动目录有目录服务，用于帮助用户从活动目录中快速找到所需的消息。活动目录使得企业可以对网络环境进行集中管理。（可类比为内网中的索引，里面存储有内网里所有资源的快捷方式）。

​		活动目录的逻辑结构包含组织单元、域、域树、域森林。域树内的所有域共享一个活动目录，因此非常适合进行统一管理。

​		活动目录的功能：

- 账号集中管理
- 软件集中管理
- 环境集中管理
- 增强安全性
- 更可靠，更短的宕机时间

**域和活动目录的区别：** 要实现域环境，其实就是要安装AD。一台计算机安装了AD之后就变成了DC。

### 4. 安全域的划分

![image-20220929225140047](image-20220929225140047.png)

（1）内网（安全级别最高）

​		分为核心区（存储企业最重要的数据，只有很少的主机能够访问）和办公区（员工日常工作区，一般能够访问DMZ，部分主机可以访问核心区）

（2）DMZ（Demilitarized Zone，边界网络，隔离区，安全级别中等）

​		作为内网中安全系统和非安全系统之间的缓冲区，用于对外提供服务，一般可以放置一些必须公开的服务器设施。

（3）外网（Internet，安全级别最低）

拥有DMZ区的网络需要制度3的一些**访问控制策略**：

- 内网可以访问外网

- 内网可以访问DMZ

- 外网不能访问内网

- 外网可以访问DMZ

- DMZ不能访问内网

- DMZ不能访问外网

### 5. 域中计算机的分类

> 域控制器、成员服务器、客户机、独立服务器

• 域控制器：用于管理所有的网络访问，存储有域内所有的账户和策略信息。允许网络中拥有多台域控制器（容灾）
• 成员服务器：安装了服务器操作系统并加入了域，但没有安装活动目录的计算机，主要任务是提供网络资源
• 客户机：安装了其他操作系统的计算机，利用这些计算机和域中的账户就可以登录到域。
• 独立服务器：和域无关，既不加入域，也没有活动目录

### 6. 域内权限

• 域本地组：
• 多域用户访问单域资源
• （访问同一个域），主要用于授予本域内资源的访问权限，可以从任何域中添加用户账号、通用组和全局组。域本地组无法嵌套在其他组中
• 全局组：
• 单域用户访问多域资源
• （必须是同一个域中的用户），只能在创建该全局组的域中添加用户和全局组，但可以在域森林中的任何域内指派权限，也可以嵌套在其他组中
• 通用组：多域用户访问多域资源，成员信息不保存在域控制器中，而是保存在全局编录（GC）中，任何变化都会导致全林复制

### 7. A-G-DL-P策略

A：用户账户
G：全局组
DL：域本地组
P：许可，资源权限
先将用户账号添加至全局组中，再将全局组添加至域本地组中，然后为域本地组分配资源权限。
