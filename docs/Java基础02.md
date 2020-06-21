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

泛型是 Java 5 的重大变化之一。泛型实现了*参数化类型*，这样你编写的组件（通常是集合）可以适用于多种类型。“泛型”这个术语的含义是“适用于很多类型”。编程语言中泛型出现的初衷是通过解耦类或方法与所使用的类型之间的约束，使得类或方法具备最宽泛的表达力。

### 简单泛型 ###

促成泛型出现的最主要的动机之一是为了创建集合类。一个对象可以持有任意类型的对象，使用泛型可以这样定义：

```java
class Automobile {}

public class GenericHolder<T> {
    private T a;
    public GenericHolder() {}
    public void set(T a) { this.a = a; }
    public T get() { return a; }

    public static void main(String[] args) {
        GenericHolder<Automobile> h3 = new GenericHolder<>();
        h3.set(new Automobile()); // 此处有类型校验
        Automobile a = h3.get();  // 无需类型转换
    }
}
```

### 泛型方法 ###

参数化类中的方法。类本身可能是泛型的，也可能不是，不过这与它的方法是否是泛型的并没有什么关系。

如果方法是 **static** 的，则无法访问该类的泛型类型参数，因此，如果使用了泛型类型参数，则它必须是泛型方法。

要定义泛型方法，请将泛型参数列表放置在返回值之前，如下所示：

```java
public class GenericMethods {
    public <T> void f(T x) {
        System.out.println(x.getClass().getName());
    }
    public static void main(String[] args) {
        GenericMethods gm = new GenericMethods();
        gm.f("");
        gm.f(1);
        gm.f(1.0);
        gm.f(1.0F);
        gm.f('c');
        gm.f(gm);
    }
}
/* Output:
java.lang.String
java.lang.Integer
java.lang.Double
java.lang.Float
java.lang.Character
GenericMethods
*/
```

对于泛型类，必须在实例化该类时指定类型参数。使用泛型方法时，通常不需要指定参数类型，因为编译器会找出这些类型。 这称为**类型参数推断**。

#### 变长参数和泛型方法 ####

泛型方法和变长参数列表可以很好地共存：

```java
public class GenericVarargs {
    @SafeVarargs
    public static <T> List<T> makeList(T... args) {
        List<T> result = new ArrayList<>();
        for (T item : args)
            result.add(item);
        return result;
    }

    public static void main(String[] args) {
        List<String> ls = makeList("A");
        System.out.println(ls);
        ls = makeList("A", "B", "C");
        System.out.println(ls);
    }
}

```

`@SafeVarargs` 注解保证我们不会对变长参数列表进行任何修改，这是正确的，因为我们只从中读取。如果没有此注解，编译器将无法知道这些并会发出警告。

### 泛型擦除 ###

Java 泛型是使用擦除实现的。这意味着当你在使用泛型时，任何具体的类型信息都被擦除了，你唯一知道的就是你在使用一个对象。因此，`List<String>` 和 `List<Integer>` 在运行时实际上是相同的类型。它们都被擦除成原生类型 `List`。

#### 擦除的问题 ####

擦除主要的正当理由是从非泛化代码到泛化代码的转变过程，以及在不破坏现有类库的情况下将泛型融入到语言中。擦除允许你继续使用现有的非泛型客户端代码，直至客户端准备好用泛型重写这些代码。这是一个崇高的动机，因为它不会骤然破坏所有现有的代码。

擦除的代价是显著的。泛型不能用于显式地引用运行时类型的操作中，例如转型、**instanceof** 操作和 **new** 表达式。因为所有关于参数的类型信息都丢失了，当你在编写泛型代码时，必须时刻提醒自己，你只是看起来拥有有关参数的类型信息而已。

#### 边界处的动作 ####

因为擦除移除了方法体中的类型信息，所以在运行时的问题就是**边界**：**即对象进入和离开方法的地点**。这些正是编译器在编译期执行类型检查并插入转型代码的地点。

考虑如下这段非泛型示例：

```java
public class SimpleHolder {
    private Object obj;

    public void set(Object obj) {
        this.obj = obj;
    }

    public Object get() {
        return obj;
    }

    public static void main(String[] args) {
        SimpleHolder holder = new SimpleHolder();
        holder.set("Item");
        String s = (String) holder.get();
    }
}
```

如果用 **javap -c SimpleHolder** 反编译这个类，会得到如下内容（经过编辑）：

```java
public void set(java.lang.Object);
   0: aload_0
   1: aload_1
   2: putfield #2; // Field obj:Object;
   5: return

public java.lang.Object get();
   0: aload_0
   1: getfield #2; // Field obj:Object;
   4: areturn

public static void main(java.lang.String[]);
   0: new #3; // class SimpleHolder
   3: dup
   4: invokespecial #4; // Method "<init>":()V
   7: astore_1
   8: aload_1
   9: ldc #5; // String Item
   11: invokevirtual #6; // Method set:(Object;)V
   14: aload_1
   15: invokevirtual #7; // Method get:()Object;
   18: checkcast #8; // class java/lang/String
   21: astore_2
   22: return
```

`set()` 和 `get()` 方法存储和产生值，转型在调用 `get()` 时接受检查。

现在将泛型融入上例代码中：

```java
public class GenericHolder2<T> {
    private T obj;

    public void set(T obj) {
        this.obj = obj;
    }

    public T get() {
        return obj;
    }

    public static void main(String[] args) {
        GenericHolder2<String> holder =  new GenericHolder2<>();
        holder.set("Item");
        String s = holder.get();
    }
}
```

从 `get()` 返回后的转型消失了，但是我们还知道传递给 `set()` 的值在编译期会被检查。下面是相关的字节码：

```c
public void set(java.lang.Object);
   0: aload_0
   1: aload_1
   2: putfield #2; // Field obj:Object;
   5: return

public java.lang.Object get();
   0: aload_0
   1: getfield #2; // Field obj:Object;
   4: areturn

public static void main(java.lang.String[]);
   0: new #3; // class GenericHolder2
   3: dup
   4: invokespecial #4; // Method "<init>":()V
   7: astore_1
   8: aload_1
   9: ldc #5; // String Item
   11: invokevirtual #6; // Method set:(Object;)V
   14: aload_1
   15: invokevirtual #7; // Method get:()Object;
   18: checkcast #8; // class java/lang/String
   21: astore_2
   22: return
```

所产生的字节码是相同的。对进入 `set()` 的类型进行检查是不需要的，因为这将由编译器执行。而对 `get()` 返回的值进行转型仍然是需要的，只不过不需要你来操作，它由编译器自动插入，这样你就不用编写（阅读）杂乱的代码。

`get()` 和 `set()` 产生了相同的字节码，这就告诉我们泛型的所有动作都发生在边界处——**对入参的编译器检查和对返回值的转型**。这有助于澄清对擦除的困惑，记住：“边界就是动作发生的地方”。

### 补偿擦除 ###

