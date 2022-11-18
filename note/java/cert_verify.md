# Certificate verification
## basic concept
### 对称加密算法 (symmetric key algorithms)
* 加密和解密使用的密钥是同一个密钥
* 算法比较块，但需要把密钥做好保密
### 非对称加密算法
* 加密和解密使用的密钥不同
* 公钥密码体制 (RSA)
  * 公钥, 私钥, 加密算法。
  * 用公钥加密的只有私钥解密，私钥加密的只有公钥可以解密。
  * 公钥和加密算法是公开的。
### 加密和签名
* 加密: 对某个信息或内容加密形成密文
* 签名
  * 用一个hash算法对信息做hash计算得到一个hash值(不可逆：无法有hash值还原信息)。
  * 这个hash值加密后就是签名
  * 把hash算法，签名和原信息一起发出，接收着可以用hash算法计算信息得到hash值和签名解密出来的hash值做比较，以防止信息被篡改

## 加密通信
### 简单模拟过程
1. "客户" -> "服务器": 你好
2. "服务器" -> "客户": 你好，我是服务器
3. "客户" -> "服务器": 向我证明是你服务器
4. "服务器" -> "客户": 你好，我是服务器 {你好，我是服务器}[私钥|RSA]
   * (约定 {} 表示加密后的内容，[ | ]表示用什么密钥和算法进去加密。)
5. "客户" -> "服务器": {我们后面用对称加密通信，这是对称加密算法和密钥}[公钥|RSA]
6. "服务器" -> "客户": {OK，收到！}[密钥|对称加密算法]
7. "客户" -> "服务器": {我的账号是aaa, 密码是123， 把我的余额信息发给我看看}[密钥|对称加密算法]
8. "服务器" -> "客户": {你的当前余额是100元}[密钥|对称加密算法]
   * RSA起到两个作用
     1. 私钥只有"服务器"拥有，可以用来判断对方是否是"服务器"
     2. 通过RSA做掩护，安全的与"服务器"协商好一个对称加密算法和密钥来保证后面通信过程内容的安全

### 数字证书
* 证书颁发机构 (issuer)
* 证书有效期
* 公钥 (public key)
* 证书所有者 (subject)
* 指纹算法: 计算整个证书的hash值用到的hash算法
* 指纹
  * 用来保证证书的完整性，确保证书没有被修改过
  * 用指纹算法对整个证书算出一个hash值作为指纹
* 签名算法： 生产签名所用的加密算法
* 签名
  * 证书颁发机构用签名算法和它自己的私钥对颁发的证书里面的指纹和指纹算法做加密，形成的密文就是签名
  * 客户端可以用证书颁发机构本身的证书里的公钥加密服务器证书的指纹和指纹算法，然后用指纹算法对服务器的证书hash一个指和指纹匹配，防止服务器证书被修改

### 客户端通过服务器数字证书获取服务器公钥
1. "客户" -> "服务器": 你好
2. "服务器" -> "客户": 你好，我是服务器，这是我的证书
   * (客户端可以做证书校验, 获取服务器公钥)
3. "客户" -> "服务器": 向我证明是你服务器，这是一个随机字符串
4. "服务器" -> "客户": {随机字符串}[私钥|RSA]
5. "客户" -> "服务器": {我们后面用对称加密通信，这是对称加密算法和密钥}[公钥|RSA]
6. "服务器" -> "客户": {OK，收到！}[密钥|对称加密算法]
7. "客户" -> "服务器": {我的账号是aaa, 密码是123， 把我的余额信息发给我看看}[密钥|对称加密算法]
8. "服务器" -> "客户": {你的当前余额是100元}[密钥|对称加密算法]

