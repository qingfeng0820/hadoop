# set up cas (central_authentication_server) project
  ```
  casVersion = '4.0.3'
  List casLibs = [
    "org.jasig.cas:cas-server-core:${casLibs}",
    "org.jasig.cas:cas-server-support-saml:${casLibs}",
    "org.jasig.cas:cas-server-support-openid:${casLibs}",
    "org.jasig.cas:cas-server-support-x509:${casLibs}"     
  ]
  
  compile casLibs
  // cas-server-web depends on cas-server-web-support and cas-server-security-filter
  compile org.jasig.cas:cas-server-web:${casLibs}
  
  // ===========================================================
  Use case:
  Create main project skelenton like cas-server-web.war.
  Do the customized changes in main project.
  Use cas-server-web.war to overlay the main project by maven overlays (modified files should be excluded in overlay)
  
  maven <overlays>标签下可以加入多个<overlay>来支持多个war包的合并到主web项目
  examples:
      <build>
        <plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.2.2</version>
                <configuration>
                  <warName>cas</warName>
                  <overlays>
		            <overlay>
                      <groupId>org.jasig.cas</groupId>
              		  <artifactId>cas-server-webapp</artifactId>
              		  <excludes>
                         <exclude>WEB-INF/lib/log4j-*.jar</exclude>
                         <exclude>WEB-INF/unused-spring-configuration/**</exclude>
              		  </excludes> 
            		</overlay>
                  </overlays>
                </configuration>
             </plugin>
	      <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>2.3.2</version>
               <configuration>
                 <source>1.8</source>
                 <target>1.8</target>
                 <encoding>iso-8859-1</encoding>
               </configuration>
	      </plugin>
          </plugins>
          <directory>${output}/target</directory>
      </build>
  
       <overlay>下若有<include>的配置，则最终合并的文件只包含include下匹配的文件
       <includes>
           <include>WEB-INF/view/jsp/**</include>
       </includes> 
  
       <overlay>下的war必须在dependencies下面
       <dependencies>
         <dependency>
            <groupId>org.jasig.cas</groupId>
            <artifactId>cas-server-webapp</artifactId>
            <version>4.0.3</version>
            <type>war</type>
        </dependency>       
       </dependencies>
  ```

* configuration
  * 在WEB-INF/spring-configuration中

    1. /WEB-INF/spring-configuration/applicationContext.xml
    这个配置文件是cas的核心类配置，你不需要改动。
    
    2. /WEB-INF/spring-configuration/argumentExtractorsConfiguration.xml
    这个配置文件主要是cas参数的提取。比如从应用端重定向到cas 服务器的url地址中的service参数，为什么cas认识，service起什么作用，换一参数名，是否可以？就是这里配置的类来处理的。但是这个你也不需要改动，cas默认是支持cas1.0,cas2.0及saml协议的。
    
    3. /WEB-INF/spring-configuration/propertyFileConfigurer.xml
    加载cas.properties文件。
    
    4. /WEB-INF/spring-configuration/securityContext.xml
    关于安全上下文配置，比如登出，认证等，一般情况下不需要改动它
    
    5. /WEB-INF/spring-configuration/ticketExpirationPolicies.xml
    从文件名就可以知道，它是关于ticket的过期策略配置的,包括ST,TGT.
    
    6. /WEB-INF/spring-configuration/ticketGrantingTicketCookieGenerator.xml
    关于cookie的生成
    
    7. /WEB-INF/spring-configuration/ticketRegistry.xml
    ticket的存储
    
    8. /WEB-INF/spring-configuration/uniqueIdGenerators.xml
    ticket Id生成器

  * 在WEB-INF/中

    1. /WEB-INF/cas-servlet.xml
    spring mvc的启动类配置
    
    2. /WebContent/WEB-INF/deployerConfigContext.xml
    cas的认证管理器，认证管理都在这个文件里，可以说进行cas开发，你需要更改的文件中，这是第一个。
    
    3. /WEB-INF/login-webflow.xml
    spring web flow的流程配置文件。读懂了这个文件就可以了解cas的登录流程。
    
    4. /WEB-INF/restlet-servlet.xml
    关于cas 的restlet对外接口服务的.