泛型的擦除丢失了在泛型代码中执行某些操作的能力，所以，在运行时任何需要早知道确切类型信息的操作都将无法工作。

```cpp
public class Erased<T> {
    private  final int SIZE = 100;
    public  void f(Object arg){
        // if (arg instanceof T){}          // Error
        // T var = new T();                 // Error
        // T[] array = new T[SIZE];         // Error
        T[] array = (T[]) new Object[SIZE]; //Unchecked warning
    }
}
```

虽然偶尔可以绕过这些问题来编程，但是有时**必须引入类型标签来对擦除进行补偿**。这就意味着需要显示地传递 `Class` 对象，以便可以在类型表达式中使用它。

#### isInstance的补偿 ####

在前面的例子中，对 `instanceof` 进行使用但是最终还是失败了，因为其类型信息已经被擦除。但如果引入**类型标签**，就可以转而使用动态的 `isInstance()`。

```java
class Building {
}

class House extends Building {
}

public class ClassTypeCapture<T> {
    Class<T> kind;

    public ClassTypeCapture(Class<T> kind) {
        this.kind = kind;
    }

    public boolean f(Object arg) {
        return kind.isInstance(arg);
    }

    public static void main(String[] args) {
        ClassTypeCapture<Building> ctt1 = new ClassTypeCapture<>(Building.class);
        System.out.println(ctt1.f(new Building()));
        System.out.println(ctt1.f(new House()));

        ClassTypeCapture<House> ctt2 = new ClassTypeCapture<>(House.class);
        System.out.println(ctt2.f(new Building()));
        System.out.println(ctt2.f(new House()));
    }
}
```

#### 创建类型的实例 ####

试图在 **Erased.java** 中 `new T()` 是行不通的，部分原因是由于擦除，部分原因是编译器无法验证 **T** 是否具有默认（无参）构造函数。

Java 中的解决方案是传入一个工厂对象，并使用该对象创建新实例。方便的工厂对象只是 **Class** 对象，因此，如果使用类型标记，则可以使用 `newInstance()` 创建该类型的新对象：

```java
class ClassAsFactory<T> implements Supplier<T> {
    Class<T> kind;

    ClassAsFactory(Class<T> kind) {
        this.kind = kind;
    }
    @Override
    public T get() {
        try {
            return kind.newInstance();
        } catch (InstantiationException |
                IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}

class Employee {
    @Override
    public String toString() {
        return "Employee";
    }
}

public class InstantiateGenericType {
    public static void main(String[] args) {
       ClassAsFactory<Employee> fe = new ClassAsFactory<>(Employee.class);
       System.out.println(fe.get());
        
       ClassAsFactory<Integer> fi = new ClassAsFactory<>(Integer.class);
        try {
            //异常，Integer 并没有无参构造函数
            System.out.println(fi.get());
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}

```

#### 泛型数组 ####

泛型数组可以这样实现：

```java
public class GenericArrayWithTypeToken<T> {
    private T[] array; //而非 Object[], 否则在获取数组时发生转换异常

    @SuppressWarnings("unchecked")
    public GenericArrayWithTypeToken(Class<T> type, int sz) {
        array = (T[]) Array.newInstance(type, sz);
    }

    public void put(int index, T item) {
        array[index] = item;
    }

    public T get(int index) {
        return array[index];
    }

    // Expose the underlying representation:
    public T[] rep() {
        return array;
    }

    public static void main(String[] args) {
        GenericArrayWithTypeToken<Integer> gai =
                new GenericArrayWithTypeToken<>(Integer.class, 10);
        Integer[] ia = gai.rep();
    }
}
```

类型标记 **Class<T>** 被传递到构造函数中以从擦除中恢复，因此尽管必须使用 **@SuppressWarnings** 关闭来自强制类型转换的警告，但我们仍可以创建所需的实际数组类型。一旦获得了实际的类型，就可以返回它并产生所需的结果，如在主方法中看到的那样。数组的运行时类型是确切的类型 `T[]` 。

### 边界 ###

**边界（bounds）**允许我们对泛型使用的参数类型施加约束。由于擦除会删除类型信息，因此唯一可用于无限制泛型参数的方法是那些 **Object** 可用的方法。但是，如果将该参数限制为某类型的子集，则可以调用该子集中的方法。为了应用约束，Java 泛型使用了 `extends` 关键字。

### 通配符 ###

 **Java 泛型中的“通配符（Wildcards）”和“边界（Bounds）”的概念：**

- **<?>** ：无界通配符，看起来意味着“任何事物”，因此使用无界通配符好像等价于使用原生类型。

- **< ? extends T>**：是指 “上界通配符（Upper Bounds Wildcards）” ，不能往容器添加元素，取出的元素最少要用 T 接收。
- **< ? super T>**：是指 “下界通配符（Lower Bounds Wildcards）”，不影响往里存，但往外取只能用 Object 类型接收。

**PECS（Producer Extends Consumer Super）原则：**

- 频繁往外读取内容的，适合用上界 **< ? extends T>**。 
- 经常往里插入的，适合用下界 **< ? super T>**。

**泛型【T】与通配符【?】的区别：**

泛型和通配符最根本的区别就是 Java 编译器会把泛型【T】推断成具体类型，而把通配符【?】推断成未知类型。Java 编辑器只能操作具体类型，不能操作未知类型，如果有对参数做修改的操作就必须要使用泛型，如果仅仅是查看就可以使用通配符。

这样，我们可以利用通配符特性设计出安全的接口。比如在一个接口的方法中定义了通配符参数，则继承该接口的所有方法都不能修改该方法传递过来的参数。

```java
interface GInterface {
    <T> void foo(List<? extends T> list);
}

class GInterfaceImpl implements GInterface {
    @Override
    public <T> void foo(List<? extends T> list) {
        // 只能遍历list，不能修改list
        for (T t : list) {
            System.out.println(t);
        }
        // list.add(new Object()); // 编译器报错
    }

    public static void main(String[] args) {
        GInterfaceImpl gIterfaceImpl = new GInterfaceImpl();
        ArrayList<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        list.add("8");
        gIterfaceImpl.foo(list);
    }
}
```

上下界通配符的使用：

```java
class Fruit {}

class Apple extends Fruit {}

class Jonathan extends Apple {}

class Orange extends Fruit {}

class NonCovariantGenerics {
    public static void main(String[] args) {

        // Compile Error: incompatible types:
        //List<Fruit> flist = new ArrayList<Apple>();

        // Wildcards allow covariance:
        List<? extends Fruit> flist = new ArrayList<>();

        // Compile Error: can't add any type of object:
        // flist.add(new Apple());
        // flist.add(new Fruit());
        // flist.add(new Object());

        flist.add(null); // Legal but uninteresting
        // We know it returns at least Fruit:
        Fruit f = flist.get(0);

        List<? super Apple> apples = new ArrayList<>();
        apples.add(new Apple());
        apples.add(new Jonathan());
        // apples.add(new Fruit());//只能装 Apple 及其子类型

        System.out.println(apples);
    }
}
```

