#个人博客

##搭建流程

[流程](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)

##分支说明

* master分支存放hexo生成的静态页面
* hexo分支存放hexo生成工具（即对post、themes等一些文件的备份，themes如果clone别的repos记得把.git移除）

##多终端

家里公司都可以写博客

首次

1. git clone xxxx.git
2. npm install hexo -g
3. npm install
4. npm install hexo-deployer-git -g

##日常改动流程

1. git add .
2. git commit -m "xxx"
3. git push origin hexo
4. hexo g -d 发布到网站的master分支