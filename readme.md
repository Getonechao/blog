# hugo配置（yaml）

## 忽略错误？
ignoreErrors: ["error-remote-getjson"]

## hugo中使用html格式
~~~
markup:
  goldmark:
    renderer:
      unsafe: true
~~~

https://gohugo.io/news/0.60.0-relnotes/

## bilibili aid
console.log(playerInfo.aid)
参考blog
https://blog.iyu.icu/posts/shortcode_bilibili/?msclkid=f85a7316b29211ecb1be8ce0a43382cb

## 引用其他markdown
[Neat]({{< ref "blog/neat.md" >}})



## 上传命令

~~~
hugo -D

git add .

git commit -m ""

git push gitee

cloudbase hosting deploy ./public  -e  blog-0g8860131649bb29
~~~






## 部署到腾讯云

~~~shell
需要安装node.js环境
npm install -g @cloudbase/cli

登入腾讯云
tcb login

在hugo根目录下执行这条命令
cloudbase hosting deploy ./public  -e  blog-0g8860131649bb29
~~~

