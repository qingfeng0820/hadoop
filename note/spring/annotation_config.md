#  org.springframework.web.SpringServletContainerInitializer
* Configuration: META-INF/services/javax.servlet.ServletContainerInitializer
  ```org.springframework.web.SpringServletContainerInitializer```
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
* Use DispatcherServlet only if there is only one servlet
  * Only one Spring conext (including core context and MVC context), which is loaded by DispatcherServlet.init.
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

     // mappings (String[]): ["/api/*"]...
     ServletRegistration.Dynamic dispatcherServlet = servletContext.addServlet(servletName, new DispatcherServlet(beanContext));
     if (dispatcherServlet != null) {
         dispatcherServlet.setLoadOnStartup(1);
         dispatcherServlet.addMapping(mappings);
     }
     // beanContext's parent is null, because no ContextLoaderListener
     ```
* Use ServletContextListener and DispatcherServlet for multiple servlet
  * If there are multiple servlets need to share backend business spring context, can introduce ContextLoaderListener to load backend spring beans.
  * Spring core context (context init-params: contextConfigLocation) loaded by ContextLoaderListener.contextInitialized.
  * Each DispatcherServlet just loads spring MVC related beans (servlet init-params: contextConfigLocation) separately by DispatcherServlet.init.
    * HttpServletBean.int -> BeanWrapper.setPropertyValues(PropertyValues, true) -> set contextConfigLocation to the instance of DispatcherServlet 
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
     // ContextLoaderListener.contextInitialized -> ContextLoader.customizeContext 
     // -> ContextLoader.determineContextInitializerClasses will collect configired ApplicationContextInitializer classes for customizing ApplicationContext
     // -> create installs for these ApplicationContextInitializer classes and sort them
     // -> call method initialize of each ApplicationContextInitializer instance 
     
     
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
     ServletRegistration.Dynamic dispatcherServlet2 = servletContext.addServlet(servletName, new DispatcherServlet(webMvcContext2));
     if (dispatcherServlet2 != null) {
         dispatcherServlet2.setLoadOnStartup(2);
         dispatcherServlet2.addMapping(mappings2);
     }
     // when creating DispatcherServlets with webMvcContext1 and webMvcContext2, the rootContext will be set as parent context for webMvcContext1 and webMvcContext2.
     ...
     ```
     
## org.springframework.web.servlet.DispatcherServlet (load context servlet level webApplicationContext -> request attribute: DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE)
* initWebApplicationContext
  1. find root webApplicationContext
  2. if DispatcherServlet's webApplicationContext is not null, set root webApplicationContext as parent of DispatcherServlet's webApplicationContext and configureAndRefreshWebApplicationContext
  3. if DispatcherServlet's webApplicationContext is null, find webApplicationContext from the attribute of servlet context with specified name by contextAttribute
  4. if DispatcherServlet's webApplicationContext is still null, create webApplicationContext with root webApplicationContext as parent and configureAndRefreshWebApplicationContext.
  * configureAndRefreshWebApplicationContext:
    1. servlet context, servlet config
    2. namespace: default name of namespace -> <servlet-name>-servlet
    3. add application listener
    4. wac.getEnvironment and env.initPropertySources
    5. postProcessWebApplicationContext
    6. applyInitializers
    7. wac.refresh: if no configLocations, use default configLocations: getDefaultConfigLocations -> "/WEB-INF/<namespace>.xmlddd"

## org.springframework.web.context.ContextLoaderListener (load context root webApplicationContext)
* contextInitialized -> initWebApplicationContext
  1. if already have root webApplicationContext throw exception
  2. if ContextLoaderListener's webApplicationContext is null, then createWebApplicationContext
  3. configureAndRefreshWebApplicationContext
  4. wac.refresh: if no configLocations, use default configLocations: getDefaultConfigLocations -> "/WEB-INF/<namespace>.xml"