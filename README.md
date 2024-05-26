# Ubuntu 编译 AzerothCore
> 虚拟机或者实体机安装ubuntu-22.04.2
> 远程ssh连接至ubuntu
> 安装1pan或者宝塔面板(推荐宝塔）
> 进入网站 https://www.azerothcore.org/wiki/linux-requirements 安装数据库依赖 Ubuntu with MySQL 8.x
```
sudo apt-get update && sudo apt-get install git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev
```


在用户目录下 创建一个目录输入 mkdir dev 用于存放代码

```
mkdir dev
```
进入 cd dev  后拉取编译文件
```
cd dev
```
```
git clone https://github.com/azerothcore/azerothcore-wotlk.git
```
安装MOD插件

进入代码的 modules 目录
```
cd ~/dev/azerothcore-wotlk/modules
```
克隆 mod 代码，以 mod-eluna 为例，其他 mod 也是类似的操作
```
git clone https://github.com/azerothcore/mod-eluna.git
```
完成后进入 azerothcore-wotlk
```
cd dev/azerothcore-wotlk/
```
创建  mkdir build 进入
```
mkdir build
```
```
cd build
```
build目录下输入 
```
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/azeroth-server/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=all -DSCRIPTS=static -DMODULES=static
```
开始编译 
先检查服务器 核心数 make -j 4  （4代表核心数）
```
nproc --all
```
```
make -j 4 
```
编译完成后输入make install
```
make install
```
进入cd etc目录
```
cd /home/wp/azeroth-server/etc
```
复制文件
```
cp authserver.conf.dist authserver.conf
cp worldserver.conf.dist worldserver.conf
```
下载 data 文件 网址
```
https://github.com/wowgaming/client-data/releases/
```
进入 azeroth-server
```
cd /home/wp/azeroth-server
```
进入后创建data目录
```
mkdir data
```
进入data目录
```
cd /home/wp/azeroth-server/data
```
面板下进入该目录上传data包

给包权限(没有权限的情况下操作) 如有权限可以跳过
```
sudo chown wp:wp data.zip
```
解压该包
```
unzip data.zip
```
无法解压 的情况下安装 如可以解压可以跳过
```
sudo apt-get install unzip
```
解压后面板下删除 data 包

进入etc目录
```
cd /home/wp/azeroth-server/etc
```
worldserver.conf 下修改DataDir = "."
为你的data目录  在宝塔下操作更方便
如DataDir = "/home/wp/azeroth-server/data" 修改命令
```
sudo vi worldserver.conf
```
执行 sudo mysql 进入数据库的终端
```
sudo mysql
```
执行以下SQL语句，创建acore用户，创建acore_world、acore_characters、acore_auth三个数据库，并授权acore用户拥有这三个数据库的所有权限
```
DROP USER IF EXISTS 'acore'@'%';
CREATE USER 'acore'@'%' IDENTIFIED BY 'acore' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0;

GRANT ALL PRIVILEGES ON * . * TO 'acore'@'%' WITH GRANT OPTION;

DROP DATABASE IF EXISTS `acore_world`;
CREATE DATABASE `acore_world` DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_general_ci;

DROP DATABASE IF EXISTS `acore_characters`;
CREATE DATABASE `acore_characters` DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_general_ci;

DROP DATABASE IF EXISTS `acore_auth`;
CREATE DATABASE `acore_auth` DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_general_ci;

GRANT ALL PRIVILEGES ON `acore_world` . * TO 'acore'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `acore_characters` . * TO 'acore'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `acore_auth` . * TO 'acore'@'%' WITH GRANT OPTION;
```
Mysql下查看是否创建成功
```
show databases;
```
Root权限进入 sudo vi /etc/mysql/mysql.conf.d 或者 使用宝塔文件浏览器，进入 /etc/mysql/mysql.conf.d 目录修改 mysqld.cnf 文件，把 
按i开始进行编辑
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
修改后按ESC 后输入 :wq 退出

打开Ubuntu 防火墙打开3306，3724，8085端口
```
sudo ufw allow 3306
sudo ufw allow 3724
sudo ufw allow 8085
```

在终端重启MySQL服务
```
sudo systemctl restart mysql
```
打开 heidisql 、进行服务器连接

以下是启动服务器  进入
```
cd azeroth-server/bin/

```
启动 authserver worldserver 两个窗口分别运行
```
./authserver
```
```
./worldserver
```
worldserver 窗口下创建账户并提升权限
AC下输入
```
account create  账户 密码
```
```
account set gmlevel 帐号 3 -1
```
以上启动方法启动后关闭窗口后台运行服务器

服务器启动需要在以下目录运行
```
cd azeroth-server/bin/
```
第一种
# 1. 以 nohup 方式启动，标准输出和错误输出都重定向到 /dev/null
# 2. 启动前检查是否有 Auth.log/DBErrors.log/Server.log，如果有先备份，格式为 log/YYMMDD_HHMMSS/原文件名.log
```
function backupLog() {
	# 把log/YYMMDD_HHMMSS目录名存到一个变量里
	dirName=log/`date +%y%m%d_%H%M%S`
	# 判断 dirName 目录是否存在，不存在则创建
	if [ ! -d $dirName ]; then
		mkdir -p $dirName
	fi
	# 判断 $1.log 是否存在，存在则备份
	if [ -f $1.log ]; then
		mv $1.log $dirName/$1.log
	fi
}
backupLog Auth
backupLog DBErrors
backupLog Server
```
# 启动服务器
```
function launch() {
	nohup ./$1 > /dev/null 2>&1 &
}
launch authserver
launch worldserver
```

快速启动方法  创建启动文本  复制以下内容到文件中
```
cd azeroth-server/bin/
```
```
vim y.sh
```
```
function launch() {
	nohup ./$1 > /dev/null 2>&1 &
}
launch authserver
launch worldserver
```
创建关闭文本  复制以下内容到文件中
```
vim n.sh
```
```
pkill authserver
pkill worldserver
```
给文件添加执行权限
```
chmod +x *.sh
```
以下是快速启动项目
启动服务器进入
```
cd azeroth-server/bin/
```
```
./y.sh
```
关闭服务器
```
./n.sh
```






第二种

# 安装 screen
sudo apt install screen

# 创建 authserver screen 会话
screen -S authserver

# 启动 authserver
cd ~/azeroth-server/bin
./authserver

# 按 Ctrl+A Ctrl+D 退出 screen 会话

# 创建 worldserver screen 会话
screen -S worldserver

# 启动 worldserver
cd ~/azeroth-server/bin
./worldserver

# 按 Ctrl+A Ctrl+D 退出 screen 会话

# 查看 screen 会话
screen -ls

# 进入 screen 会话
screen -r authserver
screen -r worldserver   下  accunt create 用户 密码




服务端配置文件
worldserver.conf
https://github.com/najoast/acore_doc/blob/master/doc/worldserver.conf.md 修改文档



