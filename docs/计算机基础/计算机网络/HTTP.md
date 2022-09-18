## HTTP基础

### HTTP发展历程

![image-20220823213940725](https://picture.lingzero.cn/img/202208232200605.png)

HTTP/1.0每请求一个文档就要有两倍RTT的开销。另一种开销就是万维网客户和服务器每一次建立新的TCP连接都要分配缓存和变量。HTTP/1.1协议较好地解决了这个问题，它使用了持续连接

#### HTTP2.0



### DNS

![image-20220823215538924](https://picture.lingzero.cn/img/202208232159090.png)

### URI 和 URL

####  URI 的格式

![image-20220823215425003](https://picture.lingzero.cn/img/202208232159044.png)

### HTTP报文

![image-20220823214045544](https://picture.lingzero.cn/img/202208232158137.png)

请求报文

![image-20220823215839335](https://picture.lingzero.cn/img/202208232158138.png)

响应报文

![image-20220823215922098](https://picture.lingzero.cn/img/202208232159237.png)

### HTTP请求方法

#### 1.GET：**获取资源**

GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务
器端解析后返回响应内容。

![image-20220823220749846](https://picture.lingzero.cn/img/202208232229671.png)

#### 2.POST：**传输资源**

![image-20220823221119066](https://picture.lingzero.cn/img/202208232229672.png)

#### 3.PUT：**传输文件**

   配合 Web 应用程序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。

   ![image-20220823221342392](https://picture.lingzero.cn/img/202208232229673.png)

#### 4.HEAD：**获得报文首部**

​    确认URI 的有效性及资源更新的日期时间等

  ![image-20220823221538338](https://picture.lingzero.cn/img/202208232229674.png)

#### 5.DELETE：**删除文件**

与 PUT 相反的方法

![image-20220823221629350](https://picture.lingzero.cn/img/202208232229675.png)

#### 6.OPTIONS：**询问支持的方法**

![image-20220823221739302](https://picture.lingzero.cn/img/202208232229676.png)

#### 7.TRACE：**追踪路径**

客户端通过 TRACE 方法可以查询发送出去的请求是怎样被加工修改 / 篡改的。
![image-20220823221829492](https://picture.lingzero.cn/img/202208232229677.png)

#### 8.CONNECT：**要求用隧道协议连接代理**

CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。主要使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。
方法格式：
CONNECT  代理服务器名:端口号  HTTP版本

![image-20220823222127975](https://picture.lingzero.cn/img/202208232229678.png)

HTTP/1.0 和 HTTP/1.1 支持的方法

![image-20220823222224096](https://picture.lingzero.cn/img/202208232229679.png)

### 持久连接与管线化

#### 持久连接

只要任意一端没有明确提出断开连接，则保持 TCP 连接状态

![image-20220823222636985](https://picture.lingzero.cn/img/202208232229680.png)

#### 管线化

同时并行发送多个请求，而不需要一个接一个地等待响应了。

![image-20220823222544961](https://picture.lingzero.cn/img/202208232229681.png)

### Cookie的状态管理

![image-20220823222821748](https://picture.lingzero.cn/img/202208232229682.png)

![image-20220823222920053](https://picture.lingzero.cn/img/202208232229683.png)

![image-20220823222935637](https://picture.lingzero.cn/img/202208232229684.png)

### 状态码

![image-20220823224029166](https://picture.lingzero.cn/img/202208262109330.png)

#### 2XX成功

![image-20220823224041574](https://picture.lingzero.cn/img/202208262109331.png)

![image-20220823224124257](https://picture.lingzero.cn/img/202208262109332.png)

204 PUT方法

![image-20220823224155435](https://picture.lingzero.cn/img/202208262109333.png)

#### 3XX 重定向

![image-20220823224822186](https://picture.lingzero.cn/img/202208262109334.png)

![image-20220823224451406](https://picture.lingzero.cn/img/202208262109335.png)

![image-20220823224507937](https://picture.lingzero.cn/img/202208262109336.png)

![image-20220823224614822](https://picture.lingzero.cn/img/202208262109337.png)

![image-20220823224647583](https://picture.lingzero.cn/img/202208262109338.png)

#### 4xx 客户端错误

![image-20220823224933336](https://picture.lingzero.cn/img/202208262109339.png)

![image-20220823224949165](https://picture.lingzero.cn/img/202208262109340.png)

![image-20220823225005000](https://picture.lingzero.cn/img/202208262109341.png)

![image-20220823225111978](https://picture.lingzero.cn/img/202208262109342.png)

#### 5xx 服务器错误

![image-20220823225213734](https://picture.lingzero.cn/img/202208262109343.png)

- **502 Bad Gateway** ：我们的网关将请求转发到服务端，但是服务端返回的却是一个错误的响应。

![image-20220823225233790](https://picture.lingzero.cn/img/202208262109344.png)

![image-20220823225338717](https://picture.lingzero.cn/img/202208262109345.png)

## 网络安全

### 对称式加密

所谓对称密钥密码体制，即加密密钥与解密密钥是使用相同的密码体制。

常见的对称加密算法有: DES、3DES、Blowfish、IDEA、RC4、RC5、RC6 和 AES 

![image-20220826204118629](https://picture.lingzero.cn/img/202208262109346.png)

### 公钥密码体制

![image-20220826210025753](https://picture.lingzero.cn/img/202208262109347.png)

### 数字签名

![image-20220826213702953](https://picture.lingzero.cn/img/202209190022874.png)

![image-20220826210248215](https://picture.lingzero.cn/img/202208262109348.png)

![image-20220826210326691](https://picture.lingzero.cn/img/202208262109349.png)

### 密码散列

![image-20220826210406518](https://picture.lingzero.cn/img/202208262109350.png)

#### md5和sha-1

MD5算法的大致过程如下：
（1） 先把任意长的报文按模264计算其余数（64位），追加在报文的后面。
（2） 在报文和余数之间填充1-512位，使得填充后的总长度是512的整数倍。填充的首 位是1,后面都是0。
（3） 把追加和填充后的报文分割为一个个512位的数据块，每个512位的报文数据再分成4个128位的数据块依次送到不同的散列函数进行4轮计算。每一轮又都按32位的小数据块进行复杂的运算。一直到最后计算出MD5报文摘要代码（128位）。

SHA和MD5相似，但 
码长为160位（比MD5的128位多了 25%）

#### 报文鉴别码

![image-20220826210903783](https://picture.lingzero.cn/img/202208262109351.png)

# HTTP vs HTTPS

![image-20220826211454718](https://picture.lingzero.cn/img/202208262120073.png)

### SSL建立安全通信线路，

### 对http传输的内容加密

![image-20220826212622507](https://picture.lingzero.cn/img/202208262126787.png)

### HTTPS采用混合加密机制

![image-20220826213206006](https://picture.lingzero.cn/img/202209190022875.png)

### 服务器端的公开密钥证书（服务器证书）建立 HTTPS 通信的整个过程。

![image-20220826213337733](https://picture.lingzero.cn/img/202209190022876.png)

![img](https://picture.lingzero.cn/img/202208262144454.jpeg)

1. 首先客户端通过URL访问服务器建立SSL连接。
2. 服务端收到客户端请求后，会将网站支持的证书信息（证书中包含公钥）传送一份给客户端。
3. 客户端的服务器开始协商SSL连接的安全等级，也就是信息加密的等级。
4. 客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。
5. 服务器利用自己的私钥解密出会话密钥。
6. 服务器利用会话密钥加密与客户端之间的通信。
