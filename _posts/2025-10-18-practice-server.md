---
layout: single
title: "服务器与网络"
date: 2025-10-17 10:00:00 +0800
categories: [practice]
tags: [环境]
toc: true
---

### 服务器简介
服务器就是电脑。连接网络后，有ip地址，可以通过ip地址来访问服务器，对服务器进行远程控制。

每一个网站，本质上也是一个电脑，一个服务器，有ip地址，只是用域名（网址）代替，更好记忆。

### 连接互联网的过程
1. DNS解析：网址==域名，实际上是ip地址的别称，方便记忆。所以输入网址后的第一步就是将域名（www.baidu.com）转换为ip地址。
2. TCP连接：ip地址对应一个电脑，一个服务器。本地的电脑（客户端）通过TCP协议与目标电脑（服务器）进行连接。TCP具体过程包含三次握手handshake。完成后就可以进行数据传输了。
3. SSL/TLS（如果使用HTTPS）：HTTPS网站会在TCP的基础上进行SSL/TLS握手handshake，也就是进行一次加密通信，保证安全。
4. HTTP请求与响应：完成连接后，客户端就可以发送HTTP请求（如GET&POST），服务器收到请求，返回响应（如HTML页面、json数据等），然后客户端处理响应（比如浏览器开始渲染页面）


### 服务器分配用户类型
1. 粗暴分配用户：所有人用同一个ip地址和端口，但是每个人分配不同的用户。这样做的好处在于简单，缺点在于不同人的数据互相可以看到，不安全。同时个人的系统操作可能会影响他人。
2. 沙盒与容器：docker等是工具，用于构建一个个容器，分配给不同的用户。容器之间相互隔开，以保证操作的独立性，互不影响。容器之间遵循沙盒，也就是一种安全机制，用于隔离不同用户的操作。最终的结果就是每个人都分配到一个容器，自己是root用户，仿佛拥有一个独立的电脑。


### 服务器踩坑
#### 总体感悟
- 报错与代码：遇到问题慢一点，一行行跑代码，尝试理解报错信息，然后以ai助手作为老师/参考进行反馈和纠正，最终既能积累经验提高水平，又能借助ai助手快速解决问题。不要妄想无脑稀里糊涂跑一遍，无脑黑盒化和AI助手交互，自己充当转发/操作手。否则只会越来越多报错，自己越来越慌，花更多时间。
    - 一个例子就是配环境，想直接跑`install.sh`然后将报错信息给GPT，将GPT的建议返回做操作。几个操作下来只会报错越来越多，而且自己心越虚。可能的原因是GPT本身就不完美，而且我的操作也不一定和GPT一样，所以误差累积越来越大；另一个例子是报错信息，看都不看无脑扔给GPT，有时候能行，有时候不行，产生更多错误，继续无脑几个来回，报错只会越来越多。
    - 目前ai还不是全能，自己不能无脑。而且无脑最终也会被替代。不如花点时间，理解一下。

#### ConnectionResetError（104）
`ConnectionResetError: [Errno 104] Connection reset by peer`一般报错于连接远程服务器/连网页时，意思是远程服务器关闭了TCP连接，所以无法继续通信。可能会看到SSL程序错误，TCP程序错误等。

一般原因就三个：服务器bug，本地客户端（程序）bug，网络连接bug。使用内网上的服务器，极有可能是网络连接bug，即管理员设置了防火墙不让访问外面的网站。

#### apt报错: W: unsandboxed as root ... pkgAcquire::Run (13: Permission denied)
W == Warning，是警告，不是报错！所以可以忽略，因为警告并不中断输入命令的操作进程。比如`apt install tmux`会报警告，但是报完了也装好了。[参考](https://askubuntu.com/questions/908800/what-does-this-apt-error-message-download-is-performed-unsandboxed-as-root)

完整警告信息`download is performed unsandboxed as root as file 'xxxx' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)`: 默认apt操作的用户是_apt，这是一种沙盒化操作来保证系统安全。但是在下载时发现权限不够，所以自动变为了root用户来操作，下载file xxx，原因就是_apt用户遇到了13 Permission denied。

#### 配环境
常见的是`requirements.txt`和`install.sh`，看着挺好，一行命令就自动配好环境。但是和大家聊完发现没人一行成功过，所以别想了。配环境就是花时间的。所以推荐的做法还是一行一行敲，这样可以清晰地知道哪一步出了问题。

配环境报错很正常，看着不是很严重可以忽略，先继续配。比如包冲突，发现最后也能跑；或者是某个包装不上，最后发现装完其他的再回来装就装好了。所以，跑通最重要，配环境有错误时小试一下解决，不行的话可以继续配，先配完了看看怎么回事。能跑代码，结果也没问题，就不管了呗先。

#### github
公开仓库以https可以直接git clone，不需要绑定ssh key；git@才需要绑定，因为是走链接。

--recursive: 工程项目不可避免要用到其他人开发的第三方库。一种方式是pip install二进制文件，方便简单；第二种方式则是git clone第三方仓库，从源码安装，这样可以直接对第三方库的功能进行修改。第二种方式，就涉及到recursive，即仓库中还有其他仓库,所以递归地都clone下来。具体git命令可以查`git submodules`，等价的安装方式是对应到子仓库目录下，将子仓库的github代码解压在此。最后是安装`pip install -v -e /path/to/submodule`完成从源码安装，就可以import submodule了。

#### nvidia驱动、cuda、cudnn
nvidia驱动：drive GPU，驱动gpu进行工作的程序。输入接口程序的指令，输出GPU硬件的操作电平。

cuda：compute unified device architecture。cuda是一个生态，代表英伟达GPU不仅可以进行画面渲染，还可以进行通用并行计算。cuda生态的编程和驱动由cuda toolkit实现，toolkit用来编程GPU程序并执行对应的行为。所以很多用到英伟达GPU做计算的工具包，都以cuda toolkit来开发，所以工具包就要包含toolkit。

cudnn：nvidia开发的用于dnn的计算包。

pytorch：用cuda toolkit开发的深度学习包，安装pytorch时自带必要的toolkit工具，不需要另外配。

版本之间的关系：
- 由底层到顶层，高版本可以兼容低版本，反之不行。所以nvidia驱动版本尽可能高，这样可以支持高的cuda版本，从而支持高的pytorch。
- pytorch和cuda版本是一一对应的。但是不需要安装cuda toolkit，因为pytorch包包含了必要的cuda toolkit组件。所以只需要确认nvidia驱动>=pytorch对应的cuda版本就可以。
- conda install cuda toolkit，只是为了保证工具齐全。其实对应到pip，也许就不用安装，因为python包自己会自带对应的cuda工具。
