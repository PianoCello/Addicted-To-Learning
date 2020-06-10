## 类型信息 ##

RTTI（RunTime Type Information，运行时类型信息）能够在程序运行时发现和使用类型信息。

 Java 是如何在运行时识别对象和类信息的：

- “传统的” RTTI：假定我们在编译时已经知道了所有的类型；
- “反射”机制：允许我们在运行时发现和使用类的信息；

### Class 类 ###

**Class 类是对所有普通类的抽象，每一个类都能用一个 Class 对象表示。**

所有的类都是第一次使用时动态加载到 JVM 中的，当程序创建第一个对类的静态成员的引用时，就会加载这个类。

> 其实构造器也是类的静态方法，虽然构造器前面并没有 `static` 关键字。所以，使用 `new` 操作符创建类的新对象，这个操作也算作对类的静态成员引用。

类加载器首先会检查这个类的 `Class` 对象是否已经加载，如果尚未加载，默认的类加载器就会根据类名查找 `.class` 文件。这个类的字节码被加载后，JVM 会对其进行验证，确保它没有损坏，并且不包含不良的 Java 代码。

所有 `Class` 对象都属于 `Class` 类，而且它跟其他普通对象一样，我们可以获取和操控它的引用(这也是类加载器的工作)。`forName()` 是 `Class` 类的一个静态方法，我们可以使用 `forName()` 根据目标类的类名（`String`）得到该类的 `Class` 对象。

#### 类字面常量 ####

Java 还提供了另一种方法来生成类对象的引用：**类字面常量**。

类字面常量不仅可以应用于普通类，也可以应用于接口、数组以及基本数据类型。另外，对于基本数据类型的包装类，还有一个标准字段 `TYPE`。`TYPE` 字段是一个引用，指向对应的基本数据类型的 `Class` 对象。

注意：当使用 `.class` 来创建对 `Class` 对象的引用时，不会自动地初始化该 `Class` 对象。为了使用类而做的准备工作实际包含三个步骤：

1. **加载**，这是由类加载器执行的。该步骤将查找字节码（通常在 classpath 所指定的路径中查找，但这并非是必须的），并从这些字节码中创建一个 `Class` 对象。
2. **链接**。在链接阶段将验证类中的字节码，为 `static` 字段分配存储空间，并且如果需要的话，将解析这个类创建的对其他类的所有引用。
3. **初始化**。如果该类具有超类，则先初始化超类，执行 `static` 初始化器和 `static` 初始化块。

直到第一次引用一个 `static` 方法（构造器隐式地是 `static`）或者非常量的 `static` 字段，才会进行类初始化。

#### 泛化的 Class 引用 ####

使用泛型对 `Class` 引用所指向的 `Class` 对象的类型进行限定：

```java
Class<Integer> genericIntClass = int.class;
			   genericIntClass = Integer.class;//同一个类型

//这看起来似乎是起作用的，因为 Integer 继承自 Number。
//但事实却是不行，因为 Integer 的 Class 对象并不是 Number的 Class 对象的子类
Class<Number> geenericNumberClass = int.class;//错误

// 通配符 ? 表示“任何事物”
Class<?> intClass = int.class;
         intClass = double.class;//编译成功

//为了创建一个限定指向某种类型或其子类的 Class 引用
//我们需要将通配符与 extends 关键字配合使用
 Class<? extends Number> bounded = int.class;
        				 bounded = double.class;
       					 bounded = Number.class;

```

使用 `Class<?>` 比单纯使用 `Class` 要好，虽然它们是等价的，并且单纯使用 `Class` 不会产生编译器警告信息。使用 `Class<?>` 的好处是它表示你并非是碰巧或者由于疏忽才使用了一个非具体的类引用，而是特意为之。

### 类的等价比较 ###

