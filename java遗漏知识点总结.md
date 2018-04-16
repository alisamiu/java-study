## Java遗漏知识点总结

### 一、Java对象的序列化

Java平台允许我们在内存中创建可复用的Java对象，但一般情况下，只有当JVM处于运行时，这些对象才可能存在，这些对象的生命周期不会比JVM的生命周期更长。但在现实应用中，就可能要求在JVM停止运行之后能够保存(持久化)指定的对象，并在将来重新读取被保存的对象。Java对象的序列化就能够帮助我们实现该功能。

使用Java对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。必须注意地是，对象序列化保存的是对象的”状态”，即它的成员变量。由此可知，**对象序列化不会关注类中的静态变量**。

除了在持久化对象时会用到对象序列化之外，当使用RMI(远程方法调用)，或在网络中传递对象时，都会用到对象序列化。



1、在Java中，只要一个类实现了`java.io.Serializable`接口，那么它就可以被序列化。

2、通过`ObjectOutputStream`和`ObjectInputStream`对对象进行序列化及反序列化

3、虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 `private static final long serialVersionUID`）

4、序列化并不保存静态变量。

5、要想将父类对象也序列化，就需要让父类也实现`Serializable` 接口。

6、Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

7、服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。



在序列化过程中，如果被序列化的类中定义了writeObject 和 readObject 方法，虚拟机会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化，并通过反射的方式被调用。

如果没有这样的方法，则默认调用是 ObjectOutputStream 的 defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject 方法。

用户自定义的 writeObject 和 readObject 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。

#### 总结

1、如果一个类想被序列化，需要实现Serializable接口。否则将抛出`NotSerializableException`异常，这是因为，在序列化操作过程中会对类型进行检查，要求被序列化的类必须属于Enum、Array和Serializable类型其中的任何一种。

2、在变量声明前加上关键字transient，可以阻止该变量被序列化到文件中。

3、在类中增加writeObject 和 readObject 方法可以实现自定义序列化策略



### 二、反射

在了解反射之前先来看一下类的加载机制。

什么叫类的加载或类的初始化？当程序主动使用某个类时，如果该类还未被加载到内存中，则系统会通过加载、连接、初始化三个步骤来对该类进行初始化。

**类的加载**

类加载指的是将类的class文件读入内存，并为之创建一个java.lang.Class对象，系统中所有的类实际上也是实例，它们都是Java.lang.Class的实例。类加载的几种来源：

（1）从本地文件系统加载class文件

（2）从JAR包加载class文件

（3）通过网络加载class文件

（4）把一个Java源文件动态编译，并执行加载

**类的连接**

当类被加载之后，系统为之生成一个对应的Class对象，接着将会进入连接阶段，连接阶段负责把类的二进制数据合并到JRE中，类的连接分为以下三个阶段：

（1）验证：验证阶段用于检验被加载的类是否有正确的内部结构，并和其他类协调一致。

（2）准备：类准备阶段则负责为类的静态Field分配内存，并设置默认初始值。

（3）解析：将类的二进制数据中的符号引用替换成直接引用。

**类的初始化**

在类的初始化阶段，主要就是对静态Field进行初始化。JVM初始化一个类包含以下几个步骤：

（1）假如这个类还没有被加载和连接，则程序先加载并连接该类。

（2）假如该类的直接父类还没有被初始化，则先初始化其直接父类。

（3）假如类中有初始化语句，则系统依次执行这些初始化语句。

通过下面6种方式使用某个类或接口时，系统才会初始化该类或接口：

- 创建类的实例。其中为某个类创建实例的方式包括：使用new操作符来创建实例，通过反射来创建实例，通过反序列化的方式来创建实例。
- 调用某个类的静态方法。
- 访问某个类或接口的静态Field，或为该静态Field赋值。注意，如果当某个静态Field使用了final修饰，而且它的值可以在编译时就确定下来，那么程序其他地方使用该静态Field时，实际上并没有使用该静态Field，而是相当于使用常量。反之，如果final类型的静态Field的值不能在编译时确定下来，则必须等到运行时才可以确定该Field的值，如果通过该类来访问它的静态Field，则会导致该类被初始化。
- 使用反射方式来强制创建某个类或接口对应的java.lang.Class对象。例如：Class.forName()会初始化类并返回类对应的java.lang.Class对象。
- 初始化某个类的子类。当初始化某个类的子类时，该子类的所有父类都会被初始化。
- 直接使用java.exe命令来运行某个主类。



