## 1. 阿里云

### 1.1 ECS购买

- 系统选择ubuntu

### 1.2 重置密码

### 1.3 实例安全组

- 配置http协议

### 1.4 连接远程服务

- 管理=》远程连接
- login：root
- 密码：XXXX

### 1.5 帮助信息

官网帮助地址：https://help.aliyun.com/knowledge_detail/41091.html?spm=a2c4e.11153987.0.0.78be23e3soHhoS



## 2. 远程连接

### 2.1 连接

> 通过git bash进行远程连接，也可以选择其他工具

```shell
ssh root@公网ip地址
```

```shell
# 基本操作
exit 退出
```



## 3. Nginx安装

- 安装教程：https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04


- 文章第二部关于防火墙的设置可以省略不做

```shell
# 上述成功完成检查
http://公网ip
```



## 4. Node安装

> 必须先安装nvm

github地址：https://github.com/creationix/nvm

```shell
# 下载nvm
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```

```shell
# 启动服务
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

```shell
# 查看版本 结果只显示node
command -v nvm
```

```shell
# 安装node
nvm install 10.0.0
```

```shell
# 查看node列表
nvm list
```



## 5. 后台文件上传

上传前忽略文件操作，.gitignore   至少将node_modules包文件忽略掉

```shell
# 仓库初始化
git init
git add -A
git commit -m 'first comiit'
# 设置地址别名方便下次提交
git remote add origin https://github.com/cnloop/blog-api.git 
git push origin master 
```

## 6. Git安装

```shell
# linux系统中安装git
sudo apt install git
# 检车安装是否成功
git --version
```

```shell
# 查看是否在根目录下
pwd 
# 从github上将文件下载下来
git clone https://github.com/cnloop/blog-api.git
# 查看是否下载成功
ls
# 切换到项目目录中
cd blog-app/
# 下载依赖包
npm install
```





## 7. Mysql安装

安装教程：https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04

```shell
# 连接数据库，会提交输入密码，最好先cd到blog-api中进行数据库连接，因为下面到导入sql源文件
mysql -u root -p
```

```mysql
# 创建数据库
create database blog default charset=utf8;
# 切换数据库
use blog;
# 显示已经创建的表
show tables;
# 载入资源文件，自动创建表，路径是源文件的相对路径，不知道可以先exit查看当前目录
source ./blog.sql;
```



## 8. 启动服务

> 远程连接窗口一旦关闭启动的服务也会随之停止，forever可以让程序后台永久运行

官网地址：https://github.com/foreverjs/forever

安装命令

```shell
npm install -g forever
```

基本操作

```shell
# 启动服务
forever start app.js
# 查看正在后台运行的服务
forever list
# 停止单个服务
forever stop app.js
# 停止所有服务
forever stopall
```

具体操作

```shell
# 切换到项目目录
cd ......
# 启动app.js
forever start app.js
# 查看是否启动
forever list
```

## 9. 反向代理配置

```shell
# 进入nginx配置文件
vi /etc/nginx/nginx.conf
# http中添加配置项
# proxy_pass指向的是本地app.js开启的端口
 server {
     listen   80;
     server_name    blog.cnloop.link;
     location / {
     	proxy_set_header  x-Real-IP $remote_addr;
     	proxy_set_header  Host      $http_host;
     	proxy_pass        http://127.0.0.1:3000; 
     }
 }
# 重启nginx服务
nginx -s reload
```

注意域名提前在阿里云中解析



## 10. 前端文件上传

> build之前最好将第三方包改成min压缩版

```shell
# 会创建一个dist目录，将package.json package-lock.json放入其中
npm run build 
```

```shell
# 将dist目录上传到远程服务器 -r是上传目录时候才加的是递归的意思
scp -r dist/ root@主机ip地址:/root
```

```shell
# 登陆远程主机 一定目录到默认www文件夹
mv dist/ /var/www/html/
# 切换到默认www目录
cd /var/www/html/
# 切换到dist目录下将文件都移动到上级目录也就是www目录
mv *.* ../
# 切换到上级目录安装第三方包记住加--production，避免安装工具包
cd ..
npm install --production
```



## 11. 配置文件

> 静态页面的nginx直接返回，需要向node后台请求数据的走代理

```shell
# 开发过程向后台请求的接口我们都加了/api,所以还是需要对配置文件进行修改
vi /etc/nginx/nginx.conf
```

```shell
# 在server中添加如下设置
location  /api {
  rewrite /api/(.*) /$1  break;
  proxy_pass         http://127.0.0.1:3000;
  proxy_redirect     off;
  proxy_set_header   Host $host;
}
location / {
    root   /var/www/html; 
    index  index.html index.htm;
}
```

```shell
# 重启nginx
nginx -s reload
```

还有一种方法：

```shell
# 在路由挂载到app中的时候将api也挂载进去
app.use('/api',router)
# 重启app.js
forever restart app.js
```

参考教程：

[URL rewrite](https://serverfault.com/questions/379675/nginx-reverse-proxy-url-rewrite)

[反向代理](https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/reverse_proxy.html?q=)

[反向代理](https://www.jianshu.com/p/bed000e1830b)