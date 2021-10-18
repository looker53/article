# **socket 网络编程**

socket 是一套操作系统提供的网络通讯协议，在国内习惯称为套接字。 一台计算机的 IP 地址和端口号会组成套接字地址（socket address），加上远程服务器的套接字地址，以及使用的网络协议（比如TCP，UDP） ，五个要素共同组成一个套接字对（socket pairs)，之后就可以交换数据。在同一台计算机上，TCP协议与UDP协议可以同时使用相同的port而互不干扰。

![https://gitee.com/looker53/pics/raw/master/img//20210603202336.png](https://gitee.com/looker53/pics/raw/master/img//20210603202336.png)

编程语言会对计算机上的套接字协议进行封装，从而实现编程语言的网络请求操作。因为这个封装偏底层，所以在平时的编程过程中很少直接接触到，我们直接接触的，是那些基于 socket 进行了更高层级封装的网络请求库。这些库用起来更加简单，但是基本上都是基于 socket 实现的。理解 socket 对深入研究这些网络请求库的原理有帮助，当出现问题时，调试起来会更有底气。

### **服务器**

在 python 语言中，socket 库是底层的协议库。下面代码启动一个服务器：

```
import socket

HOST = '127.0.0.1'
PORT = 9999

with socket.socket() as sock:
    sock.bind((HOST, PORT))
    sock.listen()
    print(f"start server {HOST}:{PORT}")
    client, address = sock.accept()
    print(f"connect from {address}")
```

- socket.socket() 初始化一个 socket 对象；
- bind 绑定域名和端口，之后就可以监听连接这个套接字地址的请求；
- listen 监听这个套接字地址；
- accept 表示接受其他链接并返回一个客户端 socket 和地址

现在你可以在浏览器或者其他的网络访问工具上访问 127.0.0.1:9999 这个套接字地址，当有一个请求链接后，则会打印客户端地址。但是这个服务只能接受一个连接，然后就会自动退出。如果需要接受多个连接，需要写一个 while 循环，一旦连接建立，则可以通过 sendall 往客户端发送数据，也可以通过 recv 接收数据。

```
...
sock.listen()
print(f"start server {HOST}:{PORT}")
while True:
    client, address = sock.accept()
    print(f"connect from {address}")
    client.sendall(b"hello")
```

### **客户端**

客户端代码也是创建一个 socket ，但是不需要绑定地址，而是主动连接远端服务器地址。一旦服务器接受连接，就可以相互发送数据了，你可以通过 sendall 一次性发送数据，也可以通过 send 部分发送。 同时，两端都可以通过 recv 接受数据。

```
import socket

HOST = '127.0.0.1'
PORT = 9990

with socket.socket() as sock:
    sock.connect((HOST, PORT))
    print(f"connecting server {HOST}:{PORT}")
    sock.sendall(b"hello")
    data = sock.recv(1024)
print(f"服务器返回结果：{data.decode()}")
```

现在每运行一次客户端的程序，服务器都能返回提前准备好的数据。但是同一个客户端不能多次发送数据，就算通过 while 循环频繁发出，服务器也不能处理。 因为服务器每次连接只处理了一次，就会等待新的连接。

### **长链接**

如果不想直接关闭客户端程序，而是进行双向多次通讯，则可以稍微修改前后端代码。

服务端代码：

服务端现在只在连接之后返回一次 response, 之后这次链接就不会有新的响应了。如果需要多次通讯，通过在 client 的上下文中加入 while 循环，如果客户端有新的数据发送过来，可以多次发送。

```
import socket

HOST = '127.0.0.1'
PORT = 9998

with socket.socket() as s:
    s.bind((HOST,PORT))
    s.listen()
    while True:
        client, address = s.accept()
        with client:
            while True:
                data = client.recv(1024)
                if not data:
                    break
                print("收到数据",data.decode())
                resp = input("回复：")
                client.sendall(resp.encode())
```

客户端代码也加入 while 循环，接收服务端返回的数据。

```
import socket

HOST = '127.0.0.1'
PORT = 9998

with socket.socket() as s:
    s.connect((HOST, PORT))
    s.sendall("正在连接主机".encode())
    while True:
        data = s.recv(1024)
        if not data:
            break
        print("收到数据",data.decode())
        content = input("输入信息：")
        s.sendall(content.encode())
```

效果：

![https://gitee.com/looker53/pics/raw/master/img//sockect_chat.gif](https://gitee.com/looker53/pics/raw/master/img//sockect_chat.gif)

TODO: 非阻塞

### **HTTP**

现在你可以通过 client.sendall() 一次性发送数据给客户端。当你使用了特定的协议，发送的数据就需要符合某个规则。假设你在浏览器当中往服务器发送了一个 http 请求，服务器只简单的响应 b"hello world" 就不符合 http 协议的规范，这个响应结果并不能在浏览器中直接看到，如果通过浏览器的 f12 查看，会发现没有响应结果。

```
sock.listen()
print(f"start server {HOST}:{PORT}")
while True:
    client, address = sock.accept()
    print(f"connect from {address}")
    client.sendall(b"hello world")
```

这是因为发送 http 响应信息需要符合 http 规范，带上响应行和相关的响应头，然后再添加响应体。

```
HTTP/1.1 200 OK\r\n
Content-type: text/html;charset=utf-8\r\n
Content-length: 11\r\n
\r\n
hello world
```

- 每行数据以\r\n结尾，而不是\n
- 第一行是响应首行，包好协议版本和状态码
- 第2/3 行是响应头，Content-type 和 Content-length 都要提供，才会正确响应。
- 响应头结束后，空一行再添加响应体。响应体的长度由 Content-length 指定。

现在你搭建了一个非常简单的 http 服务器，可以使用火狐或者chrome浏览器访问试试：

```
import socket

HOST = '127.0.0.1'
PORT = 9990
RESPONSE = b'''\
HTTP/1.1 200 OK
Content-type: text/html;charset=utf-8
Content-length: 11

hello world
'''.replace(b'\n', b'\r\n')

with socket.socket() as sock:
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind((HOST, PORT))
    sock.listen()
    print(f"start server {HOST}:{PORT}")
    while True:
        client, address = sock.accept()
        print(f"connect from {address}")
        with client:
            client.sendall(RESPONSE)
```

效果：

![https://gitee.com/looker53/pics/raw/master/img//sockect_http_server.gif](https://gitee.com/looker53/pics/raw/master/img//sockect_http_server.gif)

### **UDP**

UDP 不需要连接，这是和 TCP 相比最大的特征，一个套接字地址，只需要知道另一个套接字地址，就可以直接尝试发送数据，所以它不能保证数据一定能被送达。

具体来说，服务端不需要通过 listen 监听数据，而是接收所有被传到套接字地址的数据。也不需要通过 accept 确认连接。

```
import socket

HOST = '127.0.0.1'
PORT = 5005

with socket.socket(type=socket.SOCK_DGRAM) as s:
    s.bind((HOST, PORT))
    while True:
        data, address = s.recvfrom(1024)
        print(f"接收来自{address}数据：{data.decode()}")
        s.sendto(b'over', address)
```

- 创建 udp 的套接字需要指定 type 为 SOCK_DGRAM
- 服务器绑定地址和端口，但是不需要 listen 监听
- 不需要 accept 确认连接，而是通过 recvfrom 读取所有的数据
- 发送数据时通过 sendto 指定发送给哪个地址。

对于客户端，也不需要通过 connect 连接服务器，得到 socket 后，就可以通过 sendto 发送数据。

```
with socket.socket(type=socket.SOCK_DGRAM) as s:
    for i in range(100):
        data = '发送数据' + str(i)
        s.sendto(data.encode(), (HOST, PORT))
        data, address = s.recvfrom(1024)
        print(f"服务器的返回数据{data.decode()}")
```

这里顺便总结一下 TCP 和 UDP：

- tcp 适用于需要稳定连接的场景，有句老话说的好： when in doubt, use tcp
- udp 适用于网络直播，游戏，流媒体，vpn 隧道等领域，传输不可靠，但是比较快。
- google 基于 udp 协议提出了 quic 协议被用于浏览器的网络传输，目前，该协议以被纳入 HTTP/3

---

### **HTTP 请求客户端**

在第一节我们写了一个很简单的客户端代码连接服务器：

```
import socket

HOST = '127.0.0.1'
PORT = 9990

with socket.socket() as sock:
    sock.connect((HOST, PORT))
    print(f"connecting server {HOST}:{PORT}")
    sock.sendall(b"hello")
    data = sock.recv(1024)
print(f"服务器返回结果：{data.decode()}")
```

如果我们需要接收完整的服务器响应数据，需要修改代码：建立连接后，首先需要准备请求数据，发送请求。

```
HOST = 'httpbin.org'
PORT = 80

with socket.socket() as s:
    s.connect((HOST, PORT))
    # 发送请求
    s.sendall(b"abc")
    response = b''
    while True:
        try:
            data = s.recv(1024)
        except ConnectionResetError as e:
            break
        if not data:
            break
        response += data
print(response.decode())
```

通过上面的代码访问，服务器返回的是 400 bad request，因为我们没有按照 http 规范发送请求数据。还记不记得我们之前讲过 HTTP 协议的响应格式:

```
HTTP/1.1 200 OK\r\n
Content-type: text/html;charset=utf-8\r\n
Content-length: 11\r\n
\r\n
hello world
```

其实请求的格式也是类似的，只需要修改一下请求行：

```
GET /get HTTP/1.1\r\n
Host: httpbin.org:8000\r\n
\r\n
```

正确的 http 发送方式是这样的：

```
req_content = b'GET /get HTTP/1.1\r\nHost: httpbin.org:8000\r\n\r\n'
s.sendall(req_content)
```

现在只需要把上面的代码封装一下。 首先创建一个连接类

```
class Connection:
    def __init__(self):
        self.socket = socket.socket()

    def connect(self, host, port):
        self.socket.connect((host, port))

    def send(self, method, url, headers=None, data=None, version=None):
        request = Request(method, url, headers, data, version)
        self.connect(request.host, request.port)
        self.socket.sendall(request.content)
        resp = Response(self.socket)
        self.socket.close()
        return resp

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        return self.close()

    def close(self):
        self.socket.close()
```

重点是 send 方法，通过 Request 对象，组装 http 请求格式，最终发出去的数据符合 http 规范。 然后通过 socket 发送出去，接下来通过 Response 对象解析原始数据。

对应的 Request 类代码：

```
class Request:
    def __init__(
            self,
            method,
            url,
            headers=None,
            data=None,
            version=None
    ):

        self.method = method
        self.full_url = url

        result = urlparse(self.full_url)
        if not result.scheme:
            raise ValueError('invalid schema, please check url like http://...')
        self.schema = result.scheme.upper()
        self.host, _, port = result.netloc.partition(':')
        self.port = int(port) if port else 80
        self.path = result.path
        self.query = result.query

        self.headers = headers or {}
        self.data = data
        self.version = version or '1.1'

    @property
    def content(self):
        req = ' '.join([self.method.upper(),
                        self.path,
                        self.schema + '/' + self.version]) + '\r\n'
        self.headers.setdefault('host', self.host + ':' + str(self.port))
        for header_name, header_value in self.headers.items():
            req += ': '.join([header_name.capitalize(), header_value])
            req += '\r\n'
        req += '\r\n'

        # TODO: data 有 bug, 传递格式不对
        if self.data:
            req += _json.dumps(self.data)
            req += '\n'

        return req.encode()
```

Response 代码：

```
class Response:

    def __init__(self, sock):
        self.socket = sock
        self.headers = dict()
        self.status_code = None
        self.text = None

        self.parse()

    def read(self):
        content = b''
        while True:
            try:
                data = self.socket.recv(1024)
            except ConnectionResetError:
                break
            content += data
            if len(data) < 1024:
                break
        return content

    def parse(self):
        raw = self.read().decode()
        resp_line, *headers, text = raw.split('\r\n')
        _, code, _ = resp_line.split(' ', maxsplit=2)
        self.status_code = int(code)

        for header in headers[:-1]:
            name, value = header.split(': ')
            self.headers[name] = value

        self.text = text.strip()

    def json(self):
        return _json.loads(self.text)

```

现在，访问一个 http 请求的操作就比较简单了。

```
with Connection() as client:
    resp = client.send('GET', 'http://httpbin.org/get')

print(resp.status_code)
print(resp.text)
print(resp.json())
```

对于常用的请求方法，直接写成函数访问：

```
def get(url, headers=None, **kwargs):
    with Connection() as s:
        return s.send('GET', url, headers, **kwargs)

def post(url, headers=None, data=None, **kwargs):
    with Connection() as s:
        return s.send('POST', url, headers, data, **kwargs)

resp = get(url)
print(resp.text)
```

### http服务器