---
layout: post
title: "Posting Blog"
subtitle: "Fast Blog Settings"
date: 2021-08-27 17:25:13
header-style: text
catalog: true
author: "Yuan"
tags: 
    - rake
    - blog setting
---
> 快速发布博客

##### Rakefile
设置好根目录下的Rakefile，重点设置好时间之类的参数，然后在terminal里运行：

rake post title="xxxx" subtitle="xxx"

即可自动生成文件名，然后打开对应的md文件，书写博客，里面的参数根据需要可以调整。
我在原作者的Rakefile基础上，增加：

date2 = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d %H:%M:%S')

同时修改：

post.puts "date: #{date2}"

从而得到更准确的头设置

---
