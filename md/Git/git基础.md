
+ 配置`SSH Key`(没有安装过git的)
    + `ssh-keygen -t rsa -C "youremail@example.com"`
    + 可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥
    + 登陆`GitHub`，打开“`Account settings`”，“`SSH Keys`”页面,然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容

+ Push本地到远程
    + `git init` --初始化仓库(如果没有初始化)
    + `git add <folder>` -- 添加文件夹到暂存区
    + `git commit -m "first commit"` -- 确认提交
    + `git push -u origin master` -- 提交本地master分支到远程
