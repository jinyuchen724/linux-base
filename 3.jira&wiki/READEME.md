| 版本 | 日期 | 状态 | 修订人 | 摘要 |
| - | :-: | :-: | :-: | :-: |
| V1.0 | 2018-05-20 | 初次编写 | 开源 | 初次编写 |

# 基础环境安装

- 安装Java环境

JIRA和WIKI的运行需要依赖Java环境，安装完后执行<kbd>java -version</kbd>命令检查JDK是否安装成功。如下输出信息：

```
[root@ali02-prod-ops-jirawiki-01 ~]# java -version
java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)
```
> 本环境中的JDK均是使用脚本进行安装，安装路径均在/xinguang，以下是脚本内容:

```
#!/bin/bash

SRC_URI="https://tol-res.oss-cn-shanghai-internal.aliyuncs.com/java/jdk-8u65-linux-x64.tar.gz"
PKG_NAME=`basename $SRC_URI`
DIR=`pwd`
DATE=`date +%Y%m%d%H%M%S`

\mv /xinguang/java /xinguang/java.bak.$DATE &> /dev/null
mkdir -p /xinguang/java
mkdir -p /xinguang/install
cd /xinguang/install

if [ ! -s $PKG_NAME ]; then
  wget -c $SRC_URI
fi

mv jdk1.8.0_65 jdk1.8.0_65_bak.$DATE &> /dev/null

tar zxvf $PKG_NAME
mv jdk1.8.0_65/*  /xinguang/java
rm -rf jdk1.8.0_65
#add PATH
if ! cat /etc/profile | grep 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH' &> /dev/null;then
	echo "export JAVA_HOME=/xinguang/java" >> /etc/profile
	echo "export JRE_HOME=/xinguang/java/jre" >> /etc/profile
	echo 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH' >> /etc/profile
	echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile
fi

cd $DIR
source /etc/profile
bash
```

- 安装MySQL数据库

先检查本机上是否安装其他版本的数据库

```
[root@ali02-prod-ops-jirawiki-01 ~]# rpm -qa | grep -i mariadb                           //查看有没有安装mariadb 
[root@ali02-prod-ops-jirawiki-01 ~]# rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64    //如果有，卸载MariaDB 
[root@ali02-prod-ops-jirawiki-01 ~]# rpm -qa | grep -i mysql                   //查看有没有安装mysql,若有卸载旧版本的Mysql
[root@ali02-prod-ops-jirawiki-01 ~]# rm -rf /usr/lib64/mysql            //删除mysql分散的文件夹
[root@ali02-prod-ops-jirawiki-01 ~]# yum -y install autoconf
```

下载MySQL5.6并安装

```
[root@ali02-prod-ops-jirawiki-01 ~]# wget https://cdn.mysql.com//Downloads/MySQL-5.6/MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar
[root@ali02-prod-ops-jirawiki-01 ~]# mkdir mysql && tar -xf MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar -C mysql
[root@ali02-prod-ops-jirawiki-01 mysql]# rpm -ivh MySQL-client-5.6.40-1.el7.x86_64.rpm
[root@ali02-prod-ops-jirawiki-01 mysql]# rpm -ivh MySQL-devel-5.6.40-1.el7.x86_64.rpm
[root@ali02-prod-ops-jirawiki-01 mysql]# rpm -ivh MySQL-server-5.6.40-1.el7.x86_64.rpm 
[root@ali02-prod-ops-jirawiki-01 ~]# systemctl start mysql && chkconfig mysql on
[root@ali02-prod-ops-jirawiki-01 ~]# cat /root/.mysql_secret    //查看数据库的root密码
[root@ali02-prod-ops-jirawiki-01 ~]# mysql -uroot -p
Enter password: 
mysql> SET PASSWORD = PASSWORD('xg@2018');
mysql> exit
```
> JIRA和WIKI的安装包及破解包下载地址： https://pan.baidu.com/s/1cxDTGJB_H0Rn0CuJTfq0-A


# 二、安装JIRA 7.3.6版本

- 登录MySQL控制台，执行如下命令，创建JIRA数据库

