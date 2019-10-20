# Centos7 部署Easy Mock服务简单说明


### 1. 环境要求
1. node = v8.x (只支持8.x)
2. MongoDB >= v3.4
3. Redis >= v4.0
4. pm2


### 2. 安装node

[官网下载](https://nodejs.org/zh-cn/download/)

```bash
  # 如果没有安装wget，先安装wget
  sudo yum install wget -y

  # 去node官网获取对应node版本的下载链接
  # 进入需要安装node的目录
  cd /usr/local/

  # 下载node安装包
  sudo wget https://nodejs.org/dist/v8.12.0/node-v8.12.0-linux-x64.tar.xz

  # 解压安装包
  sudo tar -zxvf node-v8.12.0-linux-x64.tar.xz

  # 进入解压后的node目录
  cd node-v8.12.0-linux-x64/bin

  # 添加软连接到全局环境node、npm、npx
  sudo ln -s /usr/local/node-v8.12.0-linux-x64/bin/node /usr/bin/node
  sudo ln -s /usr/local/node-v8.12.0-linux-x64/bin/npm /usr/bin/npm
  sudo ln -s /usr/local/node-v8.12.0-linux-x64/bin/npx /usr/bin/npx

  # 检查是否安装成功
  node -v
  npm -v
  npx -v

```


### 3. 安装pm2

[官方教程](http://pm2.keymetrics.io/docs/usage/quick-start/)

```bash
 
  # 安装pm2
  sudo npm install pm2 -g

  # 把pm2加入到全局环境
  sudo ln -s /usr/local/node-v8.12.0/bin/pm2 /usr/bin/pm2

```


### 4. 安装mongodb
[官方教程](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)

```bash
  # 如果没有安装vim编辑器，请先安装vim编辑器，或直接使用vi编辑器
  sudo yum install vim -y

  # 新建官方推荐安装文件
  sudo vim /etc/yum.repos.d/mongodb-org-4.2.repo

  # 文件内容为
  [mongodb-org-4.2]
  name=MongoDB Repository
  baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
  gpgcheck=1
  enabled=1
  gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc

  # 安装
  sudo yum install -y mongodb-org

  # 验证安装结果
  rpm -qa |grep mongodb

  # 输出结果：
  # mongodb-org-mongos-4.2.1-1.el7.x86_64
  # mongodb-org-tools-4.2.1-1.el7.x86_64
  # mongodb-org-shell-4.2.1-1.el7.x86_64
  # mongodb-org-server-4.2.1-1.el7.x86_64
  # mongodb-org-4.2.1-1.el7.x86_64


  # 验证安装结果
  rpm -ql mongodb-org-server

  # 输出结果：
  # /etc/mongod.conf  # 配置文件
  # /lib/systemd/system/mongod.service
  # /usr/bin/mongod
  # /usr/share/doc/mongodb-org-server-4.2.1
  # /usr/share/doc/mongodb-org-server-4.2.1/LICENSE-Community.txt
  # /usr/share/doc/mongodb-org-server-4.2.1/MPL-2
  # /usr/share/doc/mongodb-org-server-4.2.1/README
  # /usr/share/doc/mongodb-org-server-4.2.1/THIRD-PARTY-NOTICES
  # /usr/share/man/man1/mongod.1
  # /var/lib/mongo
  # /var/log/mongodb
  # /var/log/mongodb/mongod.log  # log文件
  # /var/run/mongodb

  # 加入开机启动项
  sudo chkconfig mongod on

  # 启动mongodb
  sudo systemctl start mongod.service

  # 停止mongodb
  sudo systemctl stop mongod.service

  # 重启mongodb
  sudo systemctl restart mongod.service

```


## 5. 安装redis

[官方教程](https://redis.io/topics/quickstart)

```bash
  # 进入安装目录
  cd /usr/local/

  # 下载redis安装包
  sudo wget http://download.redis.io/releases/redis-5.0.5.tar.gz

  # 解压
  sudo tar -zxvf redis-5.0.5.tar.gz

  # 安装构建依赖
  sudo yum install gcc -y

  # 进入redis-5.0.5目录构建
  cd redis-5.0.5/
  sudo make MALLOC=libc

  # 进入src目录构建
  cd src
  sudo make install

  # 返回redis-5.0.5目录修改配置文件
  sudo vim redis.conf

  # [配置文件] 开启守护进程 搜索daemonize no 改为yes
  daemonize yes

  # [配置文件] 修改pidfile文件路径 搜索pidfile 改为
  pidfile /var/run/redis_6379.pid

  # [配置文件] 修改logfile 
  logfile /var/log/redis_6379.log

  # 保存配置文件

  # 新建必要的几个文件夹
  sudo mkdir /etc/redis
  sudo mkdir /var/redis
  sudo mkdir /var/redis/6379

  # 把配置文件复制到/etc/redis文件夹中并命名为6379.conf
  sudo /usr/local/redis-5.0.5/redis.conf /etc/redis/6379.conf

  # 把初始化文件复制到/etc/init.d文件夹中并命名为redis_6379
  sudo cp /usr/local/redis-5.0.5/utils/redis_init_script /etc/init.d/redis_6379

  # 加入开机启动项
  sudo chkconfig --add redis_6379
  sudo chkconfig redis_6379 on

  # 启动redis
  sudo service redisd start

  # 停止redis
  sudo service redisd stop

```

# 6. 部署easy-mock

[官方GitHub仓库](https://github.com/easy-mock/easy-mock)

```bash
  
  # 新建网页目录
  sudo mkdir /usr/local/website/mock

  # 进入相应目录，拉取代码
  cd /usr/local/website/mock/
  sudo git clone https://github.com/easy-mock/easy-mock.git

  # 进入easy-mock目录
  cd easy-mock

  # 切换到master分支
  sudo git checkout master

  # 安装依赖
  sudo npm install

  # 添加生产环节配置文件（把默认配置文件复制一份就行，不需要修改，也可以修改以符合自己的需求）
  cd config/
  sudo cp default.json production.json

  # 构建生产环境资源（回到easy-mock）目录下
  cd ..
  sudo npm run build

  # 运行easy-mock服务
  sudo NODE_ENV=production pm2 start app.js --name easy-mock

```

**通过以上步骤，我们就完成了easy-mock的本地化部署，此时访问ip:7300就可以使用服务啦**