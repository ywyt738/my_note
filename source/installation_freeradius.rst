**********************************
CentOS 7安装 freeraidus
**********************************

环境
================
- CentOS 7.0 x86_64
- FreeRadius 3.x

安装
================

1. 安装 freeradius,freeradius-mysql
************************************************
::

  yum install -y freeradius
  yum install -y freeradius-mysql

2. 配置 freeraidus 使用mysql认证
************************************************
::

  cd /etc/raddb/mods-enabled
  ln -s ../mods-available/sql

修改 ``/etc/raddb/mods-available/sql`` 文件

找到 ``driver = "rlm_sql_null"`` 修改为 ``driver = "rlm_sql_mysql"``

找到 ``dialect = "sqlite"`` 修改为 ``dialect = "mysql"``

保存并退出。

``Connection info`` 下的为数据库信息，去掉 ``#``，修改信息即可指定数据库。

#. 配置认证客户端
在 ``/etc/raddb/clients.conf`` 最后增加一下内容：
::

  client users {
          ipaddr = *
          secret = testing123
  }

ipadd = *
  允许所有IP客户端进行认证

secret = testing123
  密钥

3. 防火墙开放端口
************************************************
开放udp1812和1813端口::

  firewall-cmd --zone=public --add-port=1812/udp --permanent
  firewall-cmd --zone=public --add-port=1813/udp --permanent
  firewall-cmd --reload  #重新载入防火墙

防火墙常用命令:
::

  # 添加
  firewall-cmd --zone=public --add-port=80/tcp --permanent        #（--permanent永久生效，没有此参数重启后失效）
  # 重新载入
  firewall-cmd --reload
  # 查看
  firewall-cmd --zone= public --query-port=80/tcp
  # 删除
  firewall-cmd --zone= public --remove-port=80/tcp --permanent

4. 创建数据库
************************************************
登录数据库服务器::

  mysql -uroot -p
  Enter password:******
  mysql>create database radius;
  mysql>grant all on radius.* to radius@localhost identified by "radpass";
  mysql>exit;

5. 导入表结构
************************************************
在radius服务器上执行以下命令，或者复制 ``/etc/raddb/mods-config/sql/main/mysql/scheama.sql`` 文件到数据库服务器执行。

radius服务器上执行:
::

  mysql -u root -h <数据库IP> radius < /etc/raddb/mods-config/sql/main/mysql/scheama.sql

数据库服务器上执行:
::

  mysql -u root radius <存放路径>/scheama.sql

6. 表结构简要说明
************************************************
==================== ====================
表名                  作用
==================== ====================
radcheck             用户检查信息表
radreply             用户回复信息表
radgroupcheck        用户组检查信息表
radgroupreply        用户组检查信息表
radusergroup         用户和组关系表
radacct              计费情况表
radpostauth          认证后处理信息，可以包括认证请求成功和拒绝的记录。
==================== ====================