**获得Class对象的三种方式：**

- 使用Class类的forName(String className)静态方法。
- 调用某个类的class属性来获取该类对应的Class对象。
- 调用某个对象的getClass()方法。

其中最常用的方式是第一种。



**使用反射生成并操作对象有如下两种方式：**

- 使用Class对象的newInstance()方法来创建该Class对象对应类的实例。这种方式要求该Class对象的对应类有默认构造器，而执行newInstance()方法时实际上是利用默认构造器来创建该类的实例。
- 先使用Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法创建该Class对象对应类的实例。如果不想利用默认构造器创建Java对象，而想利用指定的构造器来创建Java对象，则需要选用第二种方式。



###三、动态代理（Spring AOP的实现原理）

关于动态代理，首先需要了解的是一种常用的设计模式—代理模式。



**代理模式**

代理模式是Java常用的设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。简单的说就是，我们在访问实际对象时，是通过代理对象来访问的，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。![屏幕快照 2018-04-07 下午5.38.31](https://ws2.sinaimg.cn/large/006tKfTcgy1fq5fequ2nrj31860pmgpt.jpg)



**静态代理**

由程序员创建或特定工具自动生成源代码，也就是在编译时就已经将接口，被代理类，代理类等确定下来。在程序运行之前，代理类的.class文件就已经生成。

先来看一个静态代理的简单例子：

```
/**

*创建Person接口

*/
public interface Person {
//上交班费
void giveMoney();
}
```

```
public class Student implements Person {
    private String name;
    public Student(String name) {
        this.name = name;
    }
    

@Override
public void giveMoney() {
   System.out.println(name + "上交班费50元");
}

}
```

```
/**

*学生代理类，也实现了Person接口，保存一个学生实体，这样既可以代理学生产生行为

 */
public class StudentsProxy implements Person{
//被代理的学生
Student stu;

public StudentsProxy(Person stu) {
    // 只代理学生对象
    if(stu.getClass() == Student.class) {
        this.stu = (Student)stu;
    }
}

//代理上交班费，调用被代理学生的上交班费行为
public void giveMoney() {
    stu.giveMoney();
}
}
```

测试一下，看如何实现代理：

```
public class StaticProxyTest {
    public static void main(String[] args) {
        //被代理的学生张三，他的班费上交有代理对象monitor（班长）完成
        Person zhangsan = new Student("张三");
        

    //生成代理对象，并将张三传给代理对象
    Person monitor = new StudentsProxy(zhangsan);

    //班长代理上交班费
    monitor.giveMoney();
}

}
```

代理模式最主要的就是有一个公共接口（Person），一个具体的类（Student），一个代理类（StudentsProxy）,代理类持有具体类的实例，代为执行具体类实例方法。上面说到，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。这里的间接性就是指不直接调用实际对象的方法，那么我们在代理过程中就可以加上一些其他用途。



**动态代理**

代理类在程序运行时创建的代理方式被称为动态代理。上面静态代理的例子中，代理类(studentProxy)是自己定义好的，在程序运行之前就已经编译完成。然而动态代理，代理类并不是在Java代码中定义的，而是在运行时根据我们在Java代码中的“指示”动态生成的。相比于静态代理， 动态代理的优势在于可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类中的方法。 

动态代理简单实现

在java的java.lang.reflect包下提供了一个Proxy类和一个InvocationHandler接口，通过这个类和这个接口可以生成JDK动态代理类和动态代理对象。

创建一个动态代理对象步骤，具体代码见后面：

- 创建一个InvocationHandler对象

  ```
  //创建一个与代理对象相关联的InvocationHandler
   InvocationHandler stuHandler = new MyInvocationHandler<Person>(stu);
  ```


- 使用Proxy类的getProxyClass静态方法生成一个动态代理类stuProxyClass 

  ```
  Class<?> stuProxyClass = Proxy.getProxyClass(Person.class.getClassLoader(), new Class<?>[] {Person.class});
  ```

- 获得stuProxyClass 中一个带InvocationHandler参数的构造器constructor

  ```
  Constructor<?> cons = stuProxyClass.getConstructor(InvocationHandler.class);
  ```

- 通过构造器constructor来创建一个动态实例stuProxy

  ```
  Person stuProxy = (Person) cons.newInstance(stuHandler);
  ```

就此，一个动态代理对象就创建完毕了，当然，上面四个步骤可以通过Proxy类的newProxyInstances方法简化：

```
//创建一个与代理对象相关联的InvocationHandler
  InvocationHandler stuHandler = new MyInvocationHandler<Person>(stu);
//创建一个代理对象stuProxy，代理对象的每个执行方法都会替换执行Invocation中的invoke方法
  Person stuProxy= (Person) Proxy.newProxyInstance(Person.class.getClassLoader(), new Class<?>[]{Person.class}, stuHandler);
```

依旧可以使用上面的例子用动态代理的方式来实现一下，其中接口类Person和被代理类Student代码都不变，其他代码如下：

创建StuInvocationHandler类，实现InvocationHandler接口，这个类中持有一个被代理对象的实例target。InvocationHandler中有一个invoke方法，所有执行代理对象的方法都会被替换成执行invoke方法。再在invoke方法中执行被代理对象target的相应方法。当然，在代理过程中，我们在真正执行被代理对象的方法前加入自己其他处理。这也是Spring中的AOP实现的主要原理。

```
public class StuInvocationHandler<T> implements InvocationHandler {
   //invocationHandler持有的被代理对象
    T target;
    

    public StuInvocationHandler(T target) {
       this.target = target;
    }

    /**

      * proxy:代表动态代理对象

      * method：代表正在执行的方法

      * args：代表调用目标方法时传入的实参
    */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理执行" +method.getName() + "方法");

        Object result = method.invoke(target, args);
        return result;
    }

}
```

再来测试一下看看：

```
public class ProxyTest {
    public static void main(String[] args) {
        
   //创建一个实例对象，这个对象是被代理的对象
    Person zhangsan = new Student("张三");  

   //创建一个与代理对象相关联的InvocationHandler
    InvocationHandler stuHandler = new StuInvocationHandler<Person>(zhangsan);

   //创建一个代理对象stuProxy来代理zhangsan，代理对象的每个执行方法都会替换执行Invocation中的invoke方法
    Person stuProxy = (Person) Proxy.newProxyInstance(Person.class.getClassLoader(), new Class<?>[]{Person.class}, stuHandler)；

   //代理执行上交班费的方法
    stuProxy.giveMoney();
}

}
```

动态代理的过程，代理对象和被代理对象的关系不像静态代理那样一目了然，清晰明了。因为动态代理的过程中，我们并没有实际看到代理类，也没有很清晰得看到代理类的具体样子，而且动态代理中被代理对象和代理对象是通过InvocationHandler来完成的代理过程的。



动态代理原理分析：我们可以对InvocationHandler看做一个中介类，中介类持有一个被代理对象，在invoke方法中调用了被代理对象的相应方法。通过聚合方式持有被代理对象的引用，把外部对invoke的调用最终都转为对被代理对象的调用。代理类调用自己方法时，通过自身持有的中介类对象来调用中介类对象的invoke方法，从而达到代理执行被代理对象的方法。也就是说，动态代理通过中介类实现了具体的代理功能。



### 四、内部类

可以将一个类的定义放在另一个类的定义内部，这就是内部类。可以定义在类体部,方法体部,甚至比方法体更小的代码块内部。

使用内部类最吸引人的原因是：每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

内部类的特性有:

1. 内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。
2. 在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。
3. 创建内部类对象的时刻并不依赖于外围类对象的创建。
4. 内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。
5. 内部类提供了更好的封装，除了该外围类，其他类都不能访问。

在外部类外面（或外部类main方法）创建内部对象的方法

```
OuterClass outerClass = new OuterClass();
OuterClass.InnerClass innerClass = outerClass.new InnerClass();
```

或者一步到位的方法

```
OuterClass.InnerClass innerClass=new OuterClass().new InnerClass();
```

在内部类中获取外部类的引用方法

```
public class InnerClass{
        //获取外部类的引用
        public OuterClass getOuterClass(){
            return OuterClass.this;
        }
        public void innerDisplay(){
            //内部类也可以通过外部类的引用访问外部元素
            getOuterClass().display();
        }
 }
```



广泛意义上的内部类一般来说包括这四种：成员内部类、局部内部类、匿名内部类和静态内部类。

**静态内部类(内部类中最简单的形式)**

1. 声明在类体内，方法体外，并且使用static修饰的内部类。

2. 访问特点可以类比静态变量和静态方法。

3. 脱离外部类的实例独立创建

   ​         在外部类的外部构建内部类的实例：

   ​                 new Outer.Inner();

   ​          在外部类的内部构建内部类的实例：

   ​                 new Inner();

4. 静态内部类体部可以直接访问外部类中所有的静态成员，包括私有成员，但不能访问外部类的所有非静态成员。

   ​

**局部内部类**

1. 定义在方法体，甚至比方法体更小的代码块中。

2. 类比局部变量。

3. 局部内部类是所有内部类中最少使用的一种形式。

4. 局部内部类可以访问的外部类的成员根据所在方法体不同而不同。

   ​        如果在静态方法中:

   ​                可以访问外部类中所有静态成员，包括私有

   ​        如果在实例方法中:

   ​               可以访问外部类中所有的成员，包含私有。

5. 局部内部类可以访问所在方法中定义的局部变量，但是要求局部变量必须使用final修饰。



**匿名内部类**

1. 没有名字的局部内部类。

2. 没有class,interface,implement,extend关键字。

3. 没有构造器（是唯一一个没有构造器的类）。

4. 一般隐式的继承某一个父类或者实现某一个接口。

   ​

### 五、包装类

虽然Java语言是典型的面向对象编程语言，但其中的八种基本数据类型并不支持面向对象的编程，基本类型的数据不具备“对象”的特性—不携带属性、没有方法可调用，为此Java为每种基本数据类型分别设计了对应的类，成为包装类。

| 基本数据类型 | 对应的包装类 |
| ------------ | ------------ |
| byte         | Byte         |
| short        | Short        |
| int          | Integer      |
| long         | Long         |
| char         | Character    |
| float        | Float        |
| double       | Double       |
| boolean      | Boolean      |

由基本类型向对应的包装类转换称为装箱。

包装类向对应的基本数据类型转换称为拆箱。

（1）实现int和Integer的相互转换

```
int m = 500;

Integer obj = new Integer(m); // 手动装箱

int n = obj.intValue(); // 手动拆箱
```

（2）将字符串转换为整数

```
Integer.parseInt(String s, int radix);//radix表示进制，默认十进制
```

（3）将整数转换为字符串

```
int m = 500;

String s = Integer.toString(m);
```

（4）自动拆箱和装箱

上面的例子是手动实例化一个包装类，称为手动拆箱装箱。Java2.5之后可以自动拆箱装箱。

```
int m = 500;

Integer obj = m; // 自动装箱

int n = obj; // 自动拆箱
```

注意事项：看下面代码的运行结果

```
Integer i1 = 127;
Integer i2 = 127;
Integer i3 = 128;
Integer i4 = 128;
System.out.println(i1==i2);
System.out.println(i3==i4);
```

输出结果：

true

false

出现这个原因是在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。因此i1和i2是直接从cache中取的已经存在的对象，而i3和i4则分别指向不同的对象。

