******************************
Google身份验证器部署
******************************

1. 安装
===================

1.1 安装依赖软件包
-------------------
::

    yum -y install gcc make pam-devel libpng-devel libtool wget git

1.2 安装Qrencode，谷歌身份验证器需要调用该程序以便终端生成并显示二维码(需要EPEL)
--------------------------------------------------------------------------------------------
::

    yum -y install qrencode

1.3 安装谷歌身份验证器
-------------------------------------
::

    cd ~
    git clone https://github.com/google/google-authenticator-libpam.git
    cd google-authenticator-libpam/
    ./bootstrap.sh
    ./configure
    make
    sudo make install

1.4 修改/etc/pam.d/sshd第一行后添加谷歌身份验证器PAM模块配置
---------------------------------------------------------------------------------
::

    auth required pam_google_authenticator.so

1.5 修改SSH服务配置/etc/ssh/sshd_config
--------------------------------------------------------------
::

    ChallengeResponseAuthentication yes

1.6 重启SSH服务
------------------------------------
RHEL 6/CentOS
::

    service sshd restart

RHEL 7/CentOS 7
::

    systemctl restart sshd.service

1.7 用户设置
运行google-authenticator-libpam目录下的 ``google-authenticator`` 二进制文件来创建一个
新的密钥，相关的设置会保存在 ``~/.google_authenticator``.
::

    Do you want authentication tokens to be time-based (y/n) y
    # 是否使用基于时间方式生成验证口令，注意时间同步
    Warning: pasting the following URL into your browser exposes the OTP secret to Google:
      https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&...%3Dlocalhost.localdomain
      # 显示二维码图片的地址，需要设法能够访问到谷歌，可以使用翻墙，或者寻找暂未被中国长城防火墙屏蔽的谷歌公网IP地址，然后修改系统hosts文件地址映射。

      [二维码图片]       # 若未安装Qrencode，则不会显示二维码，使用 ``google-authenticator`` 程序扫描二维码。

    Your new secret key is: MRQ73VZWQF7Z4EK3GACBAPEH5M
    Your verification code is 603335
    Your emergency scratch codes are:
      60323738
      69249157
      22062335
      85699230
      35755622
    # 生成的5组应急备用验证码，每个验证码只能使用一次，使用后立即失效。当多次使用手机App端显示的验证码无效时使用，保存备用。
    Do you want me to update your "/root/.google_authenticator" file? (y/n) y
    # 是否更新配置文件

    Do you want to disallow multiple uses of the same authentication
    token? This restricts you to one login about every 30s, but it increases
    your chances to notice or even prevent man-in-the-middle attacks (y/n) y

    By default, a new token is generated every 30 seconds by the mobile app.
    In order to compensate for possible time-skew between the client and the server,
    we allow an extra token before and after the current time. This allows for a
    time skew of up to 30 seconds between authentication server and client. If you
    experience problems with poor time synchronization, you can increase the window
    from its default size of 3 permitted codes (one previous code, the current
    code, the next code) to 17 permitted codes (the 8 previous codes, the current
    code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
    between client and server.
    Do you want to do so? (y/n) y

    If the computer that you are logging into isn't hardened against brute-force
    login attempts, you can enable rate-limiting for the authentication module.
    By default, this limits attackers to no more than 3 login attempts every 30s.
    Do you want to enable rate-limiting? (y/n) y


2. Module options
=============================

secret=/path/to/secret/file
------------------------------------
See :ref:`encrypted home directories`.

authtok_prompt=prompt
------------------------------------
Overrides default token prompt. If you want to include spaces in the prompt, wrap the whole argument in square brackets:
::

    auth required pam_google_authenticator.so [authtok_prompt=Your secret token: ]

user=some-user
------------------------------------
Force the PAM module to switch to a hard-coded user id prior to doing any file operations. Commonly used with secret=.

no_strict_owner
------------------------------------
DANGEROUS OPTION!

By default the PAM module requires that the secrets file must be owned the user logging in (or if user= is specified, owned by that user). This option disables that check.

This option can be used to allow daemons not running as root to still handle configuration files not owned by that user, for example owned by the users themselves.

allowed_perm=0nnn
------------------------------------
DANGEROUS OPTION!

By default, the PAM module requires the secrets file to be readable only by the owner of the file (mode 0600 by default). In situations where the module is used in a non-default configuration, an administrator may need more lenient file permissions, or a specific setting for their use case.

debug
------------------------------------
Enable more verbose log messages in syslog.

try_first_pass / use_first_pass / forward_pass
-------------------------------------------------------
Some PAM clients cannot prompt the user for more than just the password. To work around this problem, this PAM module supports stacking. If you pass the forward_pass option, the pam_google_authenticator module queries the user for both the system password and the verification code in a single prompt. It then forwards the system password to the next PAM module, which will have to be configured with the use_first_pass option.

In turn, pam_google_authenticator module also supports both the standard use_first_pass and try_first_pass options. But most users would not need to set those on the pam_google_authenticator.

noskewadj
------------------------------------
If you discover that your TOTP code never works, this is most commonly the result of the clock on your server being different from the one on your Android device. The PAM module makes an attempt to compensate for time skew. You can teach it about the amount of skew that you are experiencing, by trying to log it three times in a row. Make sure you always wait 30s (but not longer), so that you get three distinct TOTP codes.

Some administrators prefer that time skew isn't adjusted automatically, as doing so results in a slightly less secure system configuration. If you want to disable it, you can do so on the module command line:

``auth required pam_google_authenticator.so noskewadj``

no_increment_hotp
------------------------------------
Don't increment the counter for failed HOTP attempts. This is important if log attempts with failed passwords still get an OTP prompt.

nullok
------------------------------------
Allow users to log in without OTP, if they haven't set up OTP yet.

echo_verification_code
------------------------------------
By default, the PAM module does not echo the verification code when it is entered by the user. In some situations, the administrator might prefer a different behavior. Pass the echo_verification_code option to the module in order to enable echoing.

If you would like verification codes that are counter based instead of timebased, use the google-authenticator binary to generate a secret key in your home directory with the proper option. In this mode, clock skew is irrelevant and the window size option now applies to how many codes beyond the current one that would be accepted, to reduce synchronization problems.

.. _encrypted home directories:

3. Encrypted home directories
=========================================
If your system encrypts home directories until after your users entered their password, you either have to re-arrange the entries in the PAM configuration file to decrypt the home directory prior to asking for the OTP code, or you have to store the secret file in a non-standard location:

auth required pam_google_authenticator.so secret=/var/unencrypted-home/${USER}/.google_authenticator

would be a possible choice. Make sure to set appropriate permissions. You also have to tell your users to manually move their .google_authenticator file to this location.

In addition to "${USER}", the secret= option also recognizes both "~" and ${HOME} as short-hands for the user's home directory.

When using the secret= option, you might want to also set the user= option. The latter forces the PAM module to switch to a dedicated hard-coded user id prior to doing any file operations. When using the user= option, you must not include "~" or "${HOME}" in the filename.

The user= option can also be useful if you want to authenticate users who do not have traditional UNIX accounts on your system.
