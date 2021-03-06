# Web 服务器的部署地点

- 在公司里部署 Web 服务器

  - 服务器直接部署在公司网络上，并且可以从互联网直接访问。有两个缺点：

    - IP 地址不足。公司网络中的所有设备，包括服务器和客户端计算机，都分配各自的公有地址。
    - 安全问题。任何设置错误产生的安全漏洞都会暴露出来。

  - 通过防火墙隔离服务器和互联网

    防火墙屏蔽了不允许从外部访问的应用程序。

- 将 Web 服务器部署在数据中心

  服务器部署在数据中心可以获得很高的访问速度。



# 防火墙的结构和原理

- 如何设置包过滤的规则

  地址转换和包过滤中用于设置规则的字段

  | 头部类型           | 规则判断条件    | 含义                                                         |
  | ------------------ | --------------- | ------------------------------------------------------------ |
  | Mac 头部           | 发送方 MAC 地址 | 路由器在对包进行转发时会改写 MAC 地址，将转发目标路由器的 MAC 地址设置为接收方 MAC 地址，将自己的 MAC 地址设为发送方 MAC 地址。通过发送方 MAC 地址，可以知道上一个转发路由器的 MAC 地址 |
  | IP 头部            | 发送方 IP 地址  | 发送该包的原始设备的 IP 地址。如果要以发送设备来设置规则，需要使用这个字段 |
  | IP 头部            | 接收方 IP 地址  | 包的目的地 IP 地址，如果要以包的目的地来设置规则，需要使用这个字段 |
  | IP 头部            | 协议号          | TCP/IP 协议为每个协议分配一个编号，如果要以协议类型来设置规则，需要使用这个编号。主要的协议号包括 IP：0；ICMP：1；TCP：6；UDP：17；OSPF：89 |
  | IP 头部或 UDP 头部 | 发送方端口号    | 发送该包的程序对应的端口号。服务器程序对应的端口号是固定的，因此服务器返回的包的端口可以分辩是哪个程序发送的。不过客户端程序的端口号大多数是随机分配的，难以判断其来源，因此很少时候客户端发送的包的端口号来设置过滤规则。 |
  | IP 头部或 UDP 头部 | 接收方端口号    | 包的目的地程序对应的端口号。和发送方端口号一样，一般使用服务器的端口号来设置规则，很少使用客户端的端口号 |
  | IP 头部或 UDP 头部 | TCP 控制位 | TCP 协议的控制信息，主要用来控制连接操作<br />ACK：表示接收数据序号字段有效，一般用于通知发送方数据已经正确接收<br />PSH：表示发送方应用程序希望不等待发送缓冲区填充完毕，立即发送这个包<br />RST：强制断开连接，用于异常中断<br />SYN：开始通信时连接操作中发送的第一个包中，SYN 为 1，ACK 为 0。如果能够过滤这样的包，则后面的操作都无法继续，可以屏蔽整个访问<br />FIN：表示断开连接 |
  | IP 头部或 UDP 头部 | 分片            | 通过 IP 协议的分片功能拆分后的包，从第二个分片开始就会设置该字段 |
  | ICMP 消息（非头部）的内容 | ICMP 消息类型 | ICMP 消息用于通知包传输过程中产生的错误，或者用来确认通信对象的工作状态。 ICMP |

- 实现允许从互联网访问 Web 服务器，但禁止 Web 服务器访问互联网的防火墙。

  - 终点（接收方 IP 地址）为 Web 服务器 IP 地址且接收方端口号为 80 的包通过。
  - 起点（发送方 IP 地址）为 Web 服务器 IP 地址且接送方端口号为80的包通过。
  - 如果一个包是从 Web 服务器发往互联网并且 TCP 控制位中 SYN 位1且 ACK 为 0 不允许通过。



# 通过将请求平均分配给多台服务器来平衡负载

- 负载均衡的两种方法
  - 当访问服务器的时候，客户端先向 DNS 服务器查询服务器的 IP 地址。如果在 DNS 服务器中填写多个名称相同的记录，则每次查询时 DNS 服务器都会按顺序返回不同的 IP 地址。缺点：如果操作是跨网页的，这个操作将无法进行。
  - 使用负载均衡器分配访问。将域名对应的 IP 地址设置为负载均衡器的 IP 地址并注册到 DNS 服务器上。当客户端发起请求时，由负载均衡器来判断转发给哪台 Web 服务器。
    - 如果操作没有跨多个页面，则可以根据 Web 服务器的负载状况来进行判断。负载均衡器可以定期采集 Web 服务器的 CPU、内存使用率，并根据这些数据怕暖服务器的负载状况，也可以向 Web 服务器发送测试包，根据响应所需的时间来判断负载状况。或者事先根据服务器性能指数按比例发送分配请求。
    - 如果操作跨多个页面，必须将请求发送到同一台 Web 服务器上。可以通过 HTTP 头部字段中的信息来判断，例如 Cookie。



