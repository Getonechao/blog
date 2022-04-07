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