```
mysql> CREATE DATABASE jira DEFAULT CHARACTER SET utf8;
mysql> GRANT ALL ON jira.* TO 'jira' @'%' IDENTIFIED BY 'jira#2018' WITH GRANT OPTION;
mysql> GRANT ALL ON jira.* TO 'jira'@localhost IDENTIFIED BY 'jira#2018' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

- 在my.cnf文件中配置数据库字符集并设置连接数

```
[root@ali02-prod-ops-jirawiki-01 ~]# vim /usr/my.cnf
[mysqld]
character_set_server=utf8
max_connections=500
binlog_format=mixed
max_allowed_packet=34M
innodb_log_file_size=256M
[root@ali02-prod-ops-jirawiki-01 ~]# systemctl restart mysql
```

- 使用FTP工具上传atlassian-jira-software-7.3.6.zip文件包至/xinguang/install目录并解压出来

```
[root@ali02-prod-ops-jirawiki-01 install]# unzip atlassian-jira-software-7.3.6.zip 
[root@ali02-prod-ops-jirawiki-01 install]# ls
atlassian-jira-software-7.3.6-x64.bin   atlassian-jira-7.3.6-patch

```

- 解压后给安装可执行权限，然后进行交互式安装，如下：

```
[root@ali02-prod-ops-jirawiki-01 install]# chmod +x atlassian-jira-software-7.3.6-x64.bin
[root@ali02-prod-ops-jirawiki-01 install]# ./atlassian-jira-software-7.3.6-x64.bin 
Unpacking JRE ...
Starting Installer ...
May 20, 2018 5:27:03 PM java.util.prefs.FileSystemPreferences$1 run
INFO: Created user preferences directory.
May 20, 2018 5:27:03 PM java.util.prefs.FileSystemPreferences$2 run
INFO: Created system preferences directory in java.home.

This will install JIRA Software 7.3.6 on your computer.
OK [o, Enter], Cancel [c]
o
Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (use default settings) [1], Custom Install (recommended for advanced users) [2, Enter], Upgrade an existing JIRA installation [3]
1
Details on where JIRA Software will be installed and the settings that will be used.
Installation Directory: /opt/atlassian/jira 
Home Directory: /var/atlassian/application-data/jira 
HTTP Port: 8080 
RMI Port: 8005 
Install as service: Yes 
Install [i, Enter], Exit [e]
i

Extracting files ...
                                                                           

Please wait a few moments while JIRA Software is configured.
Installation of JIRA Software 7.3.6 is complete
Start JIRA Software 7.3.6 now?
Yes [y, Enter], No [n]
y

Please wait a few moments while JIRA Software starts up.
Launching JIRA Software ...
Installation of JIRA Software 7.3.6 is complete
Your installation of JIRA Software 7.3.6 is now ready and can be accessed
via your browser.
JIRA Software 7.3.6 can be accessed at http://localhost:8080
Finishing installation ...

[root@ali02-prod-ops-jirawiki-01 install]# /etc/init.d/jira stop
```

- 安装完成后，暂时不启动JIRA，将atlassian-jira-7.3.6-patch里面的破解包拷贝到指定路径，然后再启动JIRA服务

```
[root@ali02-prod-ops-jirawiki-01 install]# cd atlassian-jira-7.3.6-patch/
[root@ali02-prod-ops-jirawiki-01 atlassian-jira-7.3.6-patch]# ls
atlassian-extras-3.2.jar  atlassian-universal-plugin-manager-plugin-2.17.13.jar  mysql-connector-java-5.1.39-bin.jar
[root@ali02-prod-ops-jirawiki-01 atlassian-jira-7.3.6-patch]# cp atlassian-extras-3.2.jar mysql-connector-java-5.1.39-bin.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
cp: overwrite ‘/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/atlassian-extras-3.2.jar’? y
[root@ali02-prod-ops-jirawiki-01 atlassian-jira-7.3.6-patch]# cp atlassian-universal-plugin-manager-plugin-2.17.13.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/
[root@ali02-prod-ops-jirawiki-01 ~]# /etc/init.d/jira start
```

- 服务启动后打开浏览器输入IP地址+端口访问（JIRA默认端口为8080），访问地址后，系统会自动跳转到JIRA的默认配置页面，在此我们选择自定义配置，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/2cb3c0c9-4db5-4f23-b97d-c18d567847af.png)

- 设置数据库连接信息，在数据库方面，我们选择MySQL数据库即可，填写完毕后相关的数据库地址、用户和密码后，就可以点击下一步按钮，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/a8259344-ca05-45df-b894-a7d13637b77e.png)

- 设置应用程序的属性，然后下一步按钮，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/ecc621a2-4b78-43a7-8e77-df1b61a316b7.png)

- 点击箭头处的链接"生成JIRA试用许可证"，然后跳转到官网申请，需要使用账号登陆（建议使用Google账号登陆）

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/9663c7d0-351c-4c39-8b93-a467e50325fb.png)

- 由于没有正式的License，所以需要账户登陆JIRA，然后利用这个账号申请一个可以试用30天的License，点击"Generate License"后跳转到另一个页面，选择对应的Server ID,在弹出的对话框中选择"Yes"如下：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/bc31c66c-708c-4f9d-871a-4e9ef5bd4c36.png)

- 点击上图弹出的确认框后，生成的key会自动填充到方框内,如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/e4ea0f2e-e0e5-4af4-ada2-f6e503e238fb.png)

- 设置管理员账户

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/057cb142-43fa-444f-8424-4690676231b3.png)

- 提示是否设置电子邮件通知，选择稍后设置

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/c412cacf-71d5-4bef-820c-027b7f48bf8c.png)

到此JIRA 7.3.6软件的安装就基本完成了。

# 三、JIRA电子邮件设置

- 登陆JIRA后，点击右上角的"设置"——"系统"——"电邮（外发邮件）"

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/0a9390a6-d70b-485e-9f9b-fe31a40d4681.png)


# 四、安装Wiki 6.7.3版本

- 登录MySQL控制台，执行如下命令，创建confluence数据库

```
mysql> CREATE DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
mysql> GRANT ALL ON confluence.* TO 'wiki' @'%' IDENTIFIED BY 'wiki#2018' WITH GRANT OPTION;
mysql> GRANT ALL ON confluence.* TO 'wiki' @localhost IDENTIFIED BY 'wiki#2018' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