# 使用缓存服务器分担负载

- 如何使用缓存服务器

  - 除了使用多台功能相同的 Web 服务器分担负载之外，还可以将整个系统分成不同的服务器，如 Web 服务器、数据库服务器。
  - 缓存服务器是一台通过代理机制对数据进行缓存的服务器。待机介于 Web 服务器和客户端之间，具有对 Web 服务器访问进行中转功能。当进行中转时，他可以将 Web 服务器返回的数据保存在磁盘中，并可以代替 Web 服务器将磁盘中的数据返回给客户端。
  - 缓存服务器可以处理一部分请求。

- 缓存服务器通过更新时间管理内容

  - 缓存服务器代替 Web 服务器注册到 DNS 服务器中。
  - 客户端向缓存服务器发送 HTTP 请求消息。
  - 缓存服务器检查请求消息的呢绒，看看请求消息是否已经保存在缓存中。
  - 如果不存在缓存数据
    - 服务器会在 HTTP 头部字段中添加一个 Via 字段，表示这个消息经过缓存服务器转发，然后将消息转发给 Web 服务器。
    - 如果一台缓存服务器对应多台 Web 服务器时，缓存服务器根据请求消息的 URI 中的目录名来判断应该转发给哪一台服务器。
    - 缓存服务器会以客户端的身份向目标 Web 服务器发送请求消息。
    - 缓存服务器接收到消息后，会在响应消息中加上 Via 头部字段，并将消息发送给客户端。
    - 服务器黄酱响应消息保存到缓存中，并记录时间。
  - 如果存在缓存数据
    - 缓存服务器会添加一个 If-Modified-Since 头部字段并将请求转发给 Web 服务器，询问 Web 服务器用户请求的数据是否已经发生变化。
    - Web 服务器会根据 If-Modified-Since 的值域服务器上的页面数据的最后更新进行比较。
      - 如果在指定时间内没有发生变化，就返回一个表示没有变化的响应消息。
      - 如果发生了变化，Web 服务器会返回最新数据版本。然后缓存服务器加上 Via 字段发送给客户端，同时将数据保存在缓存中。

- 正向代理

  - 放在客户端一侧的缓存服务器称为正向代理。
  - 正向代理有两个作用，一个是缓存，另一个是实现防火墙。
  - 正向代理接收来自客户端的请求消息，然后再转发到互联网中。对于以前访问过的数据，可以直接从位于公司内网的代理服务器获得。
  - 设置了正向代理后，浏览器会忽略网址栏的内容，直接将所有请求发送给正向代理，并在请求的 URI 字段中填写完整的 http://… 网址。

- 反向代理

  将消息中的 URI 中的目录名与 Web 服务器进行关联，使得代理能够转发一般的不包含完整网址的请求消息。例子：缓存服务器。

- 透明代理

  特征：不需要像正向代理一样设置浏览器参数，也不需要在服务器上设置转发目标，可以将请求转发给任意 Web 服务器。

# 内容分发服务

- 利用内容分发服务分担负载
  - 将缓存服务器部署在 Web 服务器之前，可以降低 Web 服务器的负载，但无法减少网络流量。
  - 将缓存服务器部署在客户端，可以减少网络流量，但 Web 服务器运营者无法控制位于客户端的缓存服务器。
  - 将缓存服务器部署在互联网边缘，可以降低网络流量，而且服务器运营者可以控制缓存服务器。
- 如何找到最近的缓存服务器
  - 事先从缓存服务器不是地点的路由器收集路由信息，并将路由表集中到 DNS 服务器上。
  - DNS 服务器根据路由表查询从缓存服务器到 DNS 查询消息的发送方（客户端 DNS 服务器的路由信息）。
- 通过重定向服务器分配访问目标
  - 在 HTTP 规格中有一个称作 Location 的字段。当 Web 服务器数据转移到其他服务器时可以使用这个字段。
  - 首先需要将重定向服务器注册到 Web 服务器端的 DNS 服务器上。当客户将 HTTP 请求消息发送到注册服务器上时，重定向服务器将缓存服务器的地址放到 Location 字段中返回响应。
  - 优点：重定向的方法是根据客户端发送来的 HTTP 消息的发送方 IP 地址来估算距离，因此精度较高。距离的估算可以通过在客户端运行脚本测时间来计算出最近的缓存服务器。
- 缓存的更新方法会影响性能
  - 背景：将曾经访问过的数据保存下来，然后当再次访问时拿出来用，以提高反问操作的效率。这种方法对第一次访问是无效的，而且后面的每次访问都需要向原始服务器查询数据有没有发生变化。
  - 改善方法：当 Web 服务器在原始数据发生更新时，立即通知缓存服务器，使得缓存服务器上的数据一致保持最新状态。