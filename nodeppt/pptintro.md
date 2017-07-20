title: nodeppt介绍
speaker: liupc
url: https://github.com/ksky521/nodeppt
transition: cards

[slide]
# nodeppt介绍
## 演讲者：liupc

[slide style="background-image:url('/img/bg1.png')"]
- nodeppt是基于nodejs写的支持 **Markdown!** 语法的网页PPT {:&.moveIn}
- Github：https://github.com/ksky521/nodeppt

[slide]
## 安装
----

```bash
npm install -g nodeppt
```

[slide]
## shell使用
----

```bash
# 获取帮助 
nodeppt start -h 

# 绑定端口 
nodeppt start -p <port>
```

```bash
nodeppt start -p 8090 -d path/for/ppts 
# 绑定host，默认绑定0.0.0.0 
nodeppt start -p 8080 -d path/for/ppts -h 127.0.0.1 
# 使用socket通信（按Q键显示/关闭二维码，手机扫描，即可控制） 
# socket须知：1、注意手机和pc要可以相互访问，2、防火墙，3、ip 
nodeppt start -c socket 
# 不加-c默认使用postMessage，窗口联动，即list页面【多窗口】链接
```

[slide]
## 导出Html
----

```bash
# 获取generate帮助 
nodeppt generate -h 
# 使用generate命令 
nodeppt generate filepath 
# 导出全部，包括nodeppt的js、img和css文件夹 
# 默认导出在publish文件夹 
nodeppt generate ./ppts/demo.md -a 
# 指定导出文件夹 
nodeppt generate ./ppts/demo.md -a -o output/path
# 导出目录下所有ppt，并且生成ppt list首页
nodeppt path -o output/path -a
```


