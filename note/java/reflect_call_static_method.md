## 使用 java.lang.reflect 包的 Method#invoke 调用

```
public class ReflectExample {

    public static String stringStaticMethod(String s) {
        return s + "|" + s;
    }

    public static void main(String[] args) throws Throwable {
        Method method = ReflectExample.class.
            getDeclaredMethod("stringStaticMethod", String.class);
        
        Object invoke = method.invoke(null, "invoke"); // obj 传 null
        System.out.println(invoke); // invoke|invoke
    }
}
```


## 使用 java.lang.invoke 包的 MethodHandle 调用
```
public class InvokeExample {

    public static String stringStaticMethod(String s) {
        return s + "|" + s;
    }
    
    public static void main(String[] args) {
        // 获取静态方法类型：返回值与参数均为 String
        MethodType methodType = MethodType.methodType(String.class, String.class);
        
        // 获取静态方法的句柄
        MethodHandle method =  MethodHandles.lookup()
        .findStatic(InvokeExample.class, "stringStaticMethod", methodType);
        
        // 调用方法
        Object r = method.invoke("invoke"); // invoke|invoke
    }
}
```