# Start
![](./img/start.png)
tomcat 的源码程序启动类 Bootstrap 配置VM参数（注意路径需要修改为自己的项目位置），因为 tomcat 源码运行也需要加载配置文件等。

-Dcatalina.home=D:/IdeaProjects/apache-tomcat-8.5.35-src
-Dcatalina.base=D:/IdeaProjects/apache-tomcat-8.5.35-src
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.util.logging.config.file=D:/IdeaProjects/apache-tomcat-8.5.35-src/conf/logging.properties


# org.apache.catalina.startup.Bootstrap
* main -> init -> load -> start
* init
  1. Create three classloader: common (parent), server (for tomcat instance, parent class loader is common classloader), shared (shared for applications, parent class loader is common classloader)
  2. Set server classloader to be thread context class loader, then load tomcat related classes.
  3. Using server classloader to load Catalina class and create an instance Catalina (catalinaDaemon)
  4. Set catalinaDaemon's parent classloader to be share classloader
* load: call catalinaDaemon.load
* start: call catalinaDaemon.start


# org.apache.catalina.startup.Catalina
* load
  1. initDirs: check temporary folder "java.io.tmpdir"
  2. initNaming: set add "org.apache.naming" to variable java.naming.factory.url.pkgs, 
                 set java.naming.factory.initial to org.apache.naming.java.javaURLContextFactory if it's not set
  3. create start digester to parse configuration file (conf/server.xml) and create tomcat objects (Server, Service, Engine, Host ...)
  4. set Catalina , catalina.home and catalina.base to Server
  5. initStreams: Replace System.out and System.err with a custom PrintStream by SystemLogHandler
  6. call Server.init
* start
  1. call Server.start
  2. set shutdownHook to runtime.
  3. if await, call await and stop (listening for a shutdown command). No await for embedded tomcat.

## catalina.home和catalina.base这两个属性仅在你需要安装多个Tomcat实例而不想安装多个软件备份的时候使用，这样能节省磁盘空间。
* catalina.home(安装目录)：指向公用信息的位置，就是bin和lib的父目录。
* catalina.base(工作目录)：指向每个Tomcat目录私有信息的位置，就是conf、logs、temp、webapps和work的父目录。
```
In many circumstances, it is desirable to have a single copy of a Tomcat binary distribution shared among multiple users on the same server. 
To make this possible, you can set the CATALINA_BASE environment variable to the directory that contains the files for your 'personal' Tomcat instance.

When running with a separate CATALINA_HOME and CATALINA_BASE, the files and directories are split as following:
In CATALINA_BASE:
bin - Only: setenv.sh (*nix) or setenv.bat (Windows), tomcat-juli.jar
conf - Server configuration files (including server.xml)
lib - Libraries and classes, as explained below
logs - Log and output files
webapps - Automatically loaded web applications
work - Temporary working directories for web applications
temp - Directory used by the JVM for temporary files>

In CATALINA_HOME:
bin - Startup and shutdown scripts
lib - Libraries and classes, as explained below
endorsed - Libraries that override standard "Endorsed Standards". By default it's absent.
```
# Tomcat SSL
* org.apache.catalina.connector.Connector.setProperty set the prop to ProtocolHandler
* Configuration for SSL (defaultSSLHostConfig)
  ```
  <Connector port="443"
    protocol="org.apache.coyote.http11.Http11Protocol"
    SSLEnabled="true"
    scheme="https"
    secure="true"
    keystoreFile="/xxx/tomcat/cert/restlessman.cn.pfx"
    keystoreType="PKCS12"
    keystorePass="xxxxx"
    clientAuth="false"
    SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"
    ciphers="TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA256"/>
  ```
  * Configuration for other SSL HostConfig
    ```
    <Connector port="443"
      protocol="org.apache.coyote.http11.Http11Protocol"
      SSLEnabled="true"
      scheme="https"
      secure="true">
      <Engine name="Catalina" defaultHost="localhost">
          <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
              <SSLHostConfig 
                SSLProtocol="TLSv1+TLSv1.1+TLSv1.2" 
                ciphers="TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA256">
                   <Certificate certificateKeystoreFile="/xxx/tomcat/cert/restlessman.cn.pfx" keystoreType="PKCS12" keystorePass="xxxxx" ... />
              </SSLHostConfig>
              <!-- Additional Host configurations -->
          </Host>
      </Engine>
    </Connector>
    ```
* modify conf/web.xml for redirecting http to https
  ``` 
   <login-config>
       <!-- Authorization setting for SSL -->
       <auth-method>CLIENT-CERT</auth-method>
       <realm-name>Client Cert Users-only Area</realm-name>
   </login-config>
   <security-constraint>
       <!-- Authorization setting for SSL -->
       <web-resource-collection >
           <web-resource-name >SSL</web-resource-name>
           <url-pattern>/*</url-pattern>
       </web-resource-collection>
       <user-data-constraint>
           <transport-guarantee>CONFIDENTIAL</transport-guarantee>
       </user-data-constraint>
   </security-constraint>
  ```
  
* getContextPath, getServletPath, getPathInfo, getRequestURI
  * 你的web application 名称为news,你在浏览器中输入请求路径：http://localhost:8080/news/main/list.jsp
    ```
     servlet mapping:
     <servlet-mapping>
        <servlet-name>xxx</servlet-name>
        <url-pattern>/main/list.jsp</url-pattern>
     </servlet-mapping>
    ```
    1. System.out.println(request.getContextPath()); //可返回站点的根路径。也就是项目的名字。 打印结果：/news
    2. System.out.println(request.getServletPath()); 打印结果：/main/list.jsp
    3. request.getPathInfo(); 为null
    4. System.out.println(request.getRequestURI()); 打印结果：/news/main/list.jsp
    5. System.out.println(request.getRealPath("/")); 打印结果：F:\Tomcat 6.0\webapps\news
  * getServletPath and getPathInfo
    1. "" and "/main/list.jsp"
       ```
       servlet mapping:
       <servlet-mapping>
       <servlet-name>xxx</servlet-name>
       <url-pattern>/*</url-pattern>
       </servlet-mapping>
       ```
    2. "/main" and "/list.jsp"
       ```
       servlet mapping:
       <servlet-mapping>
       <servlet-name>xxx</servlet-name>
       <url-pattern>/main*</url-pattern>
       </servlet-mapping>
       ```
    3. "/main/list.jsp" and null
       ```
       servlet mapping:
       <servlet-mapping>
       <servlet-name>xxx</servlet-name>
       <url-pattern>*.jsp</url-pattern>
       </servlet-mapping>
       ```