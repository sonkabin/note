# static关键字

## 四种使用场景

1. 修饰成员变量和成员方法
2. 静态代码块
3. 修饰类(只能修饰内部类)
4. 静态导包(用来导入类中的静态资源，1.5之后的新特性)

### 修饰成员变量和成员方法

被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。

方法区与 Java 堆一样，是**各个线程共享的内存区域**，它用于存储已被虚拟机加载的类信息（java.lang.Class对象）、常量、静态变量、即时编译器编译后的代码等数据。

### 静态代码块

一个类中的静态代码块可以有多个，位置可以随便放，它不在任何的方法体内。如果静态代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。

静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问.

```java
class Demo{
    static {
        a = 1;
        System.out.println(a); // error: Cannot reference a field before it is defined
    }
    private static int a;
}
```

#### static{}静态代码块与{}非静态代码块（构造代码块）

相同点： 都是在JVM加载类时且在构造方法执行之前执行，在类中都可以定义多个

```java
// 不同点： 
class SuperClass{
    static{
        System.out.print(1);
        // 内部类中无法定义静态代码块与静态方法
    }
    {
        System.out.print(2);
        // 构造代码块可以调用该类的其他方法
    }
    public SuperClass(){
        System.out.print(3);
    }
}
class SubClass extends SuperClass{
    static{
        System.out.print(4);
    }
    {
        System.out.print(5);
    }
    public SubClass(){
        System.out.print(6);
    }
}
@Test
public void testInit(){
    new SubClass(); // 142356
    new SubClass(); // 2356，每次new都会调用构造代码块和构造器
}
```

#### 静态代码块与常量

```java
class A{
    final static int a = 1;
    static {
        System.out.print("init ");
    }
}
class B{
    static int a = 1;
    static {
        System.out.print("init ");
    }
}
@Test
public void testInit(){
    System.out.println(A.a); // 1
    System.out.println(B.a); // init 1
}
/* final修饰，编译阶段将1存入到A类的常量池中，A对a的引用不会引起初始化过程；B会引起初始化
具体来说，javac为a生成ConstantValue属性为1，准备阶段JVM将值设为1；而对于b，准备阶段JVM设值为0
*/
// 当子类引用父类的静态字段，会引起父类初始化，子类是否初始化取决于JVM实现，Java默认JVM的不会初始化子类
```

### 静态内部类

非静态内部类在编译完成之后会隐含地保存外围类引用，而静态内部类却没有。

Example（静态内部类实现单例模式）

```java
public class Singleton {
    private Singleton() {}
    
    声明为 private 表明静态内部该类只能在该 Singleton 类中被访问
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

当 Singleton 类加载时，静态内部类 SingletonHolder 没有被加载进内存。只有当调用 `getInstance() `方法从而触发 `SingletonHolder.INSTANCE` 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。

> 初始化阶段，JVM会保证一个类的<clinit>()方法在多线程环境中被正确的初始化。<clinit>()方法是由编译器自动收集类中的所有类变量赋值动作和静态代码块中的语句合并而成。

**注**：区分符号引用与常量引用。符号引用的目标不一定已加载到内存中，而常量引用的目标一定在内存中。

### 静态导包

