# Centos7.6安装Nginx

## 一键执行安装

一键执行脚本，包括下面所有步骤语句：

```
yum install -y make cmake gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel && wget http://nginx.org/download/nginx-1.21.6.tar.gz && tar zxvf nginx-1.21.6.tar.gz && cd nginx-1.21.6 && ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module && make && make install && ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
```

## 分步安装

### 第一步：安装依赖项和必须组件

```
yum install -y make cmake gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

### 第二步：下载安装Nginx

进入官网：<http://nginx.org/en/download.html>

可以根据需要下载不同版本，复制下载链接，然后在终端运行。示例：

```
wget http://nginx.org/download/nginx-1.12.2.tar.gz
```

### 第三步：解压

使用`tar -zvxf`命令解压下载的Nginx压缩包。

### 第四步：编译配置

```
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module && make && make install
```

执行完本命令将会在 /usr/local/nginx 生成相应的可执行文件、配置、默认站点等文件。

### 第五步：创建全局命令

```
ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
```

> 常用命令：
>
> 启动：nginx
> 重载加载配置：nginx -s reload
> 停止：nginx -s stop