## Java Keystore
### JKS，Java Key Store
* sun.security.provider.JavaKeyStore
* 此密钥库是特定于Java平台的，通常具有jks的扩展名。
* 此类型的密钥库可以包含私钥和证书，但不能用于存储密钥。
* 由于它是Java特定的密钥库，因此不能在其他编程语言中使用。
* 存储在JKS中的私钥无法在Java中提取。
* 调用KeyStore.getInstance(“JKS”)时，会创建一个不区分大小写的JKS实例版
* 当调用KeyStore.getInstance(“CaseExactJKS”)时，将创建一个区分大小写的JKS实例版本。
* Examples:
  ```
  // cretae mytestkey.jks with password
  try{
      KeyStore keyStore = KeyStore.getInstance(“JKS”);
      keyStore.load(null,null); 
      keyStore.store(new FileOutputStream("mytestkey.jks"), "password".toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
  
  // store private key and related cert chain
  try{
      KeyStore keyStore = KeyStore.getInstance(“JKS”);
      keyStore.load(new FileOutputStream("mytestkey.jks"), "password".toCharArray()); 
      CertAndKeyGen gen = new CertAndKeyGen(“RSA”,”SHA1WithRSA”);
      gen.generate(1024);
      Key key = gen.getPrivateKey();
      X509Certificate cert = gen.getSelfCertificate(new X500Name(“CN=ROOT”), (long)365*24*3600);
      X509Certificate[] chain = new X509Certificate[1];
      chain[0]=cert;
      keyStore.setKeyEntry(“mykey”, key, “password”.toCharArray(), chain);
      keyStore.store(new FileOutputStream("mytestkey.jks"), "password".toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
  
  // store single cert
  try{
      KeyStore keyStore = KeyStore.getInstance(“JKS”);
      keyStore.load(new FileOutputStream("mytestkey.jks"), "password".toCharArray()); 
      CertAndKeyGen gen = new CertAndKeyGen(“RSA”,”SHA1WithRSA”);
      gen.generate(1024);
      X509Certificate cert = gen.getSelfCertificate(new X500Name(“CN=SINGLE_CERTIFICATE”), (long)365*24*3600);
      keyStore.setCertificateEntry(“single_cert”, cert);
      keyStore.store(new FileOutputStream("mytestkey.jks"), "password".toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
  
  // load private key
  try{
      KeyStore keyStore = KeyStore.getInstance(“JKS”);
      keyStore.load(new FileOutputStream("mytestkey.jks"), "password".toCharArray()); 
      Key key = keyStore.getKey(“mykey”, “password”.toCharArray());
      java.security.cert.Certificate[] chain = keyStore.getCertificateChain(“mykey”);
  } catch(Exception ex){
      ex.printStackTrace();
  }
  
  // load single cert
  try{
      KeyStore keyStore = KeyStore.getInstance(“JKS”);
      keyStore.load(new FileOutputStream("mytestkey.jks"), "password".toCharArray()); 
      java.security.cert.Certificate cert = keyStore.getCertificate(“single_cert”);
  } catch(Exception ex){
      ex.printStackTrace();
  }   
  ```
### JCEKS (Java Cryptography Extension KeyStore)
* com.sun.crypto.provider.JceKeyStore
* 可以认为是增强式的JKS密钥库，支持更多算法。
* 此密钥库具有jceks的扩展名。
* 可以放入JCEKS密钥库的条目是私钥，密钥(对称加密算法)和证书。
* 此密钥库通过使用Triple DES加密为存储的私钥提供更强大的保护。
* Examples:
  ```
  // store key
  try{
      KeyStore keyStore = KeyStore.getInstance(“JCEKS”);
      keyStore.load(null,null); 
      KeyGenerator.getInstance(“DES”);
      keyGen.init(56);
      Key key = keyGen.generateKey();
      keyStore.setKeyEntry(“mykey”, key, “password”.toCharArray(), null);
      keyStore.store(new FileOutputStream("mytestkey.jceks"), "password".toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
  
  // load key
  try{
      KeyStore keyStore = KeyStore.getInstance(“JCEKS”);
      keyStore.load(new FileOutputStream("mytestkey.jceks"), "password".toCharArray()); 
      Key key = keyStore.getKey(“mykey”, “password”.toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
  ```
  
### PKCS12
* sun.security.pkcs12.PKCS12KeyStore
* 一种标准的密钥库类型，可以在Java和其他语言中使用。
* 它通常具有p12或pfx的扩展名。
* 以在此类型上存储私钥，密钥和证书。
* PKCS12密钥库上的私钥可以用Java提取。
* Examples:
  ```
  // store key
  try{
      KeyStore keyStore = KeyStore.getInstance(“PKCS12”);
      keyStore.load(null,null); 
      KeyGenerator.getInstance(“AES”);
      keyGen.init(128);
      Key key = keyGen.generateKey();
      keyStore.setKeyEntry(“mykey”, key, “password”.toCharArray(), null);
      keyStore.store(new FileOutputStream("mytestkey.p12"), "password".toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
  
  // store private key
  try{
      KeyStore keyStore = KeyStore.getInstance(“PKCS12”);
      keyStore.load(null,null); 
      CertAndKeyGen gen = new CertAndKeyGen(“RSA”,”SHA1WithRSA”);
      gen.generate(1024);
      Key key = gen.getPrivateKey();
      X509Certificate cert = gen.getSelfCertificate(new X500Name(“CN=ROOT”), (long)365*24*3600);
      X509Certificate[] chain = new X509Certificate[1];
      chain[0]=cert;
      keyStore.setKeyEntry(“mykey”, key, “password”.toCharArray(), chain);
      keyStore.store(new FileOutputStream("mytestkey.p12"), "password".toCharArray());
  } catch(Exception ex){
      ex.printStackTrace();
  }
    
  // load private key
  try{
      KeyStore keyStore = KeyStore.getInstance(“PKCS12”);
      keyStore.load(new FileOutputStream("mytestkey.p12"), "password".toCharArray()); 
      Key key = keyStore.getKey(“mykey”, “password”.toCharArray());
      java.security.cert.Certificate[] chain = keyStore.getCertificateChain(“mykey”);
  } catch(Exception ex){
      ex.printStackTrace();
  }
  ```
  
### PKCS11
* sun.security.pkcs11.P11KeyStore
* 一种硬件密钥库类型。 为Java库提供了一个接口，用于连接硬件密钥库设备
* 此密钥库可以存储私钥，密钥和证书。


