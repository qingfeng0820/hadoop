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
  * 证书颁发机构用签名算法和它自己的私钥对颁发的证书里面的指纹和指纹算法做加密，形成的密文就是签名 (指纹和指纹算法在签名里面)
  * 客户端可以用证书颁发机构本身的证书里的公钥和签名算法解密签名，得到服务器证书的指纹和指纹算法，然后用指纹算法对服务器的证书hash一个指纹，和获取的指纹匹配，防止服务器证书被修改

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
      KeyGenerator keyGen = KeyGenerator.getInstance(“DES”);
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
      KeyGenerator keyGen = KeyGenerator.getInstance(“AES”);
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

### Other operations
```
package com.xtayfjpk.security.certificate;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.math.BigInteger;
import java.security.Key;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.PrivateKey;
import java.security.Provider;
import java.security.PublicKey;
import java.security.Security;
import java.security.Signature;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;
import java.security.spec.RSAPublicKeySpec;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import javax.security.auth.x500.X500Principal;

import org.bouncycastle.asn1.DERBMPString;
import org.bouncycastle.asn1.pkcs.PKCSObjectIdentifiers;
import org.bouncycastle.asn1.x500.X500Name;
import org.bouncycastle.asn1.x509.AlgorithmIdentifier;
import org.bouncycastle.asn1.x509.Certificate;
import org.bouncycastle.asn1.x509.SubjectPublicKeyInfo;
import org.bouncycastle.cert.X509CertificateHolder;
import org.bouncycastle.cert.X509v3CertificateBuilder;
import org.bouncycastle.crypto.params.AsymmetricKeyParameter;
import org.bouncycastle.crypto.params.RSAKeyParameters;
import org.bouncycastle.crypto.util.PrivateKeyFactory;
import org.bouncycastle.crypto.util.PublicKeyFactory;
import org.bouncycastle.crypto.util.SubjectPublicKeyInfoFactory;
import org.bouncycastle.jce.interfaces.PKCS12BagAttributeCarrier;
import org.bouncycastle.jce.provider.BouncyCastleProvider;
import org.bouncycastle.operator.ContentSigner;
import org.bouncycastle.operator.DefaultDigestAlgorithmIdentifierFinder;
import org.bouncycastle.operator.DefaultSignatureAlgorithmIdentifierFinder;
import org.bouncycastle.operator.bc.BcRSAContentSignerBuilder;
import org.bouncycastle.pkcs.PKCS10CertificationRequest;
import org.bouncycastle.pkcs.PKCS10CertificationRequestBuilder;
import org.bouncycastle.x509.X509V3CertificateGenerator;
import org.junit.Before;
import org.junit.Test;

/**
 * issuer	证书颁发者
 * subject	证书使用者
 * 
 * DN：Distinguish Name
 * 格式：CN=姓名,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=省/市/自治区名称,C=国家双字母
 *
 */
@SuppressWarnings("deprecation")
public class CertifacateGenerateTest {
	
	private static final String KEY_PAIR_ALG = "RSA";
	private static final String SIG_ALG = "SHA1withRSA";
	private static final String DN_ZHANGSAN = "CN=zhangsan,OU=development,O=Huawei,L=ShenZhen,ST=GuangDong,C=CN";
	private static final String DN_CA = "CN=Kingyea,OU=Kingyea,O=Kingyea,L=GuangZou,ST=GuangDong,C=CN";
	private static Map<String, String> algorithmMap = new HashMap<>();
	
	static {
		/**
		 * 算法名称与算法标识符映射
		 */
		algorithmMap.put("1.2.840.113549.1.1.5", SIG_ALG);
		algorithmMap.put("1.2.840.113549.1.1.1", KEY_PAIR_ALG);
	}

	@Before
	public void before() {
		//注冊BC Provider，由于有些关于证书的操作使用到了BouncyCastle这个第三方库就顺便注冊上了，事实上不注冊也行
		Provider provider = new BouncyCastleProvider();
		Security.addProvider(provider);
	}
	
	/**
	 * 生成根证书公钥与私钥对
	 */
	@Test
	public void testGenRootKeyPair() throws Exception {
		KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(KEY_PAIR_ALG);
		keyPairGenerator.initialize(2048);
		KeyPair keyPair = keyPairGenerator.generateKeyPair();
		writeObject("H:/certtest/Kingyea.public", keyPair.getPublic());
		writeObject("H:/certtest/Kingyea.private", keyPair.getPrivate());
	}
	
	
	/**
	 * 生成用户证书公钥与私钥对
	 * @throws Exception
	 */
	@Test
	public void testZhangsanKeyPair() throws Exception {
		KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(KEY_PAIR_ALG);
		keyPairGenerator.initialize(2048);
		KeyPair keyPair = keyPairGenerator.generateKeyPair();
		writeObject("H:/certtest/zhangsan.public", keyPair.getPublic());
		writeObject("H:/certtest/zhangsan.private", keyPair.getPrivate());
	}
	
	/**
	 * 生成根证书(被BC废弃，但能够使用)
	 */
	@Test
	public void testGenRootCert() throws Exception {
		X509V3CertificateGenerator certGen = new X509V3CertificateGenerator();
		//设置证书颁发者
		certGen.setIssuerDN(new X500Principal(DN_CA));
		//设置证书有效期
		certGen.setNotAfter(new Date(System.currentTimeMillis()+ 100 * 24 * 60 * 60 * 1000));
		certGen.setNotBefore(new Date());
		//设置证书公钥
		certGen.setPublicKey(getRootPublicKey());
		//设置证书序列号
		certGen.setSerialNumber(BigInteger.TEN);
		//设置签名算法
		certGen.setSignatureAlgorithm(SIG_ALG);
		//设置证书使用者
		certGen.setSubjectDN(new X500Principal(DN_CA));
		//使用私钥生成证书。主要是为了进行签名操作
		X509Certificate certificate = certGen.generate(getRootPrivateKey());
		PKCS12BagAttributeCarrier bagAttr = (PKCS12BagAttributeCarrier)certificate;
		bagAttr.setBagAttribute(
	            PKCSObjectIdentifiers.pkcs_9_at_friendlyName,
	            new DERBMPString("Kingyea Coperation Certificate"));
		writeFile("H:/certtest/ca.cer", certificate.getEncoded());
	}
	
	/**
	 * 生成根证书的第二种方式
	 * @throws Exception
	 */
	@Test
	public void testGenRootCertWithBuilder() throws Exception {
		final AlgorithmIdentifier sigAlgId = new DefaultSignatureAlgorithmIdentifierFinder().find(SIG_ALG);
		final AlgorithmIdentifier digAlgId = new DefaultDigestAlgorithmIdentifierFinder().find(sigAlgId);
		
		PublicKey publicKey = getRootPublicKey();
		PrivateKey privateKey = getRootPrivateKey();
		
		
		X500Name issuer = new X500Name(DN_CA);
		BigInteger serial = BigInteger.TEN;
		Date notBefore = new Date();
		Date notAfter = new Date(System.currentTimeMillis()+ 100 * 24 * 60 * 60 * 1000);
		X500Name subject = new X500Name(DN_CA);
		
		
		AlgorithmIdentifier algId = AlgorithmIdentifier.getInstance(PKCSObjectIdentifiers.rsaEncryption.toString());
		System.out.println(algId.getAlgorithm());
		AsymmetricKeyParameter publicKeyParameter = PublicKeyFactory.createKey(publicKey.getEncoded());
		SubjectPublicKeyInfo publicKeyInfo = SubjectPublicKeyInfoFactory.createSubjectPublicKeyInfo(publicKeyParameter);
		//此种方式不行，生成证书不完整
		//SubjectPublicKeyInfo publicKeyInfo = new SubjectPublicKeyInfo(algId, publicKey.getEncoded());
		X509v3CertificateBuilder x509v3CertificateBuilder = new X509v3CertificateBuilder(issuer, serial, notBefore, notAfter, subject, publicKeyInfo);
		
		BcRSAContentSignerBuilder contentSignerBuilder = new BcRSAContentSignerBuilder(sigAlgId, digAlgId);
		AsymmetricKeyParameter privateKeyParameter = PrivateKeyFactory.createKey(privateKey.getEncoded());
		ContentSigner contentSigner = contentSignerBuilder.build(privateKeyParameter);
		
		X509CertificateHolder certificateHolder = x509v3CertificateBuilder.build(contentSigner);
		Certificate certificate = certificateHolder.toASN1Structure();
		writeFile("H:/certtest/ca.cer", certificate.getEncoded());
	}
	
	/**
	 * 生成用户证书
	 */
	@Test
	public void testGenZhangsanCert() throws Exception {
		X509V3CertificateGenerator certGen = new X509V3CertificateGenerator();
		certGen.setIssuerDN(new X500Principal(DN_CA));
		certGen.setNotAfter(new Date(System.currentTimeMillis()+ 100 * 24 * 60 * 60 * 1000));
		certGen.setNotBefore(new Date());
		certGen.setPublicKey(getZhangsanPublicKey());
		certGen.setSerialNumber(BigInteger.TEN);
		certGen.setSignatureAlgorithm(SIG_ALG);
		certGen.setSubjectDN(new X500Principal(DN_ZHANGSAN));
		X509Certificate certificate = certGen.generate(getRootPrivateKey());
		
		writeFile("H:/certtest/zhangsan.cer", certificate.getEncoded());
	}
	
	
	/**
	 * 验证根证书签名
	 */
	@Test
	public void testVerifyRootCert() throws Exception {
		CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
		FileInputStream inStream = new FileInputStream("H:/certtest/ca.cer");
		X509Certificate certificate = (X509Certificate) certificateFactory.generateCertificate(inStream);
		System.out.println(certificate);
		Signature signature = Signature.getInstance(certificate.getSigAlgName());
		signature.initVerify(certificate);
		signature.update(certificate.getTBSCertificate());
		boolean legal = signature.verify(certificate.getSignature());
		System.out.println(legal);
	}
	
	/**
	 * 验证用户证书签名
	 */
	@Test
	public void testVerifyZhangsanCert() throws Exception {
		CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
		FileInputStream inStream = new FileInputStream("H:/certtest/zhangsan.cer");
		X509Certificate certificate = (X509Certificate) certificateFactory.generateCertificate(inStream);
		System.out.println(certificate.getPublicKey().getClass());
		Signature signature = Signature.getInstance(certificate.getSigAlgName());
		signature.initVerify(getRootPublicKey());
		signature.update(certificate.getTBSCertificate());
		boolean legal = signature.verify(certificate.getSignature());
		System.out.println(legal);
	}
	
	
	/**
	 * 生成证书请求文件
	 */
	@Test
	public void testGenCSR() throws Exception {
		X500Name subject = new X500Name(DN_ZHANGSAN);
		AsymmetricKeyParameter keyParameter = PrivateKeyFactory.createKey(getZhangsanPrivateKey().getEncoded());
		SubjectPublicKeyInfo publicKeyInfo = SubjectPublicKeyInfoFactory.createSubjectPublicKeyInfo(keyParameter);
		PKCS10CertificationRequestBuilder certificationRequestBuilder = new PKCS10CertificationRequestBuilder(subject, publicKeyInfo);
		final AlgorithmIdentifier sigAlgId = new DefaultSignatureAlgorithmIdentifierFinder().find(SIG_ALG);
		final AlgorithmIdentifier digAlgId = new DefaultDigestAlgorithmIdentifierFinder().find(sigAlgId);
		BcRSAContentSignerBuilder contentSignerBuilder = new BcRSAContentSignerBuilder(sigAlgId, digAlgId);
		PKCS10CertificationRequest certificationRequest = certificationRequestBuilder.build(contentSignerBuilder.build(keyParameter));
		System.out.println(certificationRequest);
		writeFile("H:/certtest/zhangsan.csr", certificationRequest.getEncoded());
	}
	
	/**
	 * 依据证书请求文件生成用户证书，事实上主要是使用根证书私钥为其签名
	 */
	@Test
	public void testZhangsanCertWithCSR() throws Exception {
		byte[] encoded = readFile("H:/certtest/zhangsan.csr");
		PKCS10CertificationRequest certificationRequest = new PKCS10CertificationRequest(encoded);
		
		
		RSAKeyParameters parameter = (RSAKeyParameters) PublicKeyFactory.createKey(certificationRequest.getSubjectPublicKeyInfo());
		RSAPublicKeySpec keySpec = new RSAPublicKeySpec(parameter.getModulus(), parameter.getExponent());
		String algorithm = algorithmMap.get(certificationRequest.getSubjectPublicKeyInfo().getAlgorithm().getAlgorithm().toString());
		PublicKey publicKey = KeyFactory.getInstance(algorithm).generatePublic(keySpec);
		System.out.println(certificationRequest.getSubject());
		X509V3CertificateGenerator certGen = new X509V3CertificateGenerator();
		certGen.setIssuerDN(new X500Principal(DN_CA));
		certGen.setNotAfter(new Date(System.currentTimeMillis()+ 100 * 24 * 60 * 60 * 1000));
		certGen.setNotBefore(new Date());
		
		certGen.setPublicKey(publicKey);
		certGen.setSerialNumber(BigInteger.TEN);
		certGen.setSignatureAlgorithm(algorithmMap.get(certificationRequest.getSignatureAlgorithm().getAlgorithm().toString()));
		certGen.setSubjectDN(new X500Principal(certificationRequest.getSubject().toString()));
		X509Certificate certificate = certGen.generate(getRootPrivateKey());
		
		writeFile("H:/certtest/zhangsan.cer", certificate.getEncoded());
		
	}
	
	public PrivateKey getRootPrivateKey() throws Exception {
		return PrivateKey.class.cast(readKey("H:/certtest/Kingyea.private"));
	}
	public PublicKey getRootPublicKey() throws Exception {
		return PublicKey.class.cast(readKey("H:/certtest/Kingyea.public"));
	}
	
	public PrivateKey getZhangsanPrivateKey() throws Exception {
		return PrivateKey.class.cast(readKey("H:/certtest/zhangsan.private"));
	}
	public PublicKey getZhangsanPublicKey() throws Exception {
		return PublicKey.class.cast(readKey("H:/certtest/zhangsan.public"));
	}
	
	
	public byte[] readFile(String path) throws Exception {
		FileInputStream cntInput = new FileInputStream(path);
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
		int b = -1;
		while((b=cntInput.read())!=-1) {
			baos.write(b);
		}
		cntInput.close();
		byte[] contents = baos.toByteArray();
		baos.close();
		return contents;
	}
	
	public void writeFile(String path, byte[] content) throws Exception {
		FileOutputStream fos = new FileOutputStream(path);
		fos.write(content);
		fos.close();
	}
	
	public void writeObject(String path, Object object) throws Exception {
		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(path));
		oos.writeObject(object);
		oos.close();
	}
	
	public Object readObject(String path) throws Exception {
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream(path));
		Object obj = ois.readObject();
		ois.close();
		return obj;
	}
	
	public Key readKey(String path) throws Exception {
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream(path));
		Key key = Key.class.cast(ois.readObject());
		ois.close();
		return key;
	}
}



获取待签发的证书

CertificateFactory cf=CertificateFactory.getInstance("X.509");

FileInputStream in2=new FileInputStream("user.csr");

java.security.cert.Certificate c2=cf.generateCertificate(in);

从待签发的证书中提取证书信息

byte [] encod2=c2.getEncoded();

X509CertImpl cimp2=new X509CertImpl(encod2);  用该编码创建X509CertImpl类型对象

X509CertInfo cinfo2=(X509CertInfo)cimp2.get(X509CertImpl.NAME+"."+X509CertImpl.INFO);  获取X509CertInfo对象

设置新证书有效期

Date begindate=new Date(); 获取当前时间

Date enddate=new Date(begindate.getTime()+3000*24*60*60*1000L); 有效期为3000天

CertificateValidity cv=new CertificateValidity(begindate,enddate); 创建对象

cinfo2.set(X509CertInfo.VALIDITY,cv);  设置有效期

设置新证书序列号

int sn=(int)(begindate.getTime()/1000);    以当前时间为序列号

CertificateSerialNumber csn=new CertificateSerialNumber(sn);

cinfo2.set(X509CertInfo.SERIAL_NUMBER,csn);


设置新证书签发者

cinfo2.set(X509CertInfo.ISSUER+"."+CertificateIssuerName.DN_NAME,issuer);应用第三步的结果

设置新证书签名算法信息

AlgorithmId algorithm=new AlgorithmId(AlgorithmId.md5WithRSAEncryption_oid);

cinfo2.set(CertificateAlgorithmId.NAME+"."+CertificateAlgorithmId.ALGORITHM,algorithm);

创建证书并使用CA的私钥对其签名

X509CertImpl newcert=new X509CertImpl(cinfo2);
// caprk 是CA的private key
newcert.sign(caprk,"MD5WithRSA"); 使用CA私钥对其签名

将新证书写入密钥库

ks.setCertificateEntry("lf_signed",newcert);

FileOutputStream out=new FileOutputStream("newstore");

ks.store(out,"newpass".toCharArray());  这里是写入了新的密钥库，也可以使用第七条来增加条目



验证证书的有效期
CertificateFactory cf=CertificateFactory.getInstance("X.509");

FileInputStream in1=new FileInputStream("aa.crt");

java.security.cert.Certificate  c1=cf.generateCertificate(in1);

X509Certificate t=(X509Certificate)c1;

in1.close();

Date TimeNow=new Date();

try{

t.checkValidity(TimeNow);

System.out.println("OK");

}catch(CertificateExpiredException e){  //过期

System.out.println("Expired");

System.out.println(e.getMessage());

}catch((CertificateNotYetValidException e){ //尚未生效

System.out.println("Too early");

System.out.println(e.getMessage());}


验证证书签名的有效性
CertificateFactory cf=CertificateFactory.getInstance("X.509");

FileInputStream in2=new FileInputStream("caroot.crt");

java.security.cert.Certificate  cac=cf.generateCertificate(in2);

in2.close();

PublicKey pbk=cac.getPublicKey();

boolean pass=false;

try{

c1.verify(pbk);

pass=true;

}catch(Exception e){

pass=false;

System.out.println(e);
}

SSL认证和keystore使用
import java.io.FileInputStream;

import java.io.*;

import java.net.Socket;

import java.security.KeyStore;

import javax.net.ssl.KeyManagerFactory;

import javax.net.ssl.SSLContext;

import javax.net.ssl.SSLServerSocket;

import javax.net.ssl.SSLServerSocketFactory;

public class KeystoreTest {

/**

* name:KeystoreTest

* author:suju

*/

public static void main(String[] args) throws Exception{

String key="c:/.keystore";

KeyStore keystore=KeyStore.getInstance("JKS");

//keystore的类型，默认是jks

keystore.load(new FileInputStream(key),"123456".toCharArray());

//创建jkd密钥访问库    123456是keystore密码。

KeyManagerFactory kmf=KeyManagerFactory.getInstance("SunX509");

kmf.init(keystore,"asdfgh".toCharArray());

//asdfgh是key密码。

//创建管理jks密钥库的x509密钥管理器，用来管理密钥，需要key的密码

SSLContext sslc=SSLContext.getInstance("SSLv3");

// 构造SSL环境，指定SSL版本为3.0，也可以使用TLSv1，但是SSLv3更加常用。

sslc.init(kmf.getKeyManagers(),null,null);

//第二个参数TrustManager[] 是认证管理器，在需要双向认证时使用，

//构造ssl环境

SSLServerSocketFactory sslfactory=sslc.getServerSocketFactory();

SSLServerSocket serversocket=(SSLServerSocket) sslfactory.createServerSocket(9999);

//创建serversocket，监听，并传输数据来验证授权

for(int i=0;i<15;i++)

{

final Socket socket=serversocket.accept();

new Thread(){

public void run()

{

try{

InputStream is=socket.getInputStream();

OutputStream os=socket.getOutputStream();

byte[] buf=new byte[1024];

int len=is.read(buf);

System.out.println(new String(buf));

os.write("ssl test".getBytes());

os.close();

is.close();

}catch(Exception e)

{// }

}

}.start();

}

serversocket.close();

}

}

客户端：

[代码]java代码：

import java.io.FileInputStream;

import java.io.InputStream;

import java.io.OutputStream;

import java.security.KeyStore;

import javax.net.ssl.KeyManagerFactory;

import javax.net.ssl.SSLContext;

import javax.net.ssl.SSLServerSocket;

import javax.net.ssl.SSLServerSocketFactory;

import javax.net.ssl.SSLSocket;

import javax.net.ssl.SSLSocketFactory;

import javax.net.ssl.TrustManagerFactory;

public class KeystoreTestClient {

/**

* name:KeystoreTestClient

* author:suju

*/

public static void main(String[] args) throws Exception{

String key="c:/client";

KeyStore keystore=KeyStore.getInstance("JKS");  //创建一个keystore来管理密钥库

keystore.load(new FileInputStream(key),"123456".toCharArray());

//创建jkd密钥访问库

TrustManagerFactory tmf=TrustManagerFactory.getInstance("SunX509");

tmf.init(keystore);                 //验证数据，可以不传入key密码

//创建TrustManagerFactory,管理授权证书

SSLContext sslc=SSLContext.getInstance("SSLv3");

// 构造SSL环境，指定SSL版本为3.0，也可以使用TLSv1，但是SSLv3更加常用。

sslc.init(null,tmf.getTrustManagers(),null);

//KeyManager[] 第一个参数是授权的密钥管理器，用来授权验证。第二个是被授权的证书管理器，

//用来验证服务器端的证书。只验证服务器数据，第一个管理器可以为null

//构造ssl环境

SSLSocketFactory sslfactory=sslc.getSocketFactory();

SSLSocket socket=(SSLSocket) sslfactory.createSocket("127.0.0.1",9999);

//创建serversocket通过传输数据来验证授权

InputStream is=socket.getInputStream();

OutputStream os=socket.getOutputStream();

os.write("client".getBytes());

byte[] buf=new byte[1024];

int len=is.read(buf);

System.out.println(new String(buf));

os.close();

is.close();

}

}
```

