### 安装
1. 到[官网](https://redis.io/download)下载稳定版本，本文档所使用的版本为 4.0.11，下载并安装：
```
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
tar xzf redis-4.0.11.tar.gz
cd redis-4.0.11
make
```
2. make完后 redis-2.8.17目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下：

3. 下面启动redis服务.
```
cd src
./redis-server
```
注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动。redis.conf 是一个默认的配置文件。我们可以根据需要使用自己的配置文件。
```
cd src
./redis-server ../redis.conf
```
4. 启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 比如：
```
cd src
./redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```
5. 停止服务:
```
 ./redis-cli -h 127.0.0.1 -p 6379 shutdown
```

### 其它
- 在redis 中存储中文，读取会出现乱码（其实不是乱码，只是不是我们存的中文显示）,如何在get时取到它的中文呢？只需要在redis-cli 后面加上 --raw
```
./redis-cli --raw
```
