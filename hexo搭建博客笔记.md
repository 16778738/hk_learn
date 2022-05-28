# hexo和github搭建博客

## 安装 Hexo、git

- Node.js 官网下载地址：https://nodejs.org/en/download/
- Git 官网下载地址：https://git-scm.com/downloads

~~~~
$ npm install hexo-cli -g
$ npm install hexo-deployer-git --save
~~~~

![image-20220407233523233](E:\学习\picture\image-20220407233523233.png)

### hexo初始化

在刚才新建的文件夹里面再次**新建**一个 Hexo 文件夹,进入该 Hexo 文件夹右键鼠标，点击 Git Bash Here，输入以下命令，如图所示则安装成功

~~~~
hexo init
~~~~

![image-20220407234154088](E:\学习\picture\image-20220407234154088.png)

## 本地查看

这里一定是 http://localhost:4000/

有的浏览器默认打开的**https**的需要注意，显示失败

~~~
hexo generate
~~~

![image-20220407234254427](E:\学习\picture\image-20220407234254427.png)

~~~~
hexo server
~~~~

![image-20220407234319620](E:\学习\picture\image-20220407234319620.png)

![image-20220407234425355](E:\学习\picture\image-20220407234425355.png)

## github部署仓库

1. 注册 Github 账户：[点击此处](https://github.com/)访问 Github 官网，点击 Sign Up 注册账户

2. 创建项目代码库：点击 New repository 开始创建

   

3. 配置 SSH 密钥：只有配置好 SSH 密钥后，我们才可以通过 git 操作实现本地代码库与 Github 代码库同步，在你第一次新建的文件夹里面

   ~~~~
   $ ssh-keygen -t rsa -C "your email@example.com"
   //引号里面填写你的邮箱地址
   ~~~~

   ~~~~~
   Generating public/private rsa key pair.
   Enter file in which to save the key (/c/Users/you/.ssh/id_rsa):
   //到这里可以直接回车将密钥按默认文件进行存储
   ~~~~~

   ~~~
   Enter passphrase (empty for no passphrase):
   //这里是要你输入密码，其实不需要输什么密码，直接回车就行
   Enter same passphrase again:
   ~~~

   ~~~~
   Your identification has been saved in /c/Users/you/.ssh/id_rsa.
   Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
   The key fingerprint is:
   这里是各种字母数字组成的字符串，结尾是你的邮箱
   The key's randomart image is:
   这里也是各种字母数字符号组成的字符串
   ~~~~

   可以寻找本地C盘用户下存储的.ssh下的id_rsa.pub

4. 将自己生成的公钥id_rsa.pub内容复制添加到github

5. 测试

   输入以下命令：注意：[git@github.com](mailto:git@github.com)不要做任何更改！

   ![image-20220407235454023](E:\学习\picture\image-20220407235454023.png)

6. 配置个人信息

   ~~~~
   git config --global user.name "此处填你的用户名"
   git config --global user.email "此处填你的邮箱"
   ~~~~

   

## 将hexo部署到github

- 找到自己的仓库复制ssh

  ~~~~
  git@github.com:xxxxx/xxxxx.github.io.git
  ~~~~

- 将ssh复制到本地的_config.yml

  ~~~~
  # Deployment
  ## Docs: https://hexo.io/docs/one-command-deployment
  deploy:
    type: git
    repository: git@github.com:16778738/16778738.github.io.git
    branch: master
  
  ~~~~

  ![image-20220408000404036](E:\学习\picture\image-20220408000404036.png)

- 在 Hexo 文件夹下分别执行以下命令

  ~~~~
  hexo g -d
  ~~~~

  

![image-20220408000333190](E:\学习\picture\image-20220408000333190.png)

执行完之后会让你输入你的 Github 的账号和密码，如果此时报以下错误，说明你的 deployer 没有安装成功

```
ERROR Deployer not found: git
```

需要执行以下命令再安装一次：

```
npm install hexo-deployer-git --save
```

![image-20220408000531165](E:\学习\picture\image-20220408000531165.png)

## 访问博客

你的博客地址：https://你的用户名.github.io

[点击此处](https://hexo.io/themes/)进入 Hexo 官网的主题专栏

找到自己喜欢的，克隆他的模板 Github 上的地址

打开 Hexo 文件夹下的 themes 目

录右键 Git Bash Here，输入以下命令：

```
$ git clone 此处填写你刚才复制的主题地址
```

等待下载完成后即可在 **themes** 目录下生成 **hexo-theme-aero-dual** 文件夹，然后打开 Hexo 文件夹下的配置文件 _config.yml ，找到关键字 **theme**，修改参数为：**theme：hexo-theme-aero-dual （其他主题修改成相应名称即可）**，再次注意冒号后面有一个空格！

~~~
$ hexo clean  
//该命令的作用是清除缓存，若不输入此命令，服务器有可能更新不了主题
$ hexo g -d
~~~

