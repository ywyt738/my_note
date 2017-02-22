.. _git_guide:

******************
Git操作手册
******************
master  ：默认开发分支

origin  ：默认远程版本库

创建版本库
==============
::

    $ git clone <url>                   #克隆远程版本库
    $ git init                          #初始化本地版本库

修改和提交
==============
::

    $ git status                        #查看状态
    $ git add .                         #跟踪所有改动过的文件
    $ git add <file>                    #跟踪指定文件
    $ git commit -m "commit message"    #提交所有更新过的文件
    $ git commit --amend                #修改最后一次提交

撤销
==============
::

    $ git reset --hard HEAD             #撤销工作目录中所有未提交文件的修改内容
    $ git checkout HEAD <file>          #撤销指定的未提交文件的修改内容
    $ git revert <commit>               #撤销指定提交

分支与标签
==============
::

    $ git branch                        #显示所有本地分支
    $ git checkout <branch/tag>         #切换到指定分支或者标签
    $ git branch <new branch>           #创建新分支
    $ git branch -d <branch>            #删除本地分支
    $ git tag                           #显示所有本地标签
    $ git tag <tagname>                 #基于最新提交创建标签
    $ git tag -d <tagname>              #删除标签

合并
==============
::

    $ git merge <branch>                #合并指定分支到当前分支

远程操作
==============
::

    $ git remote -V                     #查看远程仓库版本信息
    $ git remote show <remote>          #查看指定远程版本库信息
    $ git remote add <remote> <url>     #添加远程版本库
    $ git fetch <remote>                #从远程仓库获取代码
    $ git pull <remote> <branch>        #下载代码及快速合并
    $ git push <remote> <branch>        #上传代码及快速合并
    $ git push --tag                    #上传所有标签

Git结合坚果云的使用方法
=======================
1. 将本地的项目仓库进行git初始化
::

    $ git init

2. 将所有目录下所有现有项目文件加入git仓库
::

    $ git add *

3. 查看是否所有文件已经被加入
::

    $ git status

4. 进行第一次提交
::

    $ git commit -m "first commit"

5. 将中央仓库作为自己的远程核心仓库（例：云盘同步文件假为f:\\Git_repo\\）
::

    $ git remote add origin /f/Git_repo/FORM.git

6. 将远程仓库作为自己的默认推送仓库
::

    $ git push -u origin master

7. java gitignore file
::

    *.class

    $ Mobile Tools for Java (J2ME)
    .mtj.tmp/

    $ Package Files $
    *.jar
    *.war
    *.ear

    $ virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
    hs_err_pid*
