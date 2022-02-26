# 自动化测试必须掌握的 HTTP 知识



## 1. 请求

请求首行 （请求方法 url）

请求头

请求体

## 2. 相应

相应首行（状态码）

相应头

相应体

## 3. postman 中处理 HTTP 接口

## 4. Python 中处理 HTTP 接口

# 5. HTTP 就是些约定的格式？

# 6. socket 通过 TCP 组装 HTTP 消息

## 7. 服务端如何处理 HTTP 请求

## 8. 客户端如何组装 HTTP 请求

如果我们需要接收完整的服务器响应数据，需要修改代码：建立连接后，首先需要准备请求数据，发送请求。

```
HOST = 'httpbin.org'
PORT = 80

with socket.socket() as s:
    s.connect((HOST, PORT))
    s.settimeout(1)
    # 发送请求
    s.sendall(b"abc")
    response = b''
    while True:
        try:
            data = s.recv(1024)
        except ConnectionResetError as e:
            break
        except socket.timeout as e:
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







