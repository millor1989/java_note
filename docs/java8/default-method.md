### 默认方法

Java8 允许接口中包含具有具体实现的方法，这种方法称为“默认方法”，默认方法使用 `default` 关键字修饰。

#### 1、默认方法的原则

1. “**类优先**”，如果一个类的父类中包含了一个与它所实现的接口的默认方法相同的方法，那么调用该类的该方法时，实际调用的是它的父类的方法，它所实现的接口的默认方法会被忽略。

   ```java
   public interface MyFunction {
   	default String getName() {
   		return "MyFunction";
   	}
   }
   
   public class MyClass {
       public String getName() {
           return "MyClass";
       }
   }
   
   public class SubClass extends MyClass implements MyFunction {
       
   }
   
   public class SubClassTest {
       @Test
       public void testDefaultFunction {
           SubClass sub = new SubClass();
           // 输出结果为：MyClass
           System.out.println(subClass.getName());
       }
   }
   ```

2. **接口冲突**。如果类实现多个（两个及以上）接口，这些接口具有相同的默认方法，那么该类必须通过重写（Override）这个冲突方法的方式来解决冲突

   ```java
   public interface MyFunction {
   	default String getName() {
   		return "function";
   	}
   }
   
   public interface MyInterface {
       default String getName() {
           return "interface";
       }
   }
   
   public class MyClass implements MyFunction, MyInterface {
       @Override
       public String getName() {
           // 调用 MyInterface 的 getName，返回 interface
           return MyInterface.super.getName();
           // 调用 MyFunction 的 getName，返回 function
           // return MyFunction.super.getName();
           
       }
   }
   ```

#### 2、接口中的静态方法

在 Java8 中，接口可以有静态方法，使用 &lt;interfaceName&gt;.&lt;methodName&gt; 的方式调用。