注意：不能把一个涉及 **Apple** 的泛型赋值给一个涉及 **Fruit** 的泛型。但是，有时你想在两个类型间建立某种向上转型关系。通配符可以产生这种关系。**flist** 的类型现在是 `List<? extends Fruit>`，你可以读作“一个具有任何从 **Fruit** 继承的类型的列表”。如果你调用了一个返回 **Fruit** 的方法，则是安全的，因为你知道这个 **List** 中的任何对象至少具有 **Fruit** 类型，因此编译器允许这么做。

### 自限定泛型 ###

在 Java 泛型中，有一个似乎经常性出现的惯用法，它相当令人费解：

```java
class SelfBounded<T extends SelfBounded<T>> {
    T element;
    SelfBounded<T> set(T arg) {
        element = arg;
        return this;
    }
    T get() { return element; }
}
```

**SelfBounded** 类接受泛型参数 **T**，而 **T** 由一个边界类限定，这个边界就是拥有 **T** 作为其参数的 **SelfBounded**。自限定所做的，就是要求在继承关系中，像下面这样使用这个类：

```java
class A extends SelfBounded<A>{}
```

自限定限制只能强制作用于继承关系。如果使用自限定，就应该了解这个类所用的类型参数将与使用这个参数的类具有相同的基类型。这会强制要求使用这个类的每个人都要遵循这种形式。 还可以将自限定用于泛型方法：

```java
public class SelfBoundingMethods {
    static <T extends SelfBounded<T>> T f(T arg) {
        return arg.set(arg).get();
    }

    public static void main(String[] args) {
        A a = f(new A());
    }
}
```

这可以防止这个方法被应用于除上述形式的自限定参数之外的任何事物上。

#### Enum 自限定 ####

```java
public abstract class Enum<E extends Enum<E>> {
    ...
}
```

深入解剖类声明 `Enum<E extends Enum<E>>`，它有以下几方面的意义：

- `Enum` 的泛型 `E` 的上界为 `Enum` 自身。这确保了只有 `Enum` 的子类才被允许成为泛型参数。

- 泛型 `E` 的上界被进一步限定为 `extends Enum<E>`，这确保了 `Enum<子类A>` 和 `Enum<子类A>的子类A` 的继承关系一定满足 `子类A extends Enum<子类A>` 。类似 `子类A extends Enum<子类B>` 这样的声明会被编译器拒绝，因为这个声明并不匹配泛型参数的上界。

- 基于 `Enum` 被设计为泛型，这意味着 `Enum` 中的某些方法的**方法参数及返回类型**是未知类型。而根据 `E extends Enum<E>`，我们可以知道 `E` 肯定会是 `Enum` 的子类。所以，在具象化类型 `Enum<某具体类>` 中，这些泛型方法的参数及返回类型就会被编译器转换为某具体类型。总结来说，`E extends Enum<E>` 保证了每个 `Enum<E>` 的子类中都能够接收并返回该子类类型 `E`。

#### 参数协变 ####

**协变（covariant）和逆变（contravariant）**：

- 协变：让一个带有协变参数的泛型接口（或委托）可以接收类型更加精细化，具体化的泛型接口（或委托）作为参数，可以看成OO中多态的一个延伸。
- 逆变：让一个带有协变参数的泛型接口（或委托）可以接收粒度更粗的泛型接口或委托作为参数，这个过程实际上是参数类型更加精细化的过程。

**自限定泛型**的价值在于它们可以产生**协变参数类型**——方法参数类型会随子类而变化。

尽管自限定泛型还可以产生与子类类型相同的返回类型，但是这并不十分重要，因为**协变返回类型**在 Java 5 引入，**协变返回类型**意思是子类覆盖父类方法时返回值可以更加具体化。

```java
class Base {}
class Derived extends Base {}
interface OrdinaryGetter {
    Base get();
}
interface DerivedGetter extends OrdinaryGetter {
    // Overridden method return type can vary:
    @Override
    Derived get(); //返回值更加具体了
}

public class CovariantReturnTypes {
    void test(DerivedGetter d) {
        Derived d2 = d.get();
    }
}

//使用自限定泛型实现返回值协变
interface GenericGetter<T extends GenericGetter<T>> {
    T get();
}
interface Getter extends GenericGetter<Getter> {}

public class GenericsAndReturnTypes {
    void test(Getter g) {
        Getter result = g.get();
        GenericGetter gg = g.get(); // Also the base type
    }
}
```

然而，在非泛型代码中，参数类型不能随子类型发生变化：

```java
// ------- 非泛型代码 无法实现参数协变 ----------
class OrdinarySetter {
    void set(Base base) {
        System.out.println("OrdinarySetter.set(Base)");
    }
}
class DerivedSetter extends OrdinarySetter {
    void set(Derived derived) {
        System.out.println("DerivedSetter.set(Derived)");
    }
}
public class OrdinaryArguments {
    public static void main(String[] args) {
        Base base = new Base();
        Derived derived = new Derived();
        
        DerivedSetter ds = new DerivedSetter();
        ds.set(derived);
        // Compiles--overloaded, not overridden!:
        ds.set(base); //这两个 set() 方法重载了 没有发生参数协变
    }
}

// ------- 自限定泛型 实现了参数协变 ----------
interface SelfBoundSetter<T extends SelfBoundSetter<T>> {
    void set(T arg);
}
interface Setter extends SelfBoundSetter<Setter> {}

public class SelfBoundingAndCovariantArguments {
    void testA(Setter s1, Setter s2, SelfBoundSetter sbs) {
        //只能接收 Setter 类型参数 发生了参数协变
        s1.set(s2);
        //s1.set(sbs); //编译出错 参数不匹配
    }
}

```

编译器不能识别将基类型当作参数传递给 `set()` 的尝试，因为没有任何方法具有这样的签名。实际上，这个参数已经被覆盖。



## 数组 ##

数组需要你去创建和初始化，你可以通过下标对数组元素进行访问，数组的大小不会改变。

### 数组特性 ###

将数组和集合区分开来的原因有三：**效率，类型，保存基本数据类型的能力**。在 Java 中，使用数组存储和随机访问对象引用序列是非常高效的。数组是简单的线性序列，这使得对元素的访问变得非常快，代价是数组对象的大小是固定的。

你应该总是从 **ArrayList** 开始，它将数组封装起来。必要时，它会自动分配更多的数组空间，创建新数组，并将旧数组中的引用移动到新数组。这种灵活性需要开销，所以一个 **ArrayList** 的效率不如数组。

在泛型前，集合类以一种宽泛的方式处理对象。事实上，这些集合类把保存对象的类型默认为 **Object**，也就是 Java 中所有类的基类。而数组是优于 **预泛型**（pre-generic）集合类的，因为你创建一个数组就可以保存特定类型的数据。这意味着你获得了一个编译时的类型检查，而这可以防止你插入错误的数据类型。**引入泛型之后，数组唯一剩下的优势就是效率。**

