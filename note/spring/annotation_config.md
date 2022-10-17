#  org.springframework.web.SpringServletContainerInitializer
* It's a ServletContainerInitializer for no servlet configuration in web.xml
```
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer
```
* onStartup
  1. If scanned any WebApplicationInitializer class, create instances of them to initializers
  2. AnnotationAwareOrderComparator.sort(initializers)
  3. start initializers by WebApplicationInitializer.onStartup sequentially


# org.springframework.context.annotation.Configuration
* Use annotation Configuration for configuration classes (instead of spring xml configuration)


# Enable Spring 
* Use Servlet with load-on-startup (Spring started by DispatcherServlet.init)
  1. xml
     ```
     <servlet>
         <servlet-name>myapp</servlet-name>
         <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
         <init-param>
             <param-name>contextConfigLocation</param-name>
             <param-value>/WEB-INF/unisphere-servlet.xml</param-value>
         </init-param>
         <load-on-startup>1</load-on-startup>
     </servlet>
 
     <servlet-mapping>
         <servlet-name>myapp</servlet-name>
         <url-pattern>/*</url-pattern>
     </servlet-mapping>
     ```
     
     ```
     <!-- other ways to provide spring configuration instead of servlet init-param -->
     configuration in conext
     <Context ...>
          ...
          <Parameter name="contextConfigLocation" value="/WEB-INF/applicationContext.xml" />
          ...
     </Context>
     
     in web.xml
     <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>/WEB-INF/applicationContext.xml</param-value>
     </context-param> 
     ```
     
  2. program
     ```
     // servletContext (ServletContext)
     // Spring configuration  
     // configClasses Class<?>[] which are tagged with annotation org.springframework.context.annotation.Configuration
     AnnotationConfigWebApplicationContext beanContext = new AnnotationConfigWebApplicationContext();
     beanContext.register(configClasses);
  
     // ContextLoader.CONTEXT_INITIALIZER_CLASSES_PARAM = "contextInitializerClasses"
     // APPLICATION_CONTEXT_INITIALIZER_CLASS implements org.springframework.context.ApplicationContextInitializer
     servletContext.setInitParameter(ContextLoader.CONTEXT_INITIALIZER_CLASSES_PARAM, "<APPLICATION_CONTEXT_INITIALIZER_CLASS>");   
     // mappings (String[]): ["/api/*"]...
     ServletRegistration.Dynamic dispatcherServlet = servletContext.addServlet(servletName, new DispatcherServlet(beanContext));
     if (dispatcherServlet != null) {
         dispatcherServlet.setLoadOnStartup(1);
         dispatcherServlet.addMapping(mappings);
     }
     ```
* Use ServletContextListener (Spring started by ContextLoaderListener.contextInitialized)
  * If there are multiple servlets need to share backend business spring context, can introduce ContextLoaderListener to load backend spring beans.
  * each DispatcherServlet just loads spring MVC related beans separately.
  1. xml
     ```
     <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>/WEB-INF/applicationContext.xml</param-value>
     </context-param> 
     
     <listener>
         <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
     
     <servlet>
         <servlet-name>myapp</servlet-name>
         <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
         <init-param>
             <param-name>contextConfigLocation</param-name>
             <param-value>/WEB-INF/xxx.xml</param-value>
         </init-param>
         <load-on-startup>1</load-on-startup>
     </servlet>
 
     <servlet-mapping>
         <servlet-name>myapp</servlet-name>
         <url-pattern>/app1/*</url-pattern>
     </servlet-mapping>
     <servlet>
         <servlet-name>myapp2</servlet-name>
         <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
         <init-param>
             <param-name>contextConfigLocation</param-name>
             <param-value>/WEB-INF/xxx2.xml</param-value>
         </init-param>
         <load-on-startup>2</load-on-startup>
     </servlet>
 
     <servlet-mapping>
         <servlet-name>myapp2</servlet-name>
         <url-pattern>/app2/*</url-pattern>
     </servlet-mapping>
     ```
  2. program ()
     ```
     // servletContext (ServletContext)
     // Spring configuration  
     // configClasses Class<?>[] which are tagged with annotation org.springframework.context.annotation.Configuration
     AnnotationConfigWebApplicationContext rootContext = new AnnotationConfigWebApplicationContext();
     rootContext.register(configClasses);
     servletContext.addListener(new ContextLoaderListener(rootContext));
     
     // ContextLoader.CONTEXT_INITIALIZER_CLASSES_PARAM = "contextInitializerClasses"
     // APPLICATION_CONTEXT_INITIALIZER_CLASS implements org.springframework.context.ApplicationContextInitializer
     servletContext.setInitParameter(ContextLoader.CONTEXT_INITIALIZER_CLASSES_PARAM, "<APPLICATION_CONTEXT_INITIALIZER_CLASS>");
     
     // first DispatcherServlet
     AnnotationConfigWebApplicationContext webMvcContext1 = new AnnotationConfigWebApplicationContext();
     // webMcvConfigClasses1 Class<?>[] which are tagged with annotation org.springframework.context.annotation.Configuration
     webMvcContext1.register(webMcvConfigClasses1);
     // mappings (String[]): ["/api/*"]...
     ServletRegistration.Dynamic dispatcherServlet1 = servletContext.addServlet(servletName, new DispatcherServlet(webMvcContext1));
     if (dispatcherServlet1 != null) {
         dispatcherServlet1.setLoadOnStartup(1);
         dispatcherServlet1.addMapping(mappings);
     }
     // another DispatcherServlet
     AnnotationConfigWebApplicationContext webMvcContext2 = new AnnotationConfigWebApplicationContext();
     // webMcvConfigClasses2 Class<?>[] which are tagged with annotation org.springframework.context.annotation.Configuration
     webMvcContext2.register(webMcvConfigClasses2);
     ServletRegistration.Dynamic dispatcherServlet2 = servletContext.addServlet(servletName, new DispatcherServlet(webMvcContext1));
     if (dispatcherServlet2 != null) {
         dispatcherServlet2.setLoadOnStartup(2);
         dispatcherServlet2.addMapping(mappings2);
     }
     ...
     ```