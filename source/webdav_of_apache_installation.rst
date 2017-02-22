=================
WebDAV安装
=================

操作步骤
=================
1. 安装apache

::

  #安装apache
  $ yum -y install httpd apr apr-util httpd-deve

  #设置开机启动(可选)
  $ chkconfig httpd on

  #启动apache服务
  $ service httpd start

  #临时关闭防火墙(如果不关闭，则只能本机访问apache服务)
  $ service iptables stop

2. 配置WebDAV模块
apache服务器已经集成了WebDAV模块，所以只需要启用该模块即可。

  2.1 编辑/etc/httpd/conf/httpd.conf

  ::

    $ vi /etc/httpd/conf/httpd.conf

  在文件的最后添加以下语句：

  ::

    Include conf/webdav.conf #指定webdav的配置文件路径

  2.2 新建webdav配置文件

  ::

    $ vi /etc/httpd/conf/webdav.conf

  将以下内容插入到文件中：

  ::

    <IfModule mod_dav.c>
    LimitXMLRequestBody 131072
    Alias /webdav "/var/www/webdav"
    <Directory /var/www/webdav>
    Dav On
    Options +Indexes
    IndexOptions FancyIndexing
    AddDefaultCharset UTF-8
    AuthType Basic
    AuthName "WebDAV Server"
    AuthUserFile /etc/httpd/webdav.users.pwd
    Require valid-user
    Order allow,deny
    Allow from all
    </Directory>
    </IfModule>

  2.3. 创建访问目录

  ::

    $ mkdir -p /var/www/webdav
    $ chown apache:apache /var/www/webdav

  2.4. 添加用户

  ::

    $ htpasswd -c /etc/httpd/webdav.users.pwd test #根据提示输入密码

  2.5. 重启apache服务

  ::

    $ service httpd restart

3. 测试