```java
class Base {
}

class Derived extends Base {
}

class FamilyVsExactType {
    static void test(Object x) {
        System.out.println((x instanceof Base));//true
        System.out.println((x instanceof Derived));//true

        System.out.println(Base.class.isInstance(x));//true
        System.out.println(Derived.class.isInstance(x));//true

        System.out.println((x.getClass().equals(Base.class)));//false
        System.out.println((x.getClass().equals(Derived.class)));//true
    }

    public static void main(String[] args) {
        test(new Derived());
    }
}
```

总结：子类的实例也是父类的实例，但是子类的 Class 对象和父类的 Class 对象不一致。

### 反射：运行时类信息 ###

类 `Class` 支持**反射**的概念， `java.lang.reflect` 库中包含类 `Field`、`Method` 和 `Constructor`（每一个都实现了 `Member` 接口）。这些类型的对象由 JVM 在运行时创建，以表示未知类中的对应成员。然后，可以使用 `Constructor` 创建新对象，`get()` 和 `set()` 方法读取和修改与 `Field` 对象关联的字段，`invoke()` 方法调用与 `Method` 对象关联的方法。此外，还可以调用便利方法 `getFields()`、`getMethods()`、`getConstructors()` 等，以返回表示字段、方法和构造函数的对象数组。因此，匿名对象的类信息可以在运行时完全确定，编译时不需要知道任何信息。

你使用反射与未知类型的对象交互时，JVM 将查看该对象，并看到它属于特定的类。在对其执行任何操作之前，必须加载 `Class` 对象。因此，RTTI 和反射的真正区别在于，使用 RTTI 时，编译器在编译时会打开并检查 `.class` 文件。换句话说，你可以用“正常”的方式调用一个对象的所有方法。通过反射，`.class` 文件在编译时不可用；它由运行时环境打开并检查。

### 动态代理 ###

**代理模式**：一个代理对象封装真实对象，代替其提供其他或不同的操作。

Java **动态代理**，动态创建代理对象而且动态处理对代理方法的调用。在动态代理上进行的所有调用都被重定向到单个**调用处理程序**（InvocationHandler），该处理程序负责发现调用的内容并决定如何处理。

```java
interface Interface {
    void doSomething();
    void somethingElse(String arg);
}

class RealObject implements Interface {
    @Override
    public void doSomething() {
        System.out.println("doSomething");
    }

    @Override
    public void somethingElse(String arg) {
        System.out.println("somethingElse " + arg);
    }
}

class DynamicProxyHandler implements InvocationHandler {
    private Object proxied;

    DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.printf("proxy: %s, method: %s, args: %s%n", proxy.getClass(), method, args);
        if (args != null) {
            for (Object arg : args) {
                System.out.println("  " + arg);
            }
        }
        return method.invoke(proxied, args);
    }
}

class SimpleDynamicProxy {
    private static void consumer(Interface iface) {
        iface.doSomething();
        iface.somethingElse("bonobo");
    }

    public static void main(String[] args) {
        RealObject real = new RealObject();
        consumer(real);

        // Insert a proxy and call again:
        Interface proxy = (Interface) Proxy.newProxyInstance(
                Interface.class.getClassLoader(),
                new Class[]{Interface.class},
                new DynamicProxyHandler(real));
        consumer(proxy);
    }
}
```

通过调用静态方法 `Proxy.newProxyInstance()` 来创建动态代理，该方法需要一个类加载器（通常可以从已加载的对象中获取），希望代理实现的接口列表，以及接口 `InvocationHandler` 的一个实现。动态代理会将所有调用重定向到**调用处理程序**，因此通常为调用处理程序的构造函数提供对“真实”对象的引用，以便一旦执行中介任务便可以转发请求。

`invoke()` 方法被传递给代理对象，以防万一你必须区分请求的来源---但是在很多情况下都无需关心。但是，在 `invoke()` 内的代理上调用方法时要小心，因为接口的调用是通过代理重定向的。

通常执行代理操作，然后使用 `Method.invoke()` 将请求转发给被代理对象，并携带必要的参数。



## 泛型 ##



