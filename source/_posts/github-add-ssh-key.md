title: Github add SSH key
date: 2015-09-25 17:16:57
tags: [Github,SSHKey]
categories: GitHub
---
sshkey是来用来确定受信任的计算机，而不涉及密码的一种方法。下面的步骤将引导您完成生成SSH密钥并添加公共密钥到你的GitHub上的帐户。

<!--more-->

## 1.检查电脑上是否已经有SSH key

打开Git bash

``` bash
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist
```

这两个命令就是检查是否已经存在 id_rsa.pub 或 id_dsa.pub 文件，如果文件已经存在，那么你可以跳过步骤2，直接进入步骤3

Tip:如果.ssh目录不存在，可以在第二部创建

## 2.创建一个SSHkey

``` bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label

Generating public/private rsa key pair.
Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
```

代码参数含义：

* -t：指定密钥类型，默认是 rsa ，可以省略。
* -C：设置注释文字，比如邮箱。
* -f：指定密钥文件存储文件名(官方强烈建议用默认名字，那么就会生成 id_rsa 和 id_rsa.pub 两个秘钥文件)

以上代码省略了`-f`参数，因此，运行上面那条命令后会让你输入一个文件名，用于保存刚才生成的 SSH 

``` bash
Enter passphrase (empty for no passphrase): [Type a passphrase]
# Enter same passphrase again: [Type passphrase again]
```

接下来输入密码，该密码是你用来push文件时需要输入的密码（非github的密码），接下来你会得到你的sshkey的指纹
如下所示：

``` bash
Your identification has been saved in /Users/you/.ssh/id_rsa.
# Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

当你看到上面这段代码的收，那就说明，你的SSH key已经创建成功，你只需要添加到github的SSH key上就可以了

## 3.添加你的SSH key到Github上去

1. 首先你需要拷贝 id_rsa.pub 文件的内容，你可以用编辑器打开文件复制，也可以用git命令复制该文件的内容，如：

    ``` bash
        $ clip < ~/.ssh/id_rsa.pub
    ```

2. 登录你的github账号，从又上角的设置（ Account Settings ）进入，然后点击菜单栏的 SSH key 进入页面添加 SSH key。
3. 点击 Add SSH key 按钮添加一个 SSH key 。把你复制的 SSH key 代码粘贴到 key 所对应的输入框中，记得 SSH key 代码的前后不要留有空格或者回车。当然，上面的 Title 所对应的输入框你也可以输入一个该 SSH key 显示在 github 上的一个别名。默认的会使用你的邮件名称。

## 4. 测试一下该SSH key

在git bash中输入以下代码：

``` bash
$ ssh -T git@github.com
```

当你输入以上代码时，会有一段警告代码，如：

``` bash
The authenticity of host 'github.com (207.97.227.239)' can't be established.
# RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
# Are you sure you want to continue connecting (yes/no)?
```

这是正常的，你输入 yes 回车既可。如果你创建 SSH key 的时候设置了密码，接下来就会提示你输入密码，如：

``` bash
Enter passphrase for key '/c/Users/Administrator/.ssh/id_rsa':
```

当然如果你密码输错了，会再要求你输入，知道对了为止。
注意：输入密码时如果输错一个字就会不正确，使用删除键是无法更正的。
密码正确后你会看到下面这段话，如：

``` bash
Hi username! You've successfully authenticated, but GitHub does not
# provide shell access.
```

如果用户名是正确的,你已经成功设置SSH密钥。如果你看到 “access denied” ，者表示拒绝访问，那么你就需要使用 https 去访问，而不是 SSH 

参考文档：https://help.github.com/articles/generating-ssh-keys/