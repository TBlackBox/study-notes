

#  简介

Netcat简称nc，netcat是**网络工具中的瑞士军刀**，它能通过**TCP和UDP在网络中读写数据**，通过与其它工具结合和重定向，你可以在**脚本中**以多种方式使用它，使用netcat命令所能完成许多让人惊讶的事情。(该工具可以直接理解为：可以通过命令行的形式监听端口，也可以通过命令行的形式发送数据，不仅仅如此，还有一些牛逼的功能，例如：将TCP，UDP,SCTP重定向到其他站点，支持SSL,可以通过SOCKET4 SOCKT5或HTTP代理等等)



一般Netcat有两个版本，一个版本是不提供反向连接的版本，一个是全功能版本。这两者的区别就是是否带-e参数，只有带-e参数的版本才支持反向连接。



[ncat用户指南](https://nmap.org/ncat/guide/index.html)完成的文档，包含许多技巧，敲门和实际案列。

[Ncat用户手册](https://nmap.org/book/ncat-man.html)提供了快速使用的摘要。

# 版本历史

nc 版本很乱，有很多版本，有不同的作者编写。

## netcat-traditional

这个是最早的版本，最新版本是2007年1月，版本是1.10，Kali Linux默认带的就是这个版本。

这个版本的nc具有`-e`选项，十分方便反弹shell使用

Windows版本的netcat下载地址：https://eternallybored.org/misc/netcat/

## netcat-openbsd

ubuntu里默认的nc命令指向的是netcat-openbsd。这个版本因为考虑到安全性等原因没有`-e`选项。所以我们得手动替换一下nc的版本：

```shell
# 安装传统的netcat
sudo apt-get install netcat

# 切换版本
sudo update-alternatives --config nc
```

## ncat(这里主要学这个版本)

这是netcat的比较新的现代版本，它是从头开始编写的，不使用原始的netcat代码。**ncat的作者也是Nmap程序的作者**。ncat几乎重复了原始程序的所有功能，并包含其他功能。**CentOS、Red Hat默认带的是ncat**。目前ncat已经集成到了nmap里面，安装完nmap后就可以使用`ncat`命令了。



# 下载与安装

Ncat与Nmap集成在一起，可从Nmap下载页面获得的标准Nmap下载软件包（包括源代码以及Linux，Windows和Mac二进制文件）中获得。

Linux版本下载地址：https://nmap.org/ncat/

Windows版本下载地址：http://nmap.org/dist/ncat-portable-5.59BETA1.zip



# ncat使用

命令格式

```
ncat [ <OPTIONS> ...] [ <hostname> ] [ <port> ]
```

# 总览

命令：ncat -h

```
用法：ncat [选项] [主机名] [端口]

花费时间的选项以秒为单位。附加“ ms”毫秒
“ s”表示秒，“ m”表示分钟，或“ h”表示小时（例如500ms）。
  -4仅使用IPv4
  -6仅使用IPv6
  -U，-unixsock仅使用Unix域套接字
      --vsock仅使用vsock套接字
  -C，--crlf将CRLF用于EOL序列
  -c，--sh-exec <命令>通过/ bin / sh执行给定命令
  -e，--exec <命令>执行给定的命令
      --lua-exec <文件名>执行给定的Lua脚本
  -g hop1 [，hop2，...]松散的源路由跳跃点（最大8个）
  -G <n>松散的源路由跃点指针（4、8、12，...）
  -m，--max-conns <n>最大<n>同时连接
  -h，--help显示此帮助屏幕
  -d，--delay <time>在读/写之间等待
  -o，--output <文件名>将会话数据转储到文件中
  -x，--hex-dump <文件名>将会话数据以十六进制形式转储到文件中
  -i，--idle-timeout <时间>空闲读/写超时
  -p，--source-port port指定要使用的源端口
  -s，--source addr指定要使用的源地址（不影响-l）
  -l，-listen绑定并监听传入的连接
  -k，--keep-open在侦听模式下接受多个连接
  -n，--nodns不通过DNS解析主机名
  -t，--telnet回答Telnet协商
  -u，-udp使用UDP而不是默认TCP
      --sctp使用SCTP代替默认TCP
  -v，--verbose设置详细级别（可以多次使用）
  -w，--wait <时间>连接超时
  -z  零 I/O模式，仅报告连接状态
      --append-output附加而不是破坏指定的输出文件
      --send-only仅发送数据，忽略接收；退出EOF
      --recv-only仅接收数据，从不发送任何内容
      --no-shutdown在stdin上收到EOF时继续半双工
      --allow只允许给定的主机连接到Ncat
      --allowfile允许连接到Ncat的主机文件
      --deny拒绝给定主机连接到Ncat
      --denyfile拒绝连接到Ncat的主机文件
      --broker启用Ncat的连接代理模式
      --chat启动一个简单的Ncat聊天服务器
      --proxy <addr [：port]>指定要通过其代理的主机的地址
      --proxy-type <类型>指定代理类型（“ http”，“ socks4”，“ socks5”）
      --proxy-auth <auth>使用HTTP或SOCKS代理服务器进行身份验证
      --proxy-dns <类型>指定解析代理目标的位置
      --ssl使用SSL连接或监听
      --ssl-cert指定用于侦听的SSL证书文件（PEM）
      --ssl-key指定用于侦听的SSL私钥（PEM）
      --ssl-verify验证证书的信任和域名
      --ssl-trustfile包含受信任SSL证书的PEM文件
      --ssl-ciphers包含要使用的SSL密码的密码列表
      --ssl-servername请求不同的服务器名称（SNI）
      --ssl-alpn要使用的ALPN协议列表
      --version显示Ncat的版本信息并退出

```



## 连接模式和监听模式

Ncat 主要都是以这两种模式运行，连接模式和监听模式，其他模式(例如http代理服务器)是这两种模式的特殊情况

连接模式------>充当客服端     <hostname>(主机名或IP地址，必须的)和<post> （十进制端口号，默认31337）要连接的内容。

监听模式------>充当服务端 <hostname>和<port> （如果省略，默认：31337）服务器绑定地址，都可选，默认监听ipv4,ipv6的所有可能地址.



## 协议选项

- `-4` （仅IPv4）

  强制仅使用IPv4。

- `-6` （仅IPv6）

  强制仅使用IPv6。

- `-U`， `--unixsock`（使用Unix域套接字）

  使用Unix域套接字而不是网络套接字。此选项可单独用于流套接字，或与`--udp`数据报套接字结合使用。`-U`模式的描述在 [“ Unix域套接字”一节中](https://nmap.org/book/ncat-man-unixsock.html)。

- `-u`， `--udp`（使用UDP）

  使用UDP进行连接（默认为TCP）。

- `--sctp` （使用SCTP）

  使用SCTP进行连接（默认为TCP）。SCTP支持以TCP兼容模式实现。

- `--vsock` （使用AF_VSOCK套接字）

  使用AF_VSOCK套接字而不是默认的TCP套接字（仅Linux）。此选项可以单独用于流套接字，也可以与`--udp`数据报套接字结合使用。`--vsock`模式的描述在 [“ AF_VSOCK套接字”一节中](https://nmap.org/book/ncat-man-vsock.html)。

## 连接模式选项

- `-g *`<hop1>`*[,*`<hop2>`*,...]` （源路由选择）

- `-G *`<ptr>`*` （设置源路由指针）

  设置IPv4源路由“指针”以用于`-g`。该参数必须是4的倍数且不超过28。并非所有操作系统都支持将此指针设置为除4以外的任何值。

- `-p *`<port>`*`， （指定源端口） `--source-port *`<port>`*`

  设置要绑定的Ncat的端口号。

- `-s *`<host>`*`， （指定源地址） `--source *`<host>`*`

  设置要绑定到的Ncat的地址。



## 监听模式选项

有关限制可连接到正在侦听的Ncat进程的主机的信息

- `-l`， `--listen`（收听连接）

  监听连接，而不是连接到远程计算机

- `-m *`<numconns>`*`， （指定最大连接数） `--max-conns *`<numconns>`*`

  Ncat实例接受的最大同时连接数。默认值为100（在Windows上为60）。

- `-k`， `--keep-open`（接受多个连接）

  通常，侦听服务器仅接受一个连接，然后在连接关闭时退出。此选项使它可以接受多个同时连接，并在它们全部关闭后等待更多连接。必须与结合使用 `--listen`。在这种模式下，Ncat无法知道其网络输入何时结束，因此它将一直运行直到被中断。这也意味着它将永远不会关闭其输出流，因此从Ncat读取并寻找文件结尾的任何程序也将挂起。

- `--broker` （连接代理）

  允许多方连接到集中式Ncat服务器并相互通信。Ncat可以代理位于NAT后面或无法直接连接的系统之间的通信。此选项与结合使用`--listen`，这会使`--listen`端口启用代理模式。

- `--chat`（临时“聊天服务器”）

  该`--chat`选项启用聊天模式，旨在用于多个用户之间的文本交换。在聊天模式下，连接代理处于打开状态。Ncat在将接收到的每个消息中继到其他连接之前，先给每个消息添加一个ID作为前缀。该ID对于每个连接的客户端都是唯一的。这有助于区分谁发送了什么。此外，转义了诸如控制字符之类的非打印字符，以防止它们损坏终端。

## SSL选项

- `--ssl` （使用SSL）

  在连接模式下，此选项透明地与SSL服务器协商SSL会话，以安全地加密连接。与启用SSL的HTTP服务器等通信时特别方便。在服务器模式下，此选项侦听传入的SSL连接，而不侦听普通的未隧道流量。在UDP连接模式下，此选项启用数据报TLS（DTLS）。服务器模式不支持此功能。

- `--ssl-verify` （验证服务器证书）

  在客户端模式下，除了需要验证服务器证书外，其他方面与之`--ssl-verify`类似 `--ssl`。Ncat在文件中附带了一组默认的受信任证书 `ca-bundle.crt`。 某些操作系统提供了受信任证书的默认列表。如果可用，也将使用它们。用于 `--ssl-trustfile`给出自定义列表。使用 `-v`一次或多次以获取有关验证失败的详细信息。Ncat不会检查已吊销的证书。此选项在服务器模式下无效。

- `--ssl-cert *`<certfile.pem>`*` （指定SSL证书）

  此选项提供用于对服务器（在侦听模式下）或客户端（在连接模式下）进行身份验证的PEM编码的证书文件的位置。与结合使用`--ssl-key`。

- `--ssl-key *`<keyfile.pem>`*` （指定SSL私钥）

  此选项提供了与PEM编码的私钥文件的位置，该文件与以命名的证书一起使用 `--ssl-cert`。

- `--ssl-trustfile *`<cert.pem>`*` （列出受信任的证书）

  此选项设置用于证书验证的受信任证书列表。除非与结合使用，否则它没有任何作用`--ssl-verify`。此选项的参数是PEM的名称 包含受信任证书的文件。通常，该文件将包含证书颁发机构的证书，尽管它也可能直接包含服务器证书。使用此选项时，Ncat不使用其默认证书。

- `--ssl-ciphers *`<cipherlist>`*` （指定SSL密码套件）

  此选项设置Ncat在连接到服务器或从客户端接受SSL连接时将使用的密码套件列表。该语法在OpenSSL ciphers（1）手册页中进行了描述，默认为 `ALL:!aNULL:!eNULL:!LOW:!EXP:!RC4:!MD5:@STRENGTH`

- `--ssl-servername *`<name>`*` （请求不同的服务器名称）

  在客户端模式下，此选项设置TLS SNI（服务器名称指示）扩展名，该扩展名告诉服务器Ncat正在联系的逻辑服务器的名称。当目标服务器在单个基础网络地址上托管多个虚拟服务器时，这一点很重要。如果未提供该选项，则将使用目标服务器主机名填充TLS SNI扩展名。

- `--ssl-alpn *`<ALPN list>`*` （指定ALPN协议列表）

  此选项使您可以指定一个逗号分隔的协议列表，以通过应用程序层协议协商（ALPN）TLS扩展发送。并非所有版本的OpenSSL都支持。





## 代理选项

- `--proxy *`<host>`*[:*`<port>`*]` （指定代理地址）

  使用所指定的协议，通过*`<host>`*：请求代理。*`<port>`*`--proxy-type`如果未指定端口，则使用代理协议的知名端口（对于SOCKS为1080，对于HTTP为3128）。当使用IP地址而不是主机名指定IPv6 HTTP代理服务器时，必须使用方括号表示法（例如[2001：db8 :: 1]：8080）将端口与IPv6地址分开。如果代理需要身份验证，请使用`--proxy-auth`。

- `--proxy-type *`<proto>`*` （指定代理协议）

  在连接模式下，此选项请求协议*`<proto>`* 通过所指定的代理主机进行连接`--proxy`。在侦听模式下，此选项使Ncat使用指定的协议充当代理服务器。连接模式下当前可用的协议为`http` （CONNECT），`socks4`（SOCKSv4）和 `socks5`（SOCKSv5）。当前唯一支持的服务器是`http`。如果不使用此选项，则默认协议为`http`。

- `--proxy-auth *`<user>`*[:*`<pass>`*]` （指定代理凭据）

  在连接模式下，提供将用于连接到代理服务器的凭据。在侦听模式下，提供连接客户端所需的凭据。与`--proxy-type http`或一起 使用时 `--proxy-type socks5`，表单应为username：password。对于 `--proxy-type socks4`，它只能是一个用户名。通过设置环境变量，这些凭据也可以传递到Ncat上 `NCAT_PROXY_AUTH`，这降低了在进程日志中捕获凭据的风险。（选项`--proxy-auth`优先。）

- `--proxy-dns *`<type>`*` （指定在何处解析代理目标）

  在连接模式下，它可以控制代理目标主机名是由远程代理服务器解析还是由Ncat本身在本地解析。可能的值为*`<type>`*：`local`-主机名在Ncat主机上本地解析。如果无法解析主机名，则Ncat退出并出现错误。`remote`-主机名直接传递到远程代理服务器上。这是默认行为。`both`-首先在Ncat主机上尝试解析主机名。无法解析的主机名被传递到远程代理服务器上。`none`-主机名解析已完全禁用。只能将文字IPv4或IPv6地址用作代理目标。本地主机名解析通常遵循由options`-4`或指定的IP版本`-6`，但SOCKS4除外，后者与IPv6不兼容。



## 命令执行选项

- `-e *`<command>`*`， （执行命令） `--exec *`<command>`*`

  建立连接后执行指定的命令。该命令必须指定为完整路径名。来自远程客户端的所有输入都将通过套接字发送到应用程序，并将响应通过套接字发送回远程客户端，从而使您的命令行应用程序通过套接字进行交互。与结合使用`--keep-open`，Ncat将处理与您的指定端口/应用程序的多个同时连接，如inetd。Ncat仅接受由该`-m`选项控制的最大可定义同时连接数。默认情况下，它设置为100（在Windows上为60）。

- `-c *`<command>`*`， （通过sh执行命令） `--sh-exec *`<command>`*`

  与相同`-e`，只是它尝试通过执行命令`/bin/sh`。这意味着您不必为命令指定完整路径，并且可以使用诸如环境变量之类的shell设施。

- `--lua-exec *`<file>`*` （执行.lua脚本）

  建立连接后，使用内置解释器将指定文件作为Lua脚本运行。脚本的标准输入和标准输出都被重定向到连接数据流。

所有exec选项都将以下变量添加到孩子的环境中：

1. `NCAT_REMOTE_ADDR`， `NCAT_REMOTE_PORT`

  远程主机的IP地址和端口号。在连接模式下，它是目标的地址；在监听模式下，这是客户的地址。

2. `NCAT_LOCAL_ADDR`， `NCAT_LOCAL_PORT`

  连接本地端的IP地址和端口号。

3. `NCAT_PROTO`

  使用的协议：之一`TCP`，`UDP`和`SCTP`。

## 访问控制选项

- `--allow *`<host>`*[,*`<host>`*,...]` （允许连接）

  指定的主机列表将是唯一允许连接到Ncat进程的主机。所有其他连接尝试将被断开。`--allow`和之间如果发生冲突 `--deny`，请 `--allow`以优先。主机规范遵循Nmap使用的相同语法。

- `--allowfile *`<file>`*` （允许来自文件的连接）

  它具有与相同的功能`--allow`，除了允许的主机以换行符分隔的允许文件（而不是直接在命令行中）提供。

- `--deny *`<host>`*[,*`<host>`*,...]` （拒绝连接）

  向Ncat发出不允许连接到正在侦听的Ncat进程的主机列表。如果指定的主机尝试连接，则其会话将以静默方式终止。`--allow`和之间如果发生冲突 `--deny`，请 `--allow`以优先。主机规范遵循Nmap使用的相同语法。

- `--denyfile *`<file>`*` （来自文件的拒绝连接）

  此功能与相同`--deny`，不同之处在于，排除的主机以换行符分隔的拒绝文件提供，而不是直接在命令行中提供。

## 定时选项

这些选项接受一个`time`参数。这是在几秒钟缺省情况下，虽然可以追加`ms`，`s`，`m`，或`h`到指定毫秒，秒，分钟或小时值。

- `-d *`<time>`*`， （指定线路延迟） `--delay *`<time>`*`

  设置发送线路的延迟间隔。这有效地限制了Ncat在指定时间段内发送的行数。这对于低带宽站点可能有用，或者具有其他用途，例如应付烦人的**iptables --limit**选项。

- `-i *`<time>`*`， （指定空闲超时） `--idle-timeout *`<time>`*`

  为空闲连接设置一个固定的超时。如果达到空闲超时，则连接终止。

- `-w *`<time>`*`， （指定连接超时） `--wait *`<time>`*`

  为连接尝试设置固定的超时。

## 输出选项

- `-o *`<file>`*`， （保存会话数据） `--output *`<file>`*`

  将会话数据转储到文件

- `-x *`<file>`*`， （以十六进制保存会话数据） `--hex-dump *`<file>`*`

  将会话数据以十六进制形式转储到文件中。

- `--append-output` （追加输出）

  问题NCAT与`--append-ouput`一起 `-o`和/或`-x`它会追加导致输出，而不是截断指定的输出文件。

- `-v`， `--verbose`（冗长）

  发行Ncat，`-v`它将很冗长，并显示各种有用的基于连接的信息。多次使用（`-vv`，`-vvv`...）以获得更多的详细信息。



## 其他选项

- `-C`， `--crlf`（将CRLF用作EOL）

  此选项告诉Ncat转换LF 行尾到CRLF 从标准输入中获取输入时。 这对于使用CRLF作为行尾的许多常见纯文本协议之一直接从终端与某些严格的服务器进行通信很有用。

- `-h`， `--help`（帮助屏幕）

  显示带有常用选项和参数的简短帮助屏幕，然后退出。

- `--recv-only` （仅接收数据）

  如果通过此选项，则Ncat将仅接收数据，并且不会尝试发送任何内容。

- `--send-only` （仅发送数据）

  如果通过此选项，则Ncat将仅发送数据，并且将忽略接收到的任何内容。此选项还会使Ncat关闭网络连接，并在标准输入上收到EOF后终止。

- `--no-shutdown` （请勿关闭到半双工模式）

  如果通过此选项，Ncat在stdin上看到EOF后将不会在套接字上调用关闭。提供此功能是为了与OpenBSD netcat向后兼容，当使用'-d'选项执行该操作时，它将表现出这种行为。

- `-n`， `--nodns`（不解析主机名）

  完全禁用所有Ncat选项（例如目标，源地址，源路由跃点和代理）的主机名解析。所有地址必须以数字形式指定。（请注意，代理目标的解析是通过option单独控制的`--proxy-dns`。）

- `-t`， `--telnet`（回答Telnet协商）

  处理DO / DONT WILL / WONT Telnet协商。这使得可以使用Ncat编写Telnet会话脚本。

- `--version` （显示版本）

  显示Ncat版本号并退出。

## Unix域套接字

该`-U`选项（与相同`--unixsock`）使Ncat使用Unix域套接字而不是网络套接字。Unix域套接字作为文件系统中的条目存在。您必须提供套接字的名称才能连接或监听。例如，要建立连接，

**ncat -U〜/ unixsock**

要在套接字上侦听：

**ncat -l -U〜/ unixsock**

侦听模式将创建套接字（如果不存在）。程序结束后，套接字将继续存在。

流和数据报域套接字都受支持。单独 `-U`用于流套接字，或与`--udp`数据报套接字结合使用。数据报套接字需要连接源套接字。默认情况下，将根据需要创建带有随机文件名的源套接字，并在程序结束时将其删除。使用 `--source`同一个路径使用源插座具有特定名称。

## AF_VSOCK sockets

该`--vsock`选项使Ncat使用AF_VSOCK套接字而不是网络套接字。必须提供CID而不是主机名或IP地址。例如，要建立与主机的连接，

**ncat --vsock 2 1234**

要在套接字上侦听：

**ncat -l --vsock 1234**

流域和数据报域套接字都受支持，但是套接字类型的可用性取决于管理程序。单独 `--vsock`用于流套接字，或与`--udp`数据报套接字结合使用。



## 退出码

退出代码反映连接是否成功建立和完成。

0表示没有错误。

1表示存在某种网络错误，例如“连接被拒绝”或“连接重置”。

2保留用于所有其他错误，例如无效的选项或不存在的文件。



# 案列

本机多窗口测试

**实现连接模式和监听模式**

```
#作为客服端连接本机9090端口
ncat localhost 9090

#侦听TCP端口8080上的连接。
ncat -l 9090
```

**将本地计算机上的TCP端口9090重定向到端口909上的主机。**

```
ncat --sh-exec "ncat localhost 9091" -l 9090 --keep-open
```

**绑定到TCP端口9090并附加/bin/bash ，以便其他机器自由访问。**

```
ncat --exec “/bin/bash” -l 9090 --keep-open

这样连接到这台机器的客服端可以通过命令来访问服务器，这样肯定是不安全的，黑客都喜欢这样搞
```



```
将程序绑定到TCP端口8081，限制对本地网络上主机的访问，并将同时连接的最大数量限制为3。
ncat --exec “/bin/bash” --max-conns 3 --allow 192.168.0.0/24 -l 8081 --keep-open

通过端口1080上的SOCKS4服务器连接到smtphost：25。
ncat --proxy socks4host --proxy-type socks4 --proxy-auth joe smtphost 25

通过端口1080上的SOCKS5服务器连接到smtphost：25。
ncat --proxy socks5host --proxy-type socks5 --proxy-auth joe：secret smtphost 25

在本地主机端口8888上创建HTTP代理服务器。
ncat -l-代理类型的HTTP本地主机8888

通过TCP端口9899将文件从host2（客户端）发送到host1（服务器）。
HOST1 $ ncat -l 9899>输出文件

HOST2 $ ncat HOST1 9899 <输入文件

向另一个方向传输，将Ncat变成“一个文件”服务器。
HOST1 $ ncat -l 9899 <输入文件

HOST2 $ ncat HOST1 9899>输出文件
```



# 总结

ncat 主要用来模拟和调试网络程序，它的监听模式相当于服务端，连接模式相当于客户端，通过组合他的命令选项，能实现重定向，ssl，代理，定时，访问控制和一些更加高级的功能。
