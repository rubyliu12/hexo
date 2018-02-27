---
title: IntelliJ IDEA 插件
date: 2017-06-01 14:07:09
categories: [Intellij]
tags: [IDE, plugins]
description:
permalink: idea
---
## IntelliJ插件篇
###### Maven Helper
>我一般用这款插件来查看maven的依赖树。在不使用此插件的情况下，要想查看maven的依赖树就要使用Maven命令maven dependency:tree来查看依赖。想要查看是否有依赖冲突也可以使用mvn dependency:tree -Dverbose -Dincludes=<groupId>:<artifactId>只查看关心的jar包，但是这样还是需要我执行命令，并且当项目比较复杂的时候，这个过程是比较漫长的。maven helper就能很好的解决这个问题。

###### GsonFormat
>快捷键Alt+S或者鼠标右键选择Generate>GsonFormat

<!-- more -->
###### InnerBuilder
>快捷键Alt+Insert、Shift+Alt+B或者鼠标右键选择Generate->Builder，一般给实体类建立get方法，实体类的属性使用final标记，对于不能空的值，做为Builder构造方法的参数传入，最好有一个实体类做为参数的Builder的构造方法

## IntelliJ Setting配置篇
###### spelling
>去掉拼写检测Setting->inspections->spelling，去掉勾

###### properties编码配置
>Setting->Editor->File Encodings，Default encoding for properties files选择UTF-8，并勾上Transparent native-to-ascii conversion

###### 修改注释模版
>Setting->Editor->File And Code Templates，选择要修改的文件类型，然后在Includes的File Header中修改

###### 代码快捷生成（Live Templates）
>可以根据自己需求调整快捷模版，比如sout、soutv、soutm、psvm等

###### 快捷提示（Code Completion）
>Case sensitive completion可选First letter（首字母匹配）、All（全部匹配）、None（不需要匹配，就是忽略大小写）

###### 软换行
>Settings-Editor-General，勾上use soft wraps in editor
>Settings->Code Style->General->Right margin(columns)
>Settings->Code Style->Java->Wrapping and Braces->Ensure rigth margin is not exceeded

###### 虚拟空格（光标在编辑器的任意位置都可以点）
>Settings-Editor-General，去掉Allow placement of caret at end of line前面的勾

###### 折叠代码
>默认单行方法是折叠的，在settings-Editor-General-Code Folding，取消One-line methods就不会折叠了

###### 自动保存文件
>Settings-Appearance & Behivior-System Settings，取消“Synchronize file on frame activation” 和“Save files on framedeactivation”的选择。同时我们选择"Save files automatically", 并将其设置为30秒，这样IDEA依然可以自动保持文件,所以在每次切换时，你需要按下Ctrl+S保存文件

###### 修改过未保存的文件，使用*号标识出来
Settings-Editor-General-Editor Tabs-Mark modified tabs width asterisk

## 快捷键篇
1. Ctrl + E：最近打开的文件
2. Ctrl + Shift + E：最近编辑的文件
3. Double Shift：跳到特定文件夹
4. Ctrl + Shift + Enter：快速补全行末分号
5. Ctrl + Shift + A：Rest Client
6. Ctrl + Shift + V：粘贴版历史
7. Alt + Enter：Language Injection功能，将一个字符串标记为 JSON，就可以非常方便地编写 JSON 了，再也不用担心转义的问题了；同时也可以支持简单正则测试能力
8. Shift + F7：Debug时可以选择进入哪个方法调试
9. Ctrl + Alt + L：格式化代码

##第三方jar包篇
###### fastjson
>FastJSON序列化特殊字符BUG
>问题重现：JSON.toJSONString("\u0001\u0080\u0002");
>在1.1.41+的版本已经修正此问题，https://github.com/alibaba/fastjson/commit/cdf7cb253e961666e2b3c2bdd423abe73ba4324a#diff-0

## 其他
###### idea maven乱码
配置Maven -> Runner -> VM Options
-Dfile.encoding=gbk
