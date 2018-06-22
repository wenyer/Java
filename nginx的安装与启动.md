#nginx的安装与启动  
##1、nginx的安装  
[安装参考1](http://www.cnblogs.com/gmq-sh/p/5750833.html)  
[安装参考2](https://www.cnblogs.com/wyd168/p/6636529.html)  
###1.1确认gcc g++开发类库已经安装，默认已经安装  
Ubuntu平台使用以下指令：  

	apt-get install build-essential
	apt-get install libtool  
centos平台使用以下指令：  
安装make:  
  
    yum -y install gcc automake autoconf libtool make  
安装g++:  

	yum install gcc gcc-c++

###1.2 本文以安装文件为/usr/local/src为例
###1.3安装pcre库  
	cd /usr/local/src  

	wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz 
	tar -zxvf pcre-8.37.tar.gz
	cd pcre-8.34
	./configure
	make
	make install  
###1.4 安装zlib库  
	cd /usr/local/src
 
	wget http://zlib.net/zlib-1.2.11.tar.gz
	tar -zxvf zlib-1.2.11.tar.gz
	cd zlib-1.2.11
	./configure
	make
	make install  
###1.5 安装openssl(某些vps默认没装ssl)  
	cd /usr/local/src  

	wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
	tar -zxvf openssl-1.0.1t.tar.gz  
###1.6 安装nginx  
	cd /usr/local/src
	
	wget http://nginx.org/download/nginx-1.1.10.tar.gz
	tar -zxvf nginx-1.1.10.tar.gz
	cd nginx-1.1.10
	./configure
	make
	make install  
这里可能会报错：。。。。No rule to make target 'build',needed by 'default':停止，如果报错了，就要按 1.5 安装ssl  

##2、启动、关闭与重启  
###2.1 启动（注意Ubuntu需要加 sudo） 
	cd /usr/local/nginx/sbin
	./nginx  
或者

	/usr/local/nginx/sbin/nginx -c /usr/local/src/nginx/(就是安装目录)nginx.conf  
###2.2 停止  
 	ps -ef|grep nginx （查进程号） 
 	kill -QUIT 进程号  
###2.3 重启  
	cd /usr/local/nginx/sbin
	./nginx  -s reload
#3 访问nginx  
可以在nginx.conf中查看和修改端口号，注意默认是80端口可能和Apache冲突  
直接在浏览器访问 主机名：端口号  即可看见nginx的欢迎界面。





