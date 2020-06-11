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

### 自限定的类型 ###

在 Java 泛型中，有一个似乎经常性出现的惯用法，它相当令人费解：

```java
class SelfBounded<T extends SelfBounded<T>> { }
```

这就像两面镜子彼此照向对方所引起的目眩效果一样，是一种无限反射。**SelfBounded** 类接受泛型参数 **T**，而 **T** 由一个边界类限定，这个边界就是拥有 **T** 作为其参数的 **SelfBounded**。

#### 古怪的循环泛型 ####

含义：创建一个新类，它继承自一个泛型类型，这个泛型类型接受我的类的名字作为其参数。

作用：Java 中的泛型关乎**参数和返回类型**，因此它能够产生使用导出类作为其参数和返回类型的基类。它还能将导出类型用作其域类型，尽管这些将被擦除为 **Object** 的类型。

```java
class BasicHolder<T> {
    T element;

    void set(T arg) {
        element = arg;
    }

    T get() {
        return element;
    }

    void f() {
        System.out.println(element.getClass().getSimpleName());
    }
}

class Subtype extends BasicHolder<Subtype> {
}

class CRGWithBasicHolder {
    public static void main(String[] args) {
        Subtype st1 = new Subtype(), st2 = new Subtype();
        st1.set(st2);
        Subtype st3 = st1.get();

        System.out.println(st2 == st3);
        st1.f();
    }
}
```

注意，这里有些东西很重要：新类 **Subtype** 接受的参数和返回的值具有 **Subtype** 类型而不仅仅是基类 **BasicHolder** 类型。这就是 **CuriouslyRecurringGeneric** 的本质：基类用导出类替代其参数。**这意味着泛型基类变成了一种其所有导出类的公共功能的模版**，但是这些功能对于其所有参数和返回值，将使用导出类型。也就是说，在所产生的类中将使用确切类型而不是基类型。因此，在**Subtype** 中，传递给 `set()` 的参数和从 `get()` 返回的类型都是确切的 **Subtype**。

#### 自限定 ####



#### 参数协变 ####



### 动态类型安全 ###

### 泛型异常 ###

### 混型 ###

#### C++ 中的混型 ####

#### 与接口的混型 ####

#### 使用装饰器模式 ####

#### 与动态代理混合 ####

### 潜在类型机制 ###

### 对缺乏潜在类型机制的补偿 ###

#### 反射 ####

#### 将一个方法应用于序列 ####

### Java8 中的辅助潜在类型 ###

#### 使用 Suppliers 类的通用方法 ####



