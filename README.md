制作nginx镜像

建议先把需要安装的软件包下载到本地目录

cd /usr/local/src

wget http://nginx.org/download/nginx-1.20.1.tar.gz   #nginx

wget http://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz   #nginx扩展

wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz  #nginx扩展

wget http://www.zlib.net/zlib-1.2.11.tar.gz  #nginx扩展

cd /usr/local/src  #进入目录

vi   /usr/local/src/dockerfile

#基于这个镜像进行操作
FROM centos:7.9.2009

#作者和邮箱
MAINTAINER lingwen lingwen@osyunwei.com

#是指容器里面的路径，为后面的RUN、CMD或者ENTERPOINT操作指定目录
WORKDIR  /usr/local/src

#RUN镜像操作指令
#安装依赖包
RUN yum install -y pcre pcre-devel autoconf automake  bzip2 bzip2*  gcc gcc-c++ gtk+-devel  gettext gettext-devel glibc  make mpfr  openssl openssl-devel patch pcre-devel perl   zlib-devel

#将本地的一个文件或目录拷贝到容器的某个目录里
COPY nginx-1.21.1.tar.gz  pcre-8.44.tar.gz  openssl-1.1.1k.tar.gz  zlib-1.2.11.tar.gz   ./

RUN tar zxvf nginx-1.21.1.tar.gz  && tar zxvf  pcre-8.44.tar.gz  && tar zxvf openssl-1.1.1k.tar.gz  && tar zxvf zlib-1.2.11.tar.gz

#安装pcre 
WORKDIR  /usr/local/src/pcre-8.44
RUN  ./configure --prefix=/usr/local/pcre && make  && make install

#安装openssl
WORKDIR /usr/local/src/openssl-1.1.1k
RUN  ./config -fPIC shared zlib --prefix=/usr/local/openssl && make && make install

#安装zlib
WORKDIR /usr/local/src/zlib-1.2.11
RUN ./configure --prefix=/usr/local/zlib && make && make install

#安装Nginx
WORKDIR /usr/local/src/nginx-1.21.1

#创建nginx运行用户和组
RUN groupadd www &&  useradd -g www www -s /bin/false
RUN ./configure --prefix=/usr/local/nginx --without-http_memcached_module --user=www --group=www --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-openssl=/usr/local/src/openssl-1.1.1k --with-zlib=/usr/local/src/zlib-1.2.11 --with-pcre=/usr/local/src/pcre-8.44  && make && make install

#清理安装包
RUN rm -rf /usr/local/src/ && yum clean all

#切换到根目录
WORKDIR /root

#配置nginx以www用户和组运行
RUN  sed -i "1iuser www www;" /usr/local/nginx/conf/nginx.conf

#设置nginx运行端口
EXPOSE 80/tcp   443/tcp

#设置nginx以foreground前台方式运行
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]

#构建容器
docker  build  -t  osyunwei/nginx:1 .
