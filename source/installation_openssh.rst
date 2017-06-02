***************************
CentOS6.6安装Openssh 7.3p1
***************************

前置条件
===============
1. 操作系统：CentOS6.6 mini安装版

#. Openssh版本 *7.3p1*
#. 安装所需组件

- Zlib 1.1.4或者1.2.1.2或者更高（系统默认安装）
- Openssl >= 0.9.8f < **1.1.0**
- Gcc,Make

    .. code-block:: shell

        yum install -y gcc* make


安装步骤
===============
1. 上传或者下载tar包至 ``tmp/`` 下

2. 解压

.. code-block:: shell

    cd /tmp
    tar -zxf openssh-7.3p1.tar.gz

进入openssh-7.3p1文件夹

.. code-block:: shell

    cd openssh-7.3p1

3. 编译安装

默认安装命令:

.. code-block:: shell

    ./configure
    make
    make install

在安装过程中 ``./configure`` 可能会出现错误，点击查看 :ref:`faq`

将Openssh二进制文件安装到 ``/usr/local/bin`` ,配置文件在 ``/usr/local/etc`` ,server在
``/usr/local/sbin`` 等,使用 ``--prefix`` 来定义不同的安装路径前缀，如下：

.. code-block:: shell

    ./configure --prefix=/opt
    make
    make install

以上命令，将安装到 ``/opt/{bin,etc,lib,sbin}`` .你也可以指定个别的路径，例如：

.. code-block:: shell

    ./configure --prefix=/opt --sysconfdir=/etc/ssh
    make
    make install

以上命令，将安装到 ``/opt/{bin,etc,lib,sbin}`` .但是配置文件在 ``/etc/ssh`` .

4. 复制sshd文件到 ``/etc/init.d``

.. code-block:: shell

    cp /tmp/openssh-7.3p1/contrib/redhat/sshd.init /etc/init.d/sshd

5. 修改启动文件

.. code-block:: shell

    vi /etc/init.d/sshd

修改成自定义安装路径::

    SSHD=/opt/sbin/sshd

    if [ -x /sbin/restorecon ]; then
            /sbin/restorecon /opt/etc/ssh_host_key.pub
            /sbin/restorecon /opt/etc/ssh_host_rsa_key.pub
            /sbin/restorecon /opt/etc/ssh_host_dsa_key.pub
            /sbin/restorecon /opt/etc/ssh_host_ecdsa_key.pub
    fi

注释这句 ``/sbin/restorecon /opt/etc/ssh_host_key.pub``

6. 添加快捷方式

.. code-block:: shell

    cd /usr/bin
    ln -s -T /opt/bin/scp scp
    ln -s -T /opt/bin/sftp sftp
    ln -s -T /opt/bin/ssh ssh
    ln -s -T /opt/bin/ssh-add ssh-add
    ln -s -T /opt/bin/ssh-agent ssh-agent
    ln -s -T /opt/bin/ssh-keygen ssh-keygen
    ln -s -T /opt/bin/ssh-keyscan ssh-keyscan

7. 启动sshd服务

.. code-block:: shell

    service sshd start

8. 增加服务到启动项

.. code-block:: shell

    chkconfig --add sshd
    chkconfig sshd on

配置Openssh
================
运行配置文件被存放在 ``${prefix}/etc`` 或者你指定的 ``--sysconfdir`` .（默认在 ``/usr/local/etc`` ）

.. _faq:

FAQ
================
1.configure报错
****************
报错
----------------
::

    configure: error: *** OpenSSL headers missing - please install first or check config.log ***

解决办法
----------------
安装openssl-devel

2.所有用户不能登录
**********************
原因
----------------
可能是因为受到了 ``/etc/pam.d/`` 下模块的影响

解决办法
----------------
配置文件 ``sshd_config`` 启用``UsePAM yes``

``./configure`` 的时候需要带上 ``--with-pam`` 参数。否则配置文件 ``sshd_config`` 中配置 ``UsePAM yes`` 会报错！

3.安装后root不能登录
**********************
原因
----------------
默认禁止root登入

解决办法
----------------
修改

.. code-block:: shell

    vi ${prefix}/etc/sshd_config

将  ``PermitRootLogin prohibit-password`` 改为 ``PermitRootLogin YES``
