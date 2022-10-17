# Security JVM启动参数
* -Djava.security.manager ：启动默认的安全管理器；
* -Djava.security.policy="E:/java.policy"：指定安全管理器的配置文件；
* 实例化一个java.lang.SecurityManager或继承它的子类的对象，然后通过System.setSecurityManager()来设置并启动一个安全管理器。
  ```System.setSecurityManager(new SecurityManager())；```
* JVM自带的Policy文件位于%JAVA_HOME%/jre/lib/security/java.policy；


# Protection mechanisms - Java security architecture
* Java runtime’s protection mechanisms can prevent untrusted Java applications from loading and using native code in the first place.
* Stack-based access control
  1. This mechanism limits access to certain functionalities of the JCL which restricts the capabilities of code to prevent undesired activities.
  2. Security policy
     * set customizable security policies that assign access permissions to code.
       1. programmatically
          ```
          Policy.setPolicy(new MinimalPolicy());
          public class MinimalPolicy extends Policy { 
              private static PermissionCollection perms; 
              public MinimalPolicy() {
                  super();
                  if (perms == null) {
                      perms = new MyPermissionCollection();
                      addPermissions();
                  }
              }

              @Override
              public PermissionCollection getPermissions(CodeSource codesource) {
                  return perms;
              }

              private void addPermissions() {
                  SocketPermission socketPermission = new SocketPermission("*:1024-", "connect, resolve");
                  PropertyPermission propertyPermission = new PropertyPermission("*", "read, write");
                  FilePermission filePermission = new FilePermission("<<ALL FILES>>", "read");

                  perms.add(socketPermission);
                  perms.add(propertyPermission);
                  perms.add(filePermission);
              }

          }
          
          class MyPermissionCollection extends PermissionCollection {
              private static final long serialVersionUID = 614300921365729272L;
              ArrayList<Permission> perms = new ArrayList<Permission>();
          
              public void add(Permission p) {
                  perms.add(p);
              }

              public boolean implies(Permission p) {
                  for (Iterator<Permission> i = perms.iterator(); i.hasNext();) {
                      if (((Permission) i.next()).implies(p)) {
                          return true;
                      }
                  }
                  return false;
              }

              public Enumeration<Permission> elements() {
                  return Collections.enumeration(perms);
              }

              public boolean isReadOnly() {
                  return false;
              }

          }
          ```
       2. supplying a policy file
          ```
          grant signedBy "MyCompany" {
              permission java.io. FilePermission "C:/temp/*.txt", "read ";
          };
          grant codeBase "file:C:/MyApplication/-" {
              permission java.io. FilePermission "C:/conf/*", "read ,write ";
          };
          grant {
              permission java.util.PropertyPermission "os.name", "read ";
          }
          ```
          * grant all code that was signed by “MyCompany” read access to files in “C:/temp/” whose name ends with “.txt”. 
          * grant all code that resides under the local file path “C:/MyApplication” read and write access to all files in “C:/conf/”
          * grant all code read access to Java’s system property “os.name”


# java.security.SecurityManager
```
     SecurityManager security = System.getSecurityManager();

     if (security != null) {
         security.checkPermission(permission);
     }
```


# java.security.AccessController
* getContext
* checkPermission
* doPrivileged(PrivilegedAction), doPrivileged(PrivilegedExceptionAction)
  1. 个调用者在调用doPrivileged方法时，可被标识为 "特权"。
  2. 在做访问控制决策时，如果checkPermission方法遇到一个通过doPrivileged调用而被表示为 "特权"的调用者，并且没有上下文自变量，checkPermission方法则将终止检查。
  3. example:
  ```
  somemethod() {
      ...
      AccessController.doPrivileged(new PrivilegedAction() {
          public Object run() {
              System.loadLibrary("awt");  // checkPermission in loadLibrary will be skipped.
              return null;
          }
      });
      ...
  }
  ```


# java.security.AccessControlContext
* 用于根据其封装的上下文做出系统资源访问决策。
* optimize
* checkPermission
* property: isPrivileged, isAuthorized, isLimited, isWrapped
* property: ProtectionDomain context[]               // for checking permission
* property: ProtectionDomain limitedContext[]
* property: Permission permissions[]                 // limited privilege scope
* property: AccessControlContext privilegedContext   // the doPrivileged() context. The privilegedContext.isLimited should be false 
* property: AccessControlContext parent
* property: DomainCombiner combiner


# java.security.SecureClassLoader
* assign classes to protection domains when they are first defined
```
protected final Class<?> defineClass(String name, byte[] b, int off, int len, CodeSource cs) {
    return defineClass(name, b, off, len, getProtectionDomain(cs));
}
```


# java.security.Permission
| Permission                          | Comment                                                                                                                                                                                                                                                                                                      |
|-------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| java.security.AllPermission         | Grants all permissions and thus unlimited access to features of the JCL.                                                                                                                                                                                                                                     |
| java.io.FilePermission              | Grants access to the file system. Arguments specify allowed paths and allowed actions, including “read”, “write”, “execute”, and “delete”.                                                                                                                                                                   |
| java.security.SecurityPermission    | Grants access to certain security-critical features. Parameters specify the exact kind of access that shall be granted, e.g., the “setPolicy” argument allows code to set the security policy.                                                                                                               |
| java.net.SocketPermission           | Grants access to network connections via sockets. Arguments can be used to specify allowed hosts, e.g., by URL, and allowed ways to connect to the host, including “accept”, “connect”, “listen”, and “resolve”.                                                                                             |
| javax.sound.sampled.AudioPermission | Grants access to audio resources. Arguments can be “play” or “record”.                                                                                                                                                                                                                                       |
| java.lang.reflect.ReflectPermission | Allows for bypassing Java’s rules for information hiding when reflectively accessing class members, e.g., by calling AccessibleObject.setAccessible()                                                                                                                                                        |
| java.lang.RuntimePermission         | Covers a broad range of different actions, arguments specify the exact kind of access that shall be granted. For example, “loadLibrary.{library name}” allows for dynamically loading the specified library, and “setSecurityManager” allows for putting a security manager in charge of policy enforcement. |


# High-level components of JRE
* JRE comprises two high-level components.
* The first component is the Java Virtual Machine (JVM).
  1. It is responsible for executing the bytecode contained in Java class files.
  2. For this, it implements various different features, including a class file parser, an interpreter, just-in-time (JIT) compiler, a garbage collector for memory management, and more.
  3. It is implemented in native code and thus platform dependent.
* The second high-level component is the Java Class Library (JCL).
  1. It is a large set of system classes that implement commonly used features, such as user interface functionality, file access, or network access.
  2. The JCL is similar to the standard libraries of other languages
  3. Most parts of the JCL are implemented in Java, so just as for application classes. (the JVM is responsible for their execution)
  4. Application classes can use the features implemented in the JCL by means of method calls, class inheritance, callbacks, etc.
  5. By design, any meaningful Java code is required to call methods in the JCL, because the bytecode instruction set does not provide any commands for direct access to native libraries or operating system features.