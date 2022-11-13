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
* WWW-Authenticate: Basic realm=""
  ```
  通常使用在401 error page的response header
  浏览器不会立马解析返回的401 error response，而是弹出一个验证窗口
  如果点击验证窗口的canel，浏览器则解析返回401 error response
  如果输入了验证信息，浏览器会弃掉已经返回的401 error response, 然后把验证信息组装成Authorization header加入到请求中而再次向服务器请求一次
  ```
  
