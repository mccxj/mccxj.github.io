---
layout: post
comments: true
title: "生成svn补丁的回退补丁"
description: "svn补丁回退使用"
categories: ["实践", "svn", "codediff", "工具", "回退"]
---

### 生成回退补丁的脚本
{% highlight ruby %}
open("C:/Users/peipei/Desktop/svn2.diff", 'w') do |f|
    open("C:/Users/peipei/Desktop/svn.diff", 'r') do |ff|
        ff.readlines.each do |line|
            if line.start_with?("@@")
                line =~ /@@\ \-(.+)\ \+(.+)\ @@/
                f << "@@\ \-#{$2}\ \+#{$1}\ @@\n"
            elsif line.start_with?("+ ")
                f << "-#{line.slice(1..-1)}"
            elsif line.start_with?("- ")
                f << "+#{line.slice(1..-1)}"
            else
                f << line
            end
        end
    end
end
{% endhighlight %}

### 其他计划

1. 在codediff工具中添加通过文件名查询功能(已经实现)
2. 通过codediff工具直接生成提交补丁和回退补丁(这个延后)
3. 能够支持补丁合并(这个不是那么好弄)  

### 关于代码评审
已经有个简单的方案了，我决定在codediff应用上支持每周代码评审功能，这周上线试用一下。
