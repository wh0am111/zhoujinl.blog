## 项目简介

此项目通过使用Hexo 搭建静态博客网站。  
有关Hexo介绍，详见官方网站 [Hexo](https://hexo.io/)  
搭建参考链接 [Hexo使用指南-简书](http://www.jianshu.com/p/84a8384be1ae)  
主题使用的是Yilia [Hexo主题Yilia](https://github.com/litten/hexo-theme-yilia) 



##简易部署命令

**前提是已经安装好hexo、npm命令**
**以下命令均在项目根路径下运行**
1. 生成静态文件  
  执行命令：`hexo generate`
  
2. 运行服务  
  执行命令：  `hexo server`  打开命令行提示的地址，一般是http://localhost:4000/
3. 部署到github
  执行命令：  `hexo deploy`  
  **此步骤需先安装 deployer-git  `npm install hexo-deployer-git --save`  **


**为保存图片，在/source目录下，新建目录img . 引用方法如下** 
```  
![Alt text](/img/bmw.jpg "宝马")  
```



## 提交流程

1. 创建博客文章``` hexo new blog ``` 

2. 创建静态页面 ```hexo generate```

3. 提交到git项目

   ```shell
   git add  .
   git commit -m "update"
   git push origin master
   ```

4. 部署博客文章到zhoujinl.git.io 

   需要先修改_config.yml ，如下，输入正确的用户名密码

   ```
   deploy:
     type: git
     #repository: https://name:passwd@github.com/zhoujinl/zhoujinl.github.io.git
     repository: https://zhoujinl:****@github.com/zhoujinl/zhoujinl.github.io.git
     branch: master
     message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"
   ```

   然后再执行部署命令 `hexo deploy`

​       注：记得密码再次修改，不要上传到github上面泄漏了   ╮(￣▽ ￣)╭
