# 项目简介

此项目通过使用Hexo 搭建静态博客网站。  
有关Hexo介绍，详见官方网站 [Hexo](https://hexo.io/)  
搭建参考链接 [Hexo使用指南-简书](http://www.jianshu.com/p/84a8384be1ae)  
主题使用的是Yilia [Hexo主题Yilia](https://github.com/litten/hexo-theme-yilia) 


# 简易部署命令
**前提是已经安装好hexo、npm命令**
**以下命令均在项目根路径下运行**
1. 生成静态文件  
  执行命令：`hexo generate`
  
2. 运行服务  
  执行命令：  `hexo server`  打开命令行提示的地址，一般是http://0.0.0.0:4000/
3. 部署到github
  执行命令：  `hexo deploy`  
  **此步骤需先安装 deployer-git  `npm install hexo-deployer-git --save`  **


***为保存图片，在/source目录下，新建目录img***  
***引用方法如下***  
```  
![Alt text](/img/bmw.jpg "宝马")  
```