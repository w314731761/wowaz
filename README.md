# Ubuntu 编译 AzerothCore
> 虚拟机或者实体机安装ubuntu-22.04.2
> 远程ssh连接至ubuntu
> 安装1pan或者宝塔面板(推荐宝塔）
> 相关编译说明可以进入网站 https://www.azerothcore.org/wiki/linux-requirements 

安装数据库依赖 Ubuntu with MySQL 8.x  以下为代码
```
sudo apt-get update && sudo apt-get install git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev
```
在用户目录下 创建一个目录输入 mkdir dev 用于存放代码

```
mkdir dev
```
进入  dev  后拉取编译文件
```
cd dev
```
```
git clone https://github.com/azerothcore/azerothcore-wotlk.git
```
先安装MOD插件

进入 modules 目录
```
cd ~/dev/azerothcore-wotlk/modules
```
克隆 mod 代码，按需使用，编译的时候将会一起编译进去。
```
git clone https://github.com/azerothcore/mod-eluna.git  #lua脚本模块
git clone https://github.com/azerothcore/mod-anticheat.git  #防作弊模块
git clone https://github.com/azerothcore/mod-learn-spells.git  #升级后自动学习法术,无需访问训练师
git clone https://github.com/azerothcore/mod-aoe-loot.git  #AOE一键拾取
git clone https://github.com/azerothcore/mod-who-logged.git  #玩家登录提示
git clone https://github.com/azerothcore/mod-ip-tracker.git  #ip 跟踪器
git clone https://github.com/azerothcore/mod-npc-buffer.git  #buff大师
git clone https://github.com/azerothcore/mod-npc-beastmaster.git  #猎人宠物大师
git clone https://github.com/azerothcore/mod-multi-client-check.git  #多客户端检查 
git clone https://github.com/ZhengPeiRu21/mod-challenge-modes.git  #一命通关
git clone https://github.com/azerothcore/mod-chat-transmitter  #mod-chat聊天发送器
```
完成后进入 azerothcore-wotlk
```
cd ~/dev/azerothcore-wotlk/
```
创建 build 并进入build
```
mkdir build
```
```
cd build
```
build目录下输入配置 CMake 以构建项目编译信息
```
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/azeroth-server/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=all -DSCRIPTS=static -DMODULES=static
```
开始编译前
先检查服务器 核心数 make -j 4  （4代表核心数）
```
nproc --all
```
开始进行编译
```
cd ~/dev/azerothcore-wotlk/build/  #在该目录下进行编译
```
```
make -j 4   #开始编译
```
编译完成后输入make install进行安装
```
make install
```
进入cd etc目录
```
cd ~/azeroth-server/etc
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
cd ~/azeroth-server/
```
进入后创建data目录
```
mkdir data
```
进入data目录
```
cd ~/azeroth-server/data
```
面板下进入该目录上传data包

给包权限(没有权限的情况下操作) 如有权限可以跳过
```
sudo chown wp:wp data.zip  #wp为你的用户权限
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
cd ~/azeroth-server/etc
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
Root权限进入 
```
sudo vi /etc/mysql/mysql.conf.d
```
或者使用宝塔文件浏览器，进入 /etc/mysql/mysql.conf.d 目录修改 mysqld.cnf 文件。

按i开始进行编辑以下项目
```
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```
修改后按ESC 后输入 :wq 退出

防火墙打开3306，3724，8085端口
```
sudo ufw allow 3306
sudo ufw allow 3724
sudo ufw allow 8085
```

重启MySQL服务
```
sudo systemctl restart mysql
```
以下是启动服务器  进入
```
cd ~/azeroth-server/bin/
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
cd ~/azeroth-server/bin/
```
#第一种
以 nohup 方式启动
```
function backupLog() {
	dirName=log/`date +%y%m%d_%H%M%S`
	if [ ! -d $dirName ]; then
		mkdir -p $dirName
	fi
	if [ -f $1.log ]; then
		mv $1.log $dirName/$1.log
	fi
}
backupLog Auth
backupLog DBErrors
backupLog Server
```
启动服务器
```
function launch() {
	nohup ./$1 > /dev/null 2>&1 &
}
launch authserver
launch worldserver
```

快速启动方法
进入bin目录
```
cd ~/azeroth-server/bin/
```
创建启动文本
```
vim start.sh  #可以自定义名称
```
按i复制以下内容到文件中
```
function launch() {
	nohup ./$1 > /dev/null 2>&1 &
}
launch authserver
launch worldserver
```
创建关闭文本
```
vim stop.sh  #可以自定义名称
```
复制以下内容到文件中
```
pkill authserver
pkill worldserver
```
给文件添加执行权限
```
chmod +x *.sh
```
以下是快速启动项目
进入以下目录
```
cd azeroth-server/bin/
```
启动服务器
```
./start.sh  
```
关闭服务器
```
./stop.sh
```
以上这种快速启动方法启动后无法进入AC后台所以需要优先创建GM账号进行之后的操作

#第二种启动服务器方法
安装 screen
```
sudo apt install screen
```
创建 authserver screen 会话
```
screen -S authserver
```
启动 authserver
```
cd ~/azeroth-server/bin
./authserver
```
按 Ctrl+A Ctrl+D 退出 screen 会话
创建 worldserver screen 会话
```
screen -S worldserver
```
启动 worldserver
```
cd ~/azeroth-server/bin
./worldserver
```
按 Ctrl+A Ctrl+D 退出 screen 会话
查看 screen 会话
```
screen -ls
```
进入 screen 会话
```
screen -r authserver
```
```
screen -r worldserver
```
#以上任意一个方法启动服务器后可以启动SQL管理工具如heidisql下输入你的ip 账户密码 端口3306进入acore_auth下的realmlist修改以下内容
name为你的服务器名称
address为你的服务器ip或者域名

#服务端配置文件home/wp/azeroth-server/etc/目录下面修改
worldserver.conf
修改服务器相关参数

中英对照表
https://github.com/najoast/acore_doc/blob/master/doc/worldserver.conf.md
感谢najoast大神的翻译

/azeroth-server/bin/lua_scripts为lua脚本存放处 
eluna重启命令 游戏内使用
```
.reload eluna
```



