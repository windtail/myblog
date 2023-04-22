---
title: "ssh port forwarding"
date: 2017-12-27T19:07:00+08:00
draft: false
---

SSH端口转发，总是忘记，今天记录下。端口转发有两种，一个是local一个是remote（可能还有一种dynamic，还没有研究）


贴个链接 <https://www.ssh.com/ssh/tunneling/example#sec-What-Is-SSH-Port-Forwarding-aka-SSH-Tunneling>


看完这个链接基本就会了，如果还不会再看 man ssh 里，找 -L和-R选项


### Local转发


ssh -NL [bind-address]::  


执行ssh命令的机器（ssh客户端），想通过  访问 : ，那么在执行了上述命令后，就可以访问 127.0.0.1: 来访问:了。[bind-address]是一个可选项，如果不指定的话，那么只要能访问到  的机器就都可以访问到 : 了，比如另一台和命令执行的机器在同一个局域网内的机器。


-N选项的目的是不执行连接时的命令（一般是shell），只进行端口转发，还可以指定 -f，放到后台运行。


这个功能一般用来访问只有  能访问到的资源，比如，那些跑在  上的数据库，因安全问题，数据库设置成仅本地可访问，当我们需要使用ui工具时，就可以远程端口转到本地来。例如要访问  上的 postgres 服务，则需要在本地执行 ssh -NL 5432:localhost:5432 


### Remote转发


ssh -NL [`clientspecified`-address]::  


执行ssh命令的机器（ssh客户端），希望  通过自己可以访问到 :，在执行了上述命令后，就可以通过访问 :来等效为 执行ssh命令的机器去访问 :


如果  在公网上，这个功能可以将本机的服务暴露到公网上去，供人访问。例如本机有一个测试的web服务（比如微信开发时，需要和微信的服务器交互）。这个功能还需要注意 sshd\_config 中的 `GatewayPorts` 选项，no/yes/`clientspecified，no表示只能  访问，而 yes 表示大家都可以访问， `clientspecified`表示由 [`clientspecified`-address]`指定谁可以访问。




---


 


这两个功能在开发时都比较实用，PS：remote转发我还没有试过，先写在这里


 