一个数组可以保存基本数据类型，而一个预泛型的集合不可以。然而对于泛型而言，集合可以指定和检查他们保存对象的类型，而通过 **自动装箱**（autoboxing）机制，集合表现得就像它们可以保存基本数据类型一样。

### 定义数组 ###

不管你使用的什么类型的数组，数组中的数据集实际上都是对堆中真正对象的引用。数组对象的一部分就是只读的 **length** 成员函数，它能告诉你数组对象中可以存储多少元素。**[ ]** 语法是你访问数组对象的唯一方式。

对象数组存储的是对象的引用，而基本数据类型数组则直接存储基本数据类型的值。

```java
//数组必须先分配空间再使用
int[] arr = new int[3];
int[] arr2 = new int[]{1, 2, 3};
int[] arr3 = {1, 2, 3};
String[] str = new String[3];

System.out.println(Arrays.toString(arr));
System.out.println(Arrays.toString(arr2));
System.out.println(Arrays.toString(arr3));
System.out.println(Arrays.toString(str));

//未显示初始化的数组会被执行默认初始化
[0, 0, 0]
[1, 2, 3]
[1, 2, 3]
[null, null, null]
```

### 多维数组 ###

你也可以使用 **new** 分配数组。这是一个使用 **new** 表达式分配的三维数组：

```Java
public class ThreeDWithNew {
  public static void main(String[] args) {
    // 3-D array with fixed length:
    int[][][] a = new int[2][2][4];
    System.out.println(Arrays.deepToString(a));
  }
}
```

这个例子使用 **Arrays.deepToString()** 方法，将多维数组转换成 **String** 类型。

### Arrays.fill() 方法 ###

Java 标准库 **Arrays** 类包括一个普通的 **fill()** 方法，该方法将单个值复制到整个数组，或者在对象数组的情况下，将相同的引用复制到整个数组：

```java
int[] ints = new int[10];
int[] ints2 = new int[10];
//Assigns the specified int value
//to each element of the specified array of ints.
Arrays.fill(ints, 2);
//Assigns the specified int value to each element of the specified
//range of the specified array of ints.  The range to be filled
//extends from fromIndex, inclusive, to toIndex, exclusive. 
Arrays.fill(ints2, 2,5,8);

System.out.println(Arrays.toString(ints));
System.out.println(Arrays.toString(ints2));
//[2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
//[0, 0, 8, 8, 8, 0, 0, 0, 0, 0]
```

### Arrays.setAll() 方法 ###

在 Java 8中，Array.setAll() 使用一个生成器并生成不同的值，可以选择基于数组的索引元素（通过访问当前索引，生成器可以读取数组值并对其进行修改）。

 **static Arrays.setAll()** 的重载签名为：

- **void setAll(int[] a, IntUnaryOperator gen)**
- **void setAll(long[] a, IntToLongFunction gen)**
- **void setAll(double[] a, IntToDoubleFunction gen)**
- **void setAll(T[] a, IntFunction<? extends T> gen)**

除了 **int** , **long** , **double** 有特殊的版本，其他的一切都由泛型版本处理。生成器不是 **Supplier** 因为它们不带参数，并且必须将 **int** 数组索引作为参数。

传递给 **Arrays.setAll()** 的生成器函数可以使用它接收到的数组索引修改现有的数组元素:

```java
class ModifyExisting {
    public static void main(String[] args) {

        double[] da = new double[]{1.0, 2.3, 5.65, 89.54};

        Arrays.setAll(da, n -> da[n] * 100);

        System.out.println(Arrays.toString(da));
        //[100.0, 229.99999999999997, 565.0, 8954.0]
    }
}
```

### 并行数组 ###

在你深刻理解所有的问题之前，您可以很容易地编写比非并行版本运行速度更慢的代码，并行编程看起来更像是一门艺术而非科学。**用简单的方法编写代码，不要开始处理并行性，除非它成为一个问题。**

最好从数据的角度来考虑并行性。对于大量数据(以及可用的额外处理器)，并行可能会有所帮助。但您也可能使事情变得更糟。

```java
class ParallelSetAll {
    static final int SIZE = 10_000_000;

    static void intArray() {
        int[] ia = new int[SIZE];
        Arrays.setAll(ia, i -> i + 1);
        Arrays.parallelSetAll(ia, i -> i + 2);
    }

    static void longArray() {
        long[] la = new long[SIZE];
        Arrays.setAll(la, i -> i + 1);
        Arrays.parallelSetAll(la, i -> i + 2);
    }

    public static void main(String[] args) {
        intArray();
        longArray();
    }
}
```

JDK 中的 Arrays.parallelSetAll() 方法源码：

```java
public static <T> void parallelSetAll(T[] array, 
IntFunction<? extends T> generator) {
    Objects.requireNonNull(generator);
    IntStream.range(0, array.length).parallel().
    forEach(i -> { array[i] = generator.apply(i); });
}
```

### Arrays 工具类 ###

该类包含许多其他有用的 **静态** 程序方法：

- **asList()**: 获取任何序列或数组，并将其转换为一个 **列表集合**（这个集合不能改变大小）。
- **copyOf()**：以新的长度创建现有数组的新副本。
- **copyOfRange()**：创建现有数组的一部分的新副本。
- **equals()**：比较两个数组是否相等。
- **deepEquals()**：多维数组的相等性比较。
- **stream()**：生成数组元素的流。
- **hashCode()**：生成数组的哈希值(您将在附录中了解这意味着什么:理解equals()和hashCode())。
- **deepHashCode()**: 多维数组的哈希值。
- **sort()**：排序数组
- **parallelSort()**：对数组进行并行排序，以提高速度。
- **binarySearch()**：在已排序的数组中查找元素。
- **parallelPrefix()**：使用提供的函数并行累积(以获得速度)。基本上，就是数组的reduce()。
- **spliterator()**：从数组中产生一个Spliterator;这是本书没有涉及到的流的高级部分。
- **toString()**：为数组生成一个字符串表示。你在整个章节中经常看到这种用法。
- **deepToString()**：为多维数组生成一个字符串。你在整个章节中经常看到这种用法。对于所有基本类型和对象，所有这些方法都是重载的。

### 数组比较 ###

**数组** 提供了 **equals()** 来比较一维数组，以及 **deepEquals()** 来比较多维数组。对于所有原生类型和对象，这些方法都是重载的。

数组相等的含义：数组必须有相同数量的元素，并且每个元素必须与另一个数组中的对应元素相等，对每个元素使用 **equals()**。

### 流和数组 ###

**stream()** 方法很容易从某些类型的数组中生成元素流。

```JAVA
public class StreamFromArray {
    public static void main(String[] args) {
        int[] ia = new int[0];
        Arrays.stream(ia).skip(3).limit(5)
                .map(i -> i + 10).forEach(System.out::println);
        Arrays.stream(new long[10]);
        Arrays.stream(new double[10]);
        Arrays.stream(new String[10]);
    }
}
```

