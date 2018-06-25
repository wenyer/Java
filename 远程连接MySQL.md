# 远程连接云主机 MySQL 遇见的问题  
## 连不上的原因
### 1、未启动MySQL服务  
    service mysql status  
解决：开启服务   
    service mysql start 

### 2、3306端口异常  
    telnet 39.106.100.144 3306  无法连接   
解决：关闭防火墙或开启3306端口，<font color=red>注意云主机要在云主机客户端开启3306端口，比如说阿里云主机3306端口是默认不开启的，此时即便关闭防火墙也telnet不通3306，需要在阿里云主机的安全组设置中设置</font>   
### 3、不接受tcpip连接
解决：查看my.cnf，老版本可能是mysqld.cnf 中的 skip-networking 应当被注释掉（<font color=red>如果没有意味着不会接受tcpip的连接</font>） 
### 3、只接受本地的连接，不能远程访问（MySQL默认配置）  
    netstat -an | grep 3306   
如果显示的是： tcp&nbsp;&nbsp;&nbsp;0&nbsp;&nbsp;0&nbsp;&nbsp;&nbsp;&nbsp;  127.0.0.1:3306        &nbsp;&nbsp;&nbsp;&nbsp;0.0.0.0：*  
<font color=red>说明是只接受127.0.0.1 的访问，即只接受本机访问（MySQL默认配置）</font>
解决：修改 my.cnf，老版本可能是mysqld.cnf 中的 bind-address，将其注释掉意味着所有IP都能访问，将其改为某个IP意味着只有这个IP可以访问（默认是127.0.0.1即只允许本机访问）  
### 4、MySQL用户未添加远程访问权限  
解决：root进入MySQL 

	 grant all privileges on *.* to root@"%" identified by "password" with grant option;  
第一行命令解释如下，\*.\*：第一个\*代表数据库名；第二个\*代表表名。这里的意思是所有数据库里的所有表都授权给用户。root：授予root账号。“%”：表示授权的用户IP可以指定，这里代表任意的IP地址都能访问MySQL数据库。“password”：分配账号对应的密码，这里密码自己替换成你的mysql root帐号密码。
然后重启MySQL服务  
	
    service mydql restart
