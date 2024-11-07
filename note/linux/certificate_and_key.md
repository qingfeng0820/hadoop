## 证书类型
* `.pem`: 文件一种可阅读格式的文件，文件内容可能代表的是证书、可能代表私钥等
* `.oca.pem`: （中间）证书文件
* `.rca.pem`: （根部）证书文件
* `.cer`: 二进制格式的证书文件
* `.crt.pem`: PEM格式的证书文件，文本文件可阅读
  * content example
     ```
     -----BEGIN CERTIFICATE-----
     ...
     -----END CERTIFICATE-----
     ```
* `.key`: 其实就是一个pem格式只包含私玥的文件，`.key` 作为文件名只是作为一个明显的别名。
* `.key.pem`: PEM格式的密钥文件, pkcs#1格式的PEM密钥文件
  * content example:
    ```
    -----BEGIN RSA PRIVATE KEY-----
    ....
    -----END RSA PRIVATE KEY-----
    ```
* `.key.p8`: PEM格式的私钥文件，pkcs#8格式的PEM私钥文件
   * content example
     ```
     -----BEGIN PRIVATE KEY-----
     ...
     -----END PRIVATE KEY-----
     ```
* ` .pkcs12 .pfx .p12`:
  * pkcs即 RSA定义的 公玥密码学( Public-Key Cryptography Standards)标准
  * 有多个标准 pkcs12只是其一，是描述个人信息交换语法标准。
  * 有的文件直接使用其作为文件后缀名。这种文件包含公钥和私钥证书对，
  * 跟pem文件不同的是，它的内容是完全加密的
  * 可以把其转换成包含公玥和私玥的 .pem 文件。命令： `openssl pkcs12 -in file-to-convert.p12 -out converted-file.pem -nodes`
    * -nodes: This flag specifies that the private key should not be encrypted with a password in the PEM file. The private key will be stored in the PEM file in plaintext form.
*  `.der` 
  *  其实der不是一种文件格式。der 是ASN.1 众多编码方案中的一个，使用der编码方案编码的pem文件。
  * der 编码是使用二进制编码，一般pem文件使用的是base64编码
  * 所以完全可以把der编码的文件转换成pem文件，命令： `openssl x509 -inform der -in to-convert.der -out converted.pem`
  * 使用der编码的pem文件，后缀名可以为`.der`，也可以为格式`.cert .cer .crt`
* `.csr`: 证书请求文件.
  * 由 RFC 2986定义的PKCS10格式，包含部分/全部的请求证书的信息
    * 比如，主题, 机构，国家等，并且包含了请求证书的公玥
  * 这些被CA中心签名后返回一张证书。返回的证书是公钥证书（只包含公玥不含私钥）



## 生成自签名证书
* 生成私钥
  `openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:2048`
* 生成自签名证书
  `openssl req -new -x509 -key private.key -out certificate.pem -days 365`
* 同时生成私钥和自签名证书
  `openssl req -x509 -nodes -newkey rsa:2048 -keyout private.key -out certificate.pem -days 365`
* 检查已创建的证书
  `openssl x509 -text -noout -in certificate.pem`
* 将密钥和证书组合在 PKCS#12 (P12) 捆绑软件中
  `openssl pkcs12 -inkey key.pem -in certificate.pem -export -out certificate.p12`
* 验证 P12 文件
  `openssl pkcs12 -in certificate.p12 -noout -info`
* 生成Certificate Signing Request (CSR)
  `openssl req -new -key private.key -out csr.pem`
* Sign the CSR with a CA Key 签发证书
  `openssl x509 -req -in csr.pem -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out signed-cert.pem -days 365`
  * -CAcreateserial: Creates a serial file for the signed certificate.