只有“原生类型” **int**、**long** 和 **double** 可以与 **Arrays.stream()** 一起使用;对于其他的，您必须以某种方式获得一个包装类型的数组。

通常，将数组转换为流来生成所需的结果要比直接操作数组容易得多。请注意，即使流已经“用完”(您不能重复使用它)，您仍然拥有该数组，因此您可以以其他方式使用它----包括生成另一个流。

### 数组排序 ###

对于基本数据类型的数组，Arrays.sort() 使用的是**双基准快速排序**（Dual-Pivot Quicksort）；

对于引用数据类型的数组，Arrays.sort() 使用的是优化的合并排序。

Java有两种方式提供比较功能。第一种是通过实现 **Comparable** 接口，重写方法 **compareTo()**。第二种是通过实现 **Comparator** 接口，重写方法 **compare()**。

```java
class CompType implements Comparable<CompType> {
    private static int count = 1;
    private static SplittableRandom r = new SplittableRandom(47);
    private int i;
    private int j;

    private CompType(int n1, int n2) {
        i = n1;
        j = n2;
    }

    public static CompType get() {
        return new CompType(r.nextInt(100), r.nextInt(100));
    }

    public static void main(String[] args) {
        CompType[] a = new CompType[12];
        Arrays.setAll(a, n -> get());

        System.out.println("Before sorting \n"+Arrays.toString(a));
        Arrays.sort(a);
        System.out.println("After sorting \n"+Arrays.toString(a));
    }

    @Override
    public String toString() {
        String result = "[i = " + i + ", j = " + j + "]";
        if (count++ % 3 == 0) {
            result += "\n";
        }
        return result;
    }

    @Override
    public int compareTo(CompType rv) {
        return (Integer.compare(i, rv.i));
    }
}
```

**get()** 方法通过使用随机值初始化CompType对象来构建它们。在 **main()** 中，**get()** 与 **Arrays.setAll()** 一起使用，以填充一个 **CompType类型** 数组，然后对其排序。如果没有实现 **Comparable接口**，那么当您试图调用 **sort()** 时，您将在运行时获得一个 **ClassCastException** 。这是因为 **sort()** 将其参数转换为 **Comparable类型**。

总结：应该“优先选择集合而不是数组”。只有当您证明性能是一个问题（并且切换到一个数组实际上会有很大的不同）时，才应该重构到数组。





# 枚举 #

### 枚举的定义 ###

使用 **enum** 关键字定义枚举类，它被编译器编译为 `final class Xxx extends Enum { … }`

```java
//定义 color 枚举类
public enum Color {
    RED, GREEN, BLUE;
}

//编译器编译出的 class 文件
public final class Color extends Enum<Color> {//继承自 Enum，标记为 final 
    // 每个实例均为全局唯一:
    public static final Color RED = new Color();
    public static final Color GREEN = new Color();
    public static final Color BLUE = new Color();
    // private构造方法，确保外部无法调用new操作符:
    private Color() {}
}
```

此外，还可以在定义枚举的时候添加属性：

```java
public enum Weekday {
    MON(1, "星期一"), 
    TUE(2, "星期二"), 
    WED(3, "星期三"), 
    THU(4, "星期四"), 
    FRI(5, "星期五"), 
    SAT(6, "星期六"), 
    SUN(0, "星期日");

    public final int dayValue;
    private final String chinese;

    private Weekday(int dayValue, String chinese) {
        this.dayValue = dayValue;
        this.chinese = chinese;
    }

    @Override
    public String toString() {
        return this.chinese;
    }
}
```

### enum 的方法 ###

因为 enum 是一个 class ，每个枚举的值都是 class 实例，因此，这些实例有一些方法：

- name() 返回常量名

```java
String s = Color.RED.name(); // "RED"
```

- ordinal() 返回定义的常量的顺序，从 0 开始计数

```java
int n = Color.GREEN.ordinal(); // 1
```

- 默认情况下，对枚举常量调用 toString() 会返回和 name() 一样的字符串。但是，toString() 可以被覆写，而 name() 则不行。

#### enum 的特点 ####

- 定义的 `enum` 类型总是继承自 `java.lang.Enum` （不能再继承其他类了，因为 Java 不支持多继承），且无法被继承（因为编译后被 final 修饰）；
- 只能定义出 `enum` 的实例，而无法通过 `new` 操作符创建 `enum` 的实例（构造方法私有）；
- 定义的每个实例都是引用类型的唯一实例（ jvm 中只存在一份）；
- 可以将 `enum` 类型用于 `switch` 语句，因为枚举类天生具有类型信息和有限个枚举常量，所以比 `int` 、 `String` 类型更适合用在 `switch` 语句中。

#### enum 的比较 ####

使用 `enum` 定义的枚举类是一种引用类型。一般来说，引用类型比较，要使用 equals() 方法，如果使用 `==` 比较，它比较的是两个引用类型的引用是否指向同一个对象。因此，引用类型比较，要始终使用 `equals()` 方法，但 `enum` 类型可以例外，这是因为 `enum` 类型的每个常量在 JVM 中只有一个唯一实例，所以可以直接用 `==` 比较。

```Java
if (day == Weekday.FRI) {
    // ok!
   }
if (day.equals(Weekday.SUN)) { 
    // ok, but more code!
   }
```

### 随机选择枚举 ###

```java
public class Enums {
    private static Random rand = new Random(47);

    public static <T extends Enum<T>> T random(Class<T> ec) {
        return random(ec.getEnumConstants());
    }

    public static <T> T random(T[] values) {
        return values[rand.nextInt(values.length)];
    }
}
```

自限定泛型 <T extends Enum<T>> 表示 T 是一个 enum 实例。而将 Class<T> 作为参数的话，我们就可以利用 Class 对象得到 enum 实例的数组了。重载后的 random() 方法只需使用 T[] 作为参数，因为它并不会调用 Enum 上的任何操作，它只需从数组中随机选择一个元素即可。这样，最终的返回类型正是 enum 的类型。

下面是 random() 方法的一个简单示例：

```java
enum Activity { 
    SITTING, LYING, STANDING, HOPPING,
    RUNNING, DODGING, JUMPING, FALLING, FLYING }

public class RandomTest {
    public static void main(String[] args) {
        for(int i = 0; i < 20; i++)
            System.out.print(
                    Enums.random(Activity.class) + " ");
    }
}
```

### 使用接口组织枚举 ###

在一个接口的内部，创建实现该接口的枚举，以此将元素进行分组，可以达到将枚举元素分类组织的目的。举例来说，假设你想用 enum 来表示不同类别的食物，同时还希望每个 enum 元素仍然保持 Food 类型。那可以这样实现：

```java
public interface Food {
    enum Appetizer implements Food {//开胃菜
        SALAD, SOUP, SPRING_ROLLS;
    }
    enum MainCourse implements Food {//主食
        LASAGNE, BURRITO, PAD_THAI,
        LENTILS, HUMMOUS, VINDALOO;
    }
    enum Dessert implements Food {//饭后甜点
        TIRAMISU, GELATO, BLACK_FOREST_CAKE,
        FRUIT, CREME_CARAMEL;
    }
    enum Coffee implements Food {//咖啡
        BLACK_COFFEE, DECAF_COFFEE, ESPRESSO,
        LATTE, CAPPUCCINO, TEA, HERB_TEA;
    }
}
```