- 使用FTP工具上传tlassian-confluence-wiki-6.7.3.zip文件包至/xinguang/install目录并解压出来

```
[root@ali02-prod-ops-jirawiki-01 install]# unzip atlassian-confluence-wiki-6.7.3.zip 
[root@ali02-prod-ops-jirawiki-01 install]# ls
atlassian-confluence-6.7.3-x64.bin  atlassian-confluence-wiki-6.7.3-patch
```

- 解压后给安装可执行权限，然后进行交互式安装，如下：

```
[root@ali02-prod-ops-jirawiki-01 install]# chmod +x atlassian-confluence-6.7.3-x64.bin
[root@ali02-prod-ops-jirawiki-01 install]# ./atlassian-confluence-6.7.3-x64.bin 
Unpacking JRE ...
Starting Installer ...
May 20, 2018 10:01:19 PM java.util.prefs.FileSystemPreferences$2 run
INFO: Created system preferences directory in java.home.

This will install Confluence 6.7.3 on your computer.
OK [o, Enter], Cancel [c]
o  
Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (uses default settings) [1], 
Custom Install (recommended for advanced users) [2, Enter], 
Upgrade an existing Confluence installation [3]
1
See where Confluence will be installed and the settings that will be used.
Installation Directory: /opt/atlassian/confluence 
Home Directory: /var/atlassian/application-data/confluence 
HTTP Port: 8090 
RMI Port: 8000 
Install as service: Yes 
Install [i, Enter], Exit [e]
i

Extracting files ...
                                                                           

Please wait a few moments while we configure Confluence.
Installation of Confluence 6.7.3 is complete
Start Confluence now?
Yes [y, Enter], No [n]
y

Please wait a few moments while Confluence starts up.
Launching Confluence ...
Installation of Confluence 6.7.3 is complete
Your installation of Confluence 6.7.3 is now ready and can be accessed via
your browser.
Confluence 6.7.3 can be accessed at http://localhost:8090
Finishing installation ...
```

- 打开浏览器输入IP地址+端口（WIKI默认端口为8090），访问地址后，系统会自动跳转到WIKI的默认配置页面，选择"产品安装"，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/1836b5e1-3d27-4db9-a44f-4a749f3eea12.png)

- 获取confluence扩展插件，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/e917064b-279e-45d5-a7fe-b0fdc9abe334.png)

- 输入授权码页面，先保存"服务器 ID",然后停掉confluence服务，进行破解步骤

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/7e618542-27ec-4811-a45f-d95a61151a2d.png)

```
[root@ali02-prod-ops-jirawiki-01 install]# /etc/init.d/confluence stop

[root@ali02-prod-ops-jirawiki-01 ~]# cp /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.3.0.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.3.0.jar.$(date +%F)    //先将atlassian-extras-decoder-v2-3.3.0.jar做个备份
[root@ali02-prod-ops-jirawiki-01 ~]# mv /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.3.0.jar /root/atlassian-extras-2.4.jar    //将atlassian-extras-decoder-v2-3.3.0.jar重命名为atlassian-extras-2.4.jar
[root@ali02-prod-ops-jirawiki-01 ~]# sz atlassian-extras-2.4.jar     //将atlassian-extras-2.4.jarbao拷贝到Windows
[root@ali02-prod-ops-jirawiki-01 ~]# sz /xinguang/install/atlassian-confluence-wiki-6.7.3-patch/confluence_keygen.jar  //将补丁文件目录的confluence_keygen.jar也拷贝至Windows下
```

