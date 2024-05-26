###虚拟机或者实体机安装ubuntu-22.04.2

###远程ssh连接至ubuntu

###安装1pan或者宝塔面板(推荐宝塔）

###进入网站https://www.azerothcore.org/wiki/linux-requirements 安装数据库依赖 Ubuntu with MySQL 8.x   安装时间预计和你的网速相关.

###点击 Step 2: Core Installation 进入下一步

###在用户目录下 创建一个目录输入 mkdir dev 用于存放代码

##进入 cd dev  后拉取编译文件 

git clone https://github.com/azerothcore/azerothcore-wotlk.git

7.开始编译代码  完成后进入 cd azerothcore
cd azerothcore
创建  mkdir build 进入
cd build
输入 
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/azeroth-server/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=all -DSCRIPTS=static -DMODULES=static

9.开始编译 
检查服务器 核心数 nproc –all



make -j 4       4表核心数  编译时间较长

10. 编译完成后输入make install

11.下载  data 网址https://github.com/wowgaming/client-data/releases/
12.cd /home/wp/azeroth-server/etc
13.进入cd etc目录进行如下操作复制文件

cp authserver.conf.dist authserver.conf
cp worldserver.conf.dist worldserver.conf

14. cd /home/wp/azeroth-server
进入后创建data目录
mkdir data

15.    cd /home/wp/azeroth-server/data
面板下进入该目录上传data包

16. sudo chown wp:wp data.zip 给包权限(没有权限的情况下操作)
解压该包unzip data.zip 无法解压 
安装sudo apt-get install unzip
解压后面板下删除 data包

17.进入cd /home/wp/azeroth-server/etc目录 修改 authserver.conf 可以修改数据库账户和密码
worldserver.conf 下修改DataDir = "."
为你的data目录
如DataDir = "/home/wp/azeroth-server/data"

18. 执行 sudo mysql 进入数据库的终端
执行以下SQL语句，创建acore用户，创建acore_world、acore_characters、acore_auth三个数据库，并授权acore用户拥有这三个数据库的所有权限
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

查看是否创建成功
Mysql> show databases;

Root权限进入 sudo vi /etc/mysql/mysql.conf.d 或者 使用宝塔文件浏览器，进入 /etc/mysql/mysql.conf.d 目录修改 mysqld.cnf 文件，把 
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
修改后按ESC 后输入 :wq 退出

打开Ubuntu 防火墙打开3306端口
sudo ufw allow 3306
sudo ufw allow 3724
sudo ufw allow 8085


在终端输入 
sudo systemctl restart mysql 重启MySQL服务

打开 heidisql 、进行服务器连接

以下是启动服务器
第一种
# 1. 以 nohup 方式启动，标准输出和错误输出都重定向到 /dev/null
# 2. 启动前检查是否有 Auth.log/DBErrors.log/Server.log，如果有先备份，格式为 log/YYMMDD_HHMMSS/原文件名.log
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

# 启动服务器
function launch() {
	nohup ./$1 > /dev/null 2>&1 &
}
launch authserver
launch worldserver


快速启动方法
vim statr.sh  #创建启动文本  复制以下内容到文件中
function launch() {
	nohup ./$1 > /dev/null 2>&1 &
}
launch authserver
launch worldserver

vim stop.sh   #创建关闭文本  复制以下内容到文件中
pkill authserver
pkill worldserver

chmod +x *.sh  #给文件添加执行权限

./start.sh   #启动
./stop.sh  #关闭






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


MOD插件编译
https://github.com/najoast/acore_doc/blob/master/doc/linux_compile_mod.md


# 1. 进入代码的 modules 目录
cd ~/dev/azerothcore-wotlk/modules

# 2. 克隆 mod 代码，以 mod-eluna 为例，其他 mod 也是类似的操作
git clone https://github.com/azerothcore/mod-eluna.git

# 3. 进入 build 目录
cd ../build

# 4. 重新 cmake 生成 Makefile
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/azeroth-server/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=all -DSCRIPTS=static -DMODULES=static

# 5. 编译
make -j 4     (nproc --all)

# 6. 安装
make install