想创建一个“枚举的枚举”，那么可以创建一个新的 enum，然后用其实例包装 Food 中的每一个 enum 类：

```java
public enum Meal2 {
    APPETIZER(Food.Appetizer.class),
    MAINCOURSE(Food.MainCourse.class),
    DESSERT(Food.Dessert.class),
    COFFEE(Food.Coffee.class);
    
    private Food[] values;
    private Meal2(Class<? extends Food> kind) {
        values = kind.getEnumConstants();
    }
    public interface Food {
        enum Appetizer implements Food {
            SALAD, SOUP, SPRING_ROLLS;
        }
        enum MainCourse implements Food {
            LASAGNE, BURRITO, PAD_THAI,
            LENTILS, HUMMOUS, VINDALOO;
        }
        enum Dessert implements Food {
            TIRAMISU, GELATO, BLACK_FOREST_CAKE,
            FRUIT, CREME_CARAMEL;
        }
        enum Coffee implements Food {
            BLACK_COFFEE, DECAF_COFFEE, ESPRESSO,
            LATTE, CAPPUCCINO, TEA, HERB_TEA;
        }
    }
    
    public Food randomSelection() {
        return Enums.random(values);
    }
    
    public static void main(String[] args) {
        for(int i = 0; i < 5; i++) {
            for(Meal2 meal : Meal2.values()) {
                Food food = meal.randomSelection();
                System.out.println(food);
            }
            System.out.println("***");
        }
    }
}
```

getEnumConstants() 方法，可以从该 Class 对象中取得某个 Food 子类的所有 enum 实例。通过 meal.randomSelection() 方法随机地选择一个 Food，我们便能够生成一份菜单。

### 使用 EnumSet  ###

EnumSet 是一个抽象类，其子类有两个，一个是 RegularEnumSet（当集合元素 <= 64），另一个是 JumboEnumSet（当集合元素 > 64）。在实现上，它们将一个 long 值作为**位向量**，所以 EnumSet 非常快速高效。使用 EnumSet 的优点是，它在说明一个二进制位是否存在时，具有更好的表达能力，并且无需担心性能。

```java
public class EnumSetTest {
    enum Season {
        SPRING,SUMMER,FALL,WINTER
    }

    public static void main(String[] args) {
        EnumSet<Season> enumSet = EnumSet.noneOf(Season.class);
        enumSet.add(Season.SPRING);
        enumSet.add(Season.SUMMER);
        System.out.println(enumSet);
    }
}
```

EnumSet 在创建集合对象的时候总会调用 **EnumSet.noneOf(Class)** 方法，该方法会一次性缓存枚举类的所有枚举，在后续的任何操作都只是对 **long** 类型的**位向量**进行对应的移位操作。

### 使用 EnumMap ###

EnumMap 是一种特殊的 Map，它要求其中的键（key）必须来自一个 enum，由于 enum 本身的限制，所以 EnumMap 在内部可由数组实现。

下面的例子演示了**命令模式**的用法。一般来说，命令模式首先需要一个只有单一方法的接口，然后从该接口实现具有各自不同的行为的多个子类。

```java
public class EnumMapTest {
    interface Command {
        void action();
    }

    enum Season {
        SPRING, SUMMER, FALL, WINTER
    }

    public static void main(String[] args) {
        EnumMap<Season, Command> em = new EnumMap<>(Season.class);

        em.put(Season.SPRING, () -> System.out.println("Spring is coming !"));
        em.put(Season.SUMMER, () -> System.out.println("oh, it's so hot!"));

        for (Map.Entry<Season, Command> e : em.entrySet()) {
            System.out.print(e.getKey() + ": ");
            e.getValue().action();
        }
        try { // If there's no value for a particular key:
            em.get(Season.WINTER).action();
        } catch (Exception e) {
            System.out.println("Expected: " + e);
        }
    }
}
```

与 EnumSet 一样，enum 实例定义时的次序决定了其在 EnumMap 中的顺序。

main() 方法的最后部分说明，enum 的每个实例作为一个键，总是存在的。但是，如果你没有为这个键调用 put() 方法来存入相应的值的话，其对应的值就是 null。

### 常量特定方法 ###

Java 的 enum 允许程序员编写实例方法，从而为每个 enum 实例赋予各自不同的行为。要实现常量相关的方法，你需要为 enum 定义一个或多个 abstract 方法，然后为每个 enum 实例实现该抽象方法。参考下面的例子：

```java
public enum ConstantSpecificMethod {
    DATE_TIME {
        @Override
        String getInfo() {
            return DateFormat.getDateInstance()
                            .format(new Date());
        }
    },
    CLASSPATH {
        @Override
        String getInfo() {
            return System.getenv("CLASSPATH");
        }
    },
    VERSION {
        @Override
        String getInfo() {
            return System.getProperty("java.version");
        }
    };
    abstract String getInfo();
    public static void main(String[] args) {
        for(ConstantSpecificMethod csm : values())
            System.out.println(csm.getInfo());
    }
}
```

通过相应的 enum 实例，我们可以调用其上的方法。这通常也称为表驱动的代码（table-driven code，和**命令模式**很相似）。

### 多路分发 ###

如果一个系统要执行数学表达式，我们可能会声明 Number.plus(Number) 等，其中 Number 是各种数字对象的超类。然而，当你声明 a.plus(b) 时，你并不知道 a 或 b 的确切类型，那你如何能让它们正确地交互呢？

**Java 只支持单路分发**。也就是说，如果要执行的操作包含了不止一个类型未知的对象时，那么 Java 的动态绑定机制只能处理其中一个的类型。

解决这个问题的办法就是**多路分发**，**多态只能发生在方法调用时**，要利用多路分发，程序员必须为每一个类型提供一个实际的方法调用，如果你要处理两个不同的类型体系，就需要为每个类型体系执行一个方法调用。

#### 基于接口的多路分发 ####

在下面的例子中（实现了 “石头、剪刀、布”游戏，也称为 RoShamBo）对应的方法是 compete() 和 eval()，二者都是同一个类型的成员，它们可以产生三种 Outcome 实例中的一个作为结果：

