# Http response headers
## Strict-Transport-Security
* Strict-Transport-Security:max-age=31536000; includeSubDomains 
* solve problem: 
  ```
  网站全站HTTPS后，如果用户手动敲入网站的HTTP地址，或者从其它地方点击了网站的HTTP链接，通常依赖于服务端301/302跳转才能使用HTTPS服务。
  而第一次的HTTP请求就有可能被劫持，导致请求无法到达服务器，从而构成HTTPS降级劫持。这个问题目前可以通过HSTS(HTTP Strict Transport Security，RFC6797)来解决。
  网站通过HTTP Strict Transport Security通知浏览器，这个网站禁止使用HTTP方式加载，浏览器应该自动把所有尝试使用HTTP的请求自动替换为HTTPS请求。
  ```
* max-age，单位是秒，用来告诉浏览器在指定时间内，这个网站必须通过HTTPS协议来访问。也就是对于这个网站的HTTP地址，浏览器需要先在本地替换为HTTPS之后再发送请求。
* includeSubDomains，可选参数，如果指定这个参数，表明这个网站所有子域名也必须通过HTTPS协议来访问
* preload，可选参数，一个浏览器内置的使用HTTPS的域名列表。

## WWW-Authenticate
* for http: WWW-Authenticate: Basic realm="Realm"
* for https: WWW-Authenticate: Basic realm="Security Realm"
  ```
  通常使用在401 error page的response header
  浏览器不会立马解析返回的401 error response，而是弹出一个验证窗口
  如果点击验证窗口的canel，浏览器则解析返回401 error response
  如果输入了验证信息，浏览器会弃掉已经返回的401 error response, 然后把验证信息组装成Authorization header加入到请求中而再次向服务器请求一次
  

  WWW-Authenticate must with at least one chanllenge:
  
  // Challenges specified in single header
  WWW-Authenticate: challenge1, ..., challengeN
  
  // Challenges specified in multiple headers
  WWW-Authenticate: challenge1
  ...
  WWW-Authenticate: challengeN
  
  // Possible challenge formats (scheme dependent)
  WWW-Authenticate: <auth-scheme>
  WWW-Authenticate: <auth-scheme> realm=<realm>
  WWW-Authenticate: <auth-scheme> token68
  WWW-Authenticate: <auth-scheme> auth-param1=token1, ..., auth-paramN=auth-paramN-token
  WWW-Authenticate: <auth-scheme> realm=<realm> token68
  WWW-Authenticate: <auth-scheme> realm=<realm> token68 auth-param1=auth-param1-token , ..., auth-paramN=auth-paramN-token
  WWW-Authenticate: <auth-scheme> realm=<realm> auth-param1=auth-param1-token, ..., auth-paramN=auth-paramN-token
  WWW-Authenticate: <auth-scheme> token68 auth-param1=auth-param1-token, ..., auth-paramN=auth-paramN-token
  
  <auth-scheme>
  The Authentication scheme. Some of the more common types are (case-insensitive): Basic, Digest, Negotiate and AWS4-HMAC-SHA256.
  
  realm=<realm> 
  A string describing a protected area. A realm allows a server to partition up the areas it protects 
  (if supported by a scheme that allows such partitioning), and informs users about which particular username/password are required. 
  If no realm is specified, clients often display a formatted hostname instead.
  
  After receiving the WWW-Authenticate header, a client will typically prompt the user for credentials, and then re-request the resource.
  This new request uses the Authorization header to supply the credentials to the server, encoded appropriately for the selected "challenge" authentication method.
  ("challenge" means `Basic realm="Realm"`)
  
  For example, Basic authentication allows for optional realm and charset keys
  WWW-Authenticate: Basic
  WWW-Authenticate: Basic realm=<realm>
  WWW-Authenticate: Basic realm=<realm>, charset="UTF-8"
  ```
  ```
  在成功认证之后，Basic认证会把Authorization认证信息缓存在浏览器中一段时间，之后每次请求接口时都会自动带上，
  所以直到用户关闭浏览器才会销毁认证信息，也就是说我们无法在服务端进行有效的注销。
  
  不过在请求注销时，前端也可以手动 在请求头配置一个错误的Authorization，
  或者在浏览器的命令行执行document.execuCommand("ClearAuthenticationCache")方法来清空认证信息，但该方式对Chrome浏览器无效。
  ```