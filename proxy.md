### [动态代理](https://mp.weixin.qq.com/s/kgB03P7Ocqj8EUTv55-zNw)

动态代理，是使用反射和字节码计数，在运行期创建指定接口或类的子类、以及其实例对象的计数。使用动态代理可以无侵入的为代码进行增强。

动态代理主要有 JDK 和 CGLIB 两种实现方式。

#### 1、JDK 实现动态代理

##### 1.1、JDK 实现动态代理的两个成员 `Proxy`、`InvocationHandler`：

- `Proxy`：所有动态代理的父类，提供一个静态方法来创建动态代理的类对象和实例；
- `InvocatiionHandler`：每个动态代理实例都有一个关联的 `InvocationHandler`，在代理实例上调用方法时，方法调用将被转发到 `InvocationHandler` 的 `invoke` 方法。

###### JDK 动态代理原理图

![图片](/assets/jdk_proxy_image.png)

##### 1.2、代码实现

模拟用户注册功能——使用动态代理对用户的注册功能进行增强：判断用户名和密码的长度，如果用户名长度小于等于 1 或密码长度小于等于 6 则抛出异常。

`User.java`

```java
public class User {
 
 private String name;
 
 private Integer age;
 
 private String password;
 
 public String getName() {
  return name;
 }
 
 public void setName(String name) {
  this.name = name;
 }
 
 
 public Integer getAge() {
  return age;
 }
 
 public void setAge(Integer age) {
  this.age = age;
 }
 
 public String getPassword() {
  return password;
 }
 
 public void setPassword(String password) {
  this.password = password;
 }
 
 @Override
 public String toString() {
  return "User [name=" + name + ", age=" + age + ", password=" + password + "]";
 }
 
}
```

`UserService.java`

```java
public interface UserService {
 
 void addUser(User user);
}
```

`UserServiceImpl.java`

```java
public class UserServiceImpl implements UserService {

   @Override
   public void addUser(User user) {
    System.out.println("jdk...正在注册用户，用户信息为："+user);
   }

}
```

`UserServiceInterceptor.java`

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class UserServiceInterceptor implements InvocationHandler {
 
 private Object realObj;
 
 public UserServiceInterceptor(Object realObject) {
  super();
  this.realObj = realObject;
 }
 
 @Override
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  if (args!=null && args.length > 0 && args[0] instanceof User) {
   User user = (User)args[0];
   //进行增强判断
   if (user.getName().length() <= 1) {
    throw new RuntimeException("用户名长度必须大于1");
   }
   if (user.getPassword().length() <= 6) {
    throw new RuntimeException("密码长度必须大于6");
   }
  }
  Object result = method.invoke(realObj, args);
  System.out.println("用户注册成功...");
  return result;
 }
 
 public Object getRealObj() {
  return realObj;
 }
 
 public void setRealObj(Object realObj) {
  this.realObj = realObj;
 }
 
}
```

`ClientTest.java`

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;


public class ClientTest {
 
 public static void main(String[] args) {
  User user = new User();
  user.setName("hhh");
  user.setPassword("h");
  user.setAge(2);
  //被代理类
  UserService delegate = new UserServiceImpl();
  InvocationHandler userServiceInterceptor = new UserServiceInterceptor(delegate);
  //动态代理类
  UserService userServiceProxy = (UserService)Proxy.newProxyInstance(delegate.getClass().getClassLoader(),
    delegate.getClass().getInterfaces(), userServiceInterceptor);
  System.out.println("动态代理类："+userServiceProxy.getClass());
  userServiceProxy.addUser(user);
 }
}
```

这里就使用 JDK 动态代理实现了动态增强的作用。表面上调用了代理对象 `userServiceProxy` 的 `addUser(user)` 方法，实际上，调用了 `userServiceInterceptor` 的 `invoke` 方法，通过 `invoke` 方法返回实际被代理的对象 `delegate` 并调用了它的 `addUser(user)` 方法。

#### 2、CGLIB 动态代理

CGLIB（Code Generation Library）是一个基于 ASM 的字节码生成库，允许在运行时对字节码进行修改和动态生成。CGLIB 通过继承的方式实现代理。

##### 2.1、CGLIB 实现动态代理的两个成员 `Enhancer`、`MethodInterceptor`：

- `Enhancer`：用于指定要代理的目标对象，实际处理代理逻辑的是最终通过调用 `create()` 方法得到的对象，对这个对象的所有的非 `final` 方法的调用都会转发给 `MethodInterceptor`。
- `MethodInteceptor`：对动态代理对象方法的调用进行增强

###### CGLIB 动态代理原理图：

![图片](/assets/cglib_proxy_image.png)

##### 2.2、代码实现

`UserServiceCglibInterceptor.java`

```java
import java.lang.reflect.Method;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
 
public class UserServiceCglibInterceptor implements MethodInterceptor {
 
 private Object realObject;
 
 public UserServiceCglibInterceptor(Object realObject) {
  super();
  this.realObject = realObject;
 }
 
 @Override
 public Object intercept(Object object, Method method, Object[] args, MethodProxy proxy) throws Throwable {
  if (args!=null && args.length > 0 && args[0] instanceof User) {
   User user = (User)args[0];
   //进行增强判断
   if (user.getName().length() <= 1) {
    throw new RuntimeException("用户名长度必须大于1");
   }
   if (user.getPassword().length() <= 6) {
    throw new RuntimeException("密码长度必须大于6");
   }
  }
  Object result = method.invoke(realObject, args);
  System.out.println("用户注册成功...");
  return result;
 }
 
}
```

`ClientTest.java`

```java
import net.sf.cglib.proxy.Enhancer;
 
public class ClientTest {
 
 public static void main(String[] args) {
  User user = new User();
  user.setName("hhh");
  user.setPassword("h");
  user.setAge(2);
  //被代理的对象
  UserServiceImplCglib delegate = new UserServiceImplCglib();
  UserServiceCglibInterceptor serviceInterceptor = new UserServiceCglibInterceptor(delegate);
  Enhancer enhancer = new Enhancer();
  enhancer.setSuperclass(delegate.getClass());
  enhancer.setCallback(serviceInterceptor);
  //动态代理类
  UserServiceImplCglib cglibProxy = (UserServiceImplCglib)enhancer.create();
  System.out.println("动态代理类父类："+cglibProxy.getClass().getSuperclass());
  cglibProxy.addUser(user);
 }
}
```

#### 3、总结

JDK 动态代理是 Java 原生支持的、不需要外部依赖，但是它只能基于接口进行代理——已经继承了 `Proxy`，而 Java 不支持多继承。

由于 CGLIB 通过继承的方式实现代理，即使目标对象没有实现接口也可以代理，但是它不能增强被 `final` 修饰的方法——`final` 方法不能被重写。