```java
enum Outcome {
    WIN,
    LOSE,
    DRAW
}

interface Item {
    Outcome compete(Item it);

    Outcome eval(Paper p);

    Outcome eval(Scissors s);

    Outcome eval(Rock r);
}

class Paper implements Item {
    @Override
    public Outcome compete(Item it) {
        return it.eval(this);
    }

    @Override
    public Outcome eval(Paper p) {
        return DRAW;
    }

    @Override
    public Outcome eval(Scissors s) {
        return WIN;
    }

    @Override
    public Outcome eval(Rock r) {
        return LOSE;
    }

    @Override
    public String toString() {
        return "Paper";
    }
}

class Scissors implements Item {
    @Override
    public Outcome compete(Item it) {
        return it.eval(this);
    }

    @Override
    public Outcome eval(Paper p) {
        return LOSE;
    }

    @Override
    public Outcome eval(Scissors s) {
        return DRAW;
    }

    @Override
    public Outcome eval(Rock r) {
        return WIN;
    }

    @Override
    public String toString() {
        return "Scissors";
    }
}

class Rock implements Item {
    @Override
    public Outcome compete(Item it) {
        return it.eval(this);
    }

    @Override
    public Outcome eval(Paper p) {
        return WIN;
    }

    @Override
    public Outcome eval(Scissors s) {
        return LOSE;
    }

    @Override
    public Outcome eval(Rock r) {
        return DRAW;
    }

    @Override
    public String toString() {
        return "Rock";
    }
}

public class RoShamBo1 {
    private static Random rand = new Random(47);

    public static Item newItem() {
        switch (rand.nextInt(3)) {
            default:
            case 0:
                return new Scissors();
            case 1:
                return new Paper();
            case 2:
                return new Rock();
        }
    }

    public static void match(Item a, Item b) {
        System.out.println(
                a + " vs. " + b + ": " + a.compete(b));
    }

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            match(newItem(), newItem());
        }
    }
}
```

Item 是这几种类型的接口，将会被用作多路分发。RoShamBo1.match() 有两个 Item 参数，通过调用 Item.compete() 方法开始两路分发。要判定 a 的类型，分发机制会在 a 的实际类型的 compete() 内部起到分发的作用。compete() 方法通过调用 eval() 来为另一个类型实现第二次分法。将自身（this）作为参数调用 eval()，能够调用重载过的 eval() 方法，这能够保留第一次分发的类型信息。当第二次分发完成时，你就能够知道两个 Item 对象的具体类型了。

#### 使用 enum 分发 ####

```java
class Enums {
    private static Random rand = new Random(47);

    public static <T extends Enum<T>> T random(Class<T> ec) {
        return random(ec.getEnumConstants());
    }

    public static <T> T random(T[] values) {
        return values[rand.nextInt(values.length)];
    }
}

enum Outcome {
    WIN,
    LOSE,
    DRAW
}

interface Competitor<T extends Competitor<T>> {
    Outcome compete(T competitor);
}

class RoShamBo {
    static <T extends Competitor<T>> void match(T a, T b) {
        System.out.println(
                a + " vs. " + b + ": " + a.compete(b));
    }

    static <T extends Enum<T> & Competitor<T>> void play(Class<T> rsbClass, int size) {
        for (int i = 0; i < size; i++) {
            match(Enums.random(rsbClass), Enums.random(rsbClass));
        }
    }
}

enum RoShamBo2 implements Competitor<RoShamBo2> {
    PAPER(DRAW, LOSE, WIN),
    SCISSORS(WIN, DRAW, LOSE),
    ROCK(LOSE, WIN, DRAW);

    Outcome vPAPER, vSCISSORS, vROCK;

    RoShamBo2(Outcome paper,
              Outcome scissors, Outcome rock) {
        this.vPAPER = paper;
        this.vSCISSORS = scissors;
        this.vROCK = rock;
    }

    @Override
    public Outcome compete(RoShamBo2 it) {
        switch (it) {
            default:
            case PAPER:
                return vPAPER;
            case SCISSORS:
                return vSCISSORS;
            case ROCK:
                return vROCK;
        }
    }

    public static void main(String[] args) {
        RoShamBo.play(RoShamBo2.class, 20);
    }
}
```

#### 使用常量相关的方法 ####

常量相关的方法允许我们为每个 enum 实例提供方法的不同实现，这使得常量相关的方法似乎是实现多路分发的完美解决方案。

```java
public enum RoShamBo3 implements Competitor<RoShamBo3> {
    PAPER {
        @Override
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default: // To placate the compiler
                case PAPER: return DRAW;
                case SCISSORS: return LOSE;
                case ROCK: return WIN;
            }
        }
    },
    SCISSORS {
        @Override
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default:
                case PAPER: return WIN;
                case SCISSORS: return DRAW;
                case ROCK: return LOSE;
            }
        }
    },
    ROCK {
        @Override
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default:
                case PAPER: return LOSE;
                case SCISSORS: return WIN;
                case ROCK: return DRAW;
            }
        }
    };
    @Override
    public abstract Outcome compete(RoShamBo3 it);
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo3.class, 20);
    }
}
```

#### 使用 EnumMap 进行分发 ####

使用 EnumMap 能够实现“真正的”两路分发。

```java
import static enums.Outcome.*;
enum RoShamBo5 implements Competitor<RoShamBo5> {
    PAPER, SCISSORS, ROCK;
    static EnumMap<RoShamBo5,EnumMap<RoShamBo5,Outcome>>
            table = new EnumMap<>(RoShamBo5.class);
    static {
        for(RoShamBo5 it : RoShamBo5.values())
            table.put(it, new EnumMap<>(RoShamBo5.class));
        initRow(PAPER, DRAW, LOSE, WIN);
        initRow(SCISSORS, WIN, DRAW, LOSE);
        initRow(ROCK, LOSE, WIN, DRAW);
    }
    static void initRow(RoShamBo5 it,
                        Outcome vPAPER, Outcome vSCISSORS, Outcome vROCK) {
        EnumMap<RoShamBo5,Outcome> row =
                RoShamBo5.table.get(it);
        row.put(RoShamBo5.PAPER, vPAPER);
        row.put(RoShamBo5.SCISSORS, vSCISSORS);
        row.put(RoShamBo5.ROCK, vROCK);
    }
    @Override
    public Outcome compete(RoShamBo5 it) {
        return table.get(this).get(it);
    }
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo5.class, 20);
    }
}
```

#### 使用二维数组 ####

我们还可以进一步简化实现两路分发的解决方案。我们注意到，每个 enum 实例都有一个固定的值（基于其声明的次序），并且可以通过 ordinal() 方法取得该值。因此我们可以使用二维数组，将竞争者映射到竞争结果。

```java
enum RoShamBo6 implements Competitor<RoShamBo6> {
    PAPER, SCISSORS, ROCK;

    private static Outcome[][] table = {
            {DRAW, LOSE, WIN}, // PAPER
            {WIN, DRAW, LOSE}, // SCISSORS
            {LOSE, WIN, DRAW}, // ROCK
    };

    @Override
    public Outcome compete(RoShamBo6 other) {
        return table[this.ordinal()][other.ordinal()];
    }

    public static void main(String[] args) {
        RoShamBo.play(RoShamBo6.class, 20);
    }
}
```

事实上，以上所有的解决方案只是各种不同类型的表罢了。不过，分析各种表的表现形式，找出最适合的那一种，还是很有价值的。对于某类问题而言，“**表驱动式编码**”的概念具有非常强大的功能。



## 注解 ##