- 在Windows下，生成License Key，操作方法如下：

```
c:\Users\XG\Desktop>java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

c:\Users\XG\Desktop>java -jar confluence_keygen.jar
```

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/cb52d616-7ba5-48d4-8978-4ab7fe843d8b.jpg)

- 将新生成的atlassian-extras-2.4.jar上传服务重命名为atlassian-extras-decoder-v2-3.3.0.jar并将破解包拷贝至/opt/atlassian/confluence/confluence/WEB-INF/lib路径下

```
[root@ali02-prod-ops-jirawiki-01 ~]# mv atlassian-extras-2.4.jar atlassian-extras-decoder-v2-3.3.0.jar
[root@ali02-prod-ops-jirawiki-01 ~]# mv atlassian-extras-decoder-v2-3.3.0.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
[root@ali02-prod-ops-jirawiki-01 ~]# cd /xinguang/install/atlassian-confluence-wiki-6.7.3-patch/
[root@ali02-prod-ops-jirawiki-01 atlassian-confluence-wiki-6.7.3-patch]# cp Confluence-6.7.0-language-pack-zh_CN.jar mysql-connector-java-5.1.39-bin.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/

[root@ali02-prod-ops-jirawiki-01 atlassian-confluence-wiki-6.7.3-patch]# /etc/init.d/confluence start    //启动confluence服务
```

- 再次打开浏览器输入IP地址+端口（WIKI默认端口为8090）访问地址，在方框内输入生成的key文件，然后下一步，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/2b918844-11d1-424e-93e2-b96581a9f31b.png)

- 设置数据库，这里选择"我自己的数据库"，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/3996457e-a680-42d9-bff6-9b687179c4fd.png)

- 设置数据库连接信息，选择MySQL数据库即可，填写完毕后相关的数据库地址、用户和密码后，就可以点击下一步按钮，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/1061c17c-1ce4-45d3-98c2-3a8ef4ccf362.png)

- 加载内容，选择"加载"示范站点"",如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/59f22203-4baf-4e0c-8333-5df2753a5ba7.png)

- 配置用户管理，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/e23af0e9-7e4c-4a5f-8d9b-6bf989d5cbd4.png)

- 配置系统管理员账户，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/e69d0693-f928-443f-afdf-8d1a353eb666.png)

- 设置完成,至此Wiki 6.7.3就安装完成了

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/4e273213-73b4-4d13-b580-daf7f581c3d2.png)

# 五、WIKI邮件服务器配置

- 将${HOME}/confluence/confluence/WEB-INF/lib/mail-1.4.5.jar 转移到${HOME}/confluence/lib/目录下

```
[root@ali02-prod-ops-jirawiki-01 ~]# mv /opt/atlassian/confluence/confluence/WEB-INF/lib/mail-1.4.5.jar /opt/atlassian/confluence/lib/
```

- 修改server.xml文件，在</Context>上一行添加邮箱配置

```
[root@ali02-prod-ops-jirawiki-01 ~]# vim /opt/atlassian/confluence/conf/server.xml
 14                     <Resource name="mail/QqSMTPServer"
 15                            auth="Container"
 16                            type="javax.mail.Session"
 17                            mail.smtp.host="smtp.exmail.qq.com"
 18                            mail.smtp.port="465"
 19                            mail.smtp.auth="true"
 20                            mail.smtp.user="sysadmin@xxxxx.com"
 21                            password="**********"
 22                            mail.smtp.starttls.enable="true"
 23                            mail.smtp.socketFactory.class="javax.net.ssl.SSLSocketFactory" />
```

- 登陆WIKI后，点击右上角的"设置"——"一般配置"——"邮件服务器"——"增加新的SMTP邮件服务器"

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/c3b40d21-739c-479d-9f3d-7f23a2031a96.png)

- Confluence自带的邮件服务配置不支持腾讯邮箱，因此我们需要使用JNDI方式:
 
![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/3965340a-b2cf-45e9-aef4-5ebfc60e5474.png)

- 发送测试邮件，如下图：

![](/web_upload/markdown/picture/81eeddbf-5bf1-4d6c-82d7-8f93e55be89d/a094b72c-d1e3-4afa-bbd9-5789854e62b4.png)