注解（也被称为元数据）为我们在代码中添加信息提供了一种形式化的方式，使我们可以在稍后的某个时刻更容易的使用这些数据。

通过使用注解，你可以将元数据保存在 Java 源代码中。并拥有如下优势：简单易读的代码，编译器类型检查，使用 annotation API 为自己的注解构造处理工具。

 **java.lang** 包中的标准注解：

- **@Override**：表示当前的方法定义将覆盖基类的方法。如果你不小心拼写错误，编译器就会发出错误提示。
- **@Deprecated**：如果使用该注解的元素被调用，编译器就会发出警告信息。
- **@SuppressWarnings**：关闭不当的编译器警告信息。
- **@SafeVarargs**：在 Java 7 中加入用于禁止对具有泛型 varargs 参数的方法或构造函数的调用方发出警告。
- **@FunctionalInterface**：Java 8 中加入用于表示类型声明为函数式接口。

### 定义注解 ###

在下面的例子中，使用 `@Test` 对 `testExecute()` 进行注解。该注解本身不做任何事情，但是编译器要保证其类路径上有 `@Test` 注解的定义。

```java
public class Testable {
    public void execute() {
        System.out.println("Executing..");
    }
    @Test
    void testExecute() { execute(); }
}
```

如下是 `Test` 注解的定义，它看起来很像接口的定义。事实上，它们和其他 Java 接口一样，也会被编译成 class 文件。除了 @ 符号之外， `@Test` 的定义看起来更像一个空接口。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test { }
```

注解的定义也需要一些元注解（meta-annoation），比如 `@Target` 和 `@Retention`。

注解通常会包含一些表示特定值的元素。当分析处理注解的时候，程序或工具可以利用这些值。注解的元素看起来就像接口的方法，但是可以为其指定默认值。

不包含任何元素的注解称为**标记注解**（marker annotation），例如上例中的 `@Test` 就是标记注解。

#### 元注解 ####

Java 语言中目前有 5 种标准注解，以及 5 种元注解。元注解用于定义其他的注解。

| 注解        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| @Target     | 表示注解可以用于哪些地方。可能的 **ElementType** 参数包括： <br/>**CONSTRUCTOR**：构造器的声明<br/> **FIELD**：字段声明（包括 enum 实例）<br/> **LOCAL_VARIABLE**：局部变量声明 <br/> **METHOD**：方法声明 <br/> **PACKAGE**：包声明 <br/> **PARAMETER**：参数声明 <br/> **TYPE**：类、接口（包括注解类型）或者 enum 声明 |
| @Retention  | 表示注解信息保存的时长。可选的 **RetentionPolicy** 参数包括： <br/>**SOURCE**：注解将被编译器丢弃。 <br/> **CLASS**：注解在 class 文件中可用，但是会被 VM 丢弃。<br/> **RUNTIME**：VM 将在运行期也保留注解，因此可以通过反射机制读取注解的信息。 |
| @Documented | 将此注解保存在 Javadoc 中                                    |
| @Inherited  | 允许子类继承父类的注解                                       |
| @Repeatable | 允许一个注解可以被使用一次或者多次（Java 8）。               |

大多数时候，程序员定义自己的注解，并编写自己的处理器来处理他们。

### 编写注解处理器 ###

如果没有用于读取注解的工具，那么注解将不会工作。使用注解中一个很重要的部分就是，创建与使用**注解处理器**。Java 拓展了反射机制的 API 用于帮助你创造这类工具。同时他还提供了 javac 编译器钩子在编译时使用注解。

下面是一个简单的注解，拥有两个元素 **id** 和 **description**，**description** 属性拥有一个 **default** 值。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    int id();
    String description() default "no description";
}
```

在下面的类中，有三个方法被 @UseCase 标注：

```java
public class PasswordUtils {
    @UseCase(id = 47, description =
            "Passwords must contain at least one numeric")
    public boolean validatePassword(String passwd) {
        return (passwd.matches("\\w*\\d\\w*"));
    }
    @UseCase(id = 48)
    public String encryptPassword(String passwd) {
        return new StringBuilder(passwd)
                .reverse().toString();
    }
    @UseCase(id = 49, description =
            "New passwords can't equal previously used ones")
    public boolean checkForNewPassword(
            List<String> prevPasswords, String passwd) {
        return !prevPasswords.contains(passwd);
    }
}
```

下面是一个非常简单的注解处理器，我们用它来读取被注解的 **PasswordUtils** 类，并且使用反射机制来寻找 **@UseCase** 标记。给定一组 **id** 值，然后列出在 **PasswordUtils** 中找到的用例，以及缺失的用例。

```java
public class UseCaseTracker {
    static void trackUseCases(List<Integer> useCases, Class<?> cl) {
        for(Method m : cl.getDeclaredMethods()) {
            UseCase uc = m.getAnnotation(UseCase.class);
            if(uc != null) {
                System.out.println("Found Use Case " +
                        uc.id() + "\n " + uc.description());
                useCases.remove(Integer.valueOf(uc.id()));
            }
        }
        useCases.forEach(i ->
                System.out.println("Missing use case " + i));
    }
    public static void main(String[] args) {
        List<Integer> useCases = IntStream.range(47, 51)
                .boxed().collect(Collectors.toList());
        trackUseCases(useCases, PasswordUtils.class);
    }
}
```

这个程序用了两个反射的方法：`getDeclaredMethods()` 和 `getAnnotation()`，它们都属于 **AnnotatedElement** 接口（**Class**，**Method** 与 **Field** 类都实现了该接口）。`getAnnotation()` 方法返回指定类型的注解对象。

#### 注解元素 ####

在定义 **@UseCase** 时包含 int 元素 **id** 和 String 元素 **description**。注解元素可用的类型如下：

- 所有基本类型（int、float、boolean等）
- String
- Class
- enum
- Annotation
- 以上类型的数组

如果你使用了其他类型，编译器就会报错。**注解也可以作为元素的类型**。

#### 默认值限制 ####

编译器对于元素的默认值有些过于挑剔。首先，元素不能有不确定的值。也就是说，元素要么有默认值，要么就在使用注解时提供元素的值。

这里有另外一个限制：任何非基本类型的元素， 无论是在源代码声明时还是在注解接口中定义默认值时，都不能使用 null 作为其值。这个限制使得处理器很难表现一个元素的存在或者缺失的状态，因为在每个注解的声明中，所有的元素都存在，并且具有相应的值。为了绕开这个约束，可以自定义一些特殊的值，比如空字符串或者负数用于表达某个元素不存在。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SimulatingNull {
    int id() default -1;
    String description() default "";
}// 这是一个在定义注解的习惯用法。
```

#### 注解不支持继承 ####

不能使用 **extends** 关键字来继承 **@interfaces**。

### 使用 javac 处理注解 ###



#### 最简单的处理器 ####



#### 更复杂的处理器 ####



### 基于注解的单元测试 ###



#### 在 @Unit 中使用泛型 ####



#### 实现 @Unit ####





