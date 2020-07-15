## 集合概述 ##

**java.util** 库提供了一套相当完整的*集合类*（collection classes），也被称作*容器类*（container classes）。其中基本的类型有 **List** 、 **Set** 、 **Queue** 和 **Map**。集合提供了完善的方法来保存对象，可以使用这些工具来解决大量的问题。

集合还有一些其它特性。例如， **Set** 对于每个值都只保存一个对象， **Map** 是一个关联数组，允许将某些对象与其他对象关联起来。Java 集合类都可以自动地调整自己的大小。因此，与数组不同，在编程时，可以将任意数量的对象放置在集合中，而不用关心集合应该有多大。

### 泛型和类型安全的集合 ###

使用泛型，你不仅知道编译器将检查放入集合的对象类型，而且在使用集合中的对象时也可以获得更清晰的语法。

```java
List<Apple> apples = new ArrayList<>();
	apples.add(new Apple());
    // Compile-time error:
    // apples.add(new Orange());
```

### 基本概念 ###

Java集合类库采用“持有对象”（holding objects）的思想，并将其分为两个不同的概念，表示为类库的基本接口：

1. **集合（Collection）** ：一个独立元素的序列，这些元素都服从一条或多条规则。**List** 必须以插入的顺序保存元素， **Set** 不能包含重复元素， **Queue** 按照*排队规则*来确定对象产生的顺序（通常与它们被插入的顺序相同）。
2. **映射（Map）** ： 一组成对的“键值对”对象，允许使用键来查找值。 **ArrayList** 使用数字来查找对象，因此在某种意义上讲，它是将数字和对象关联在一起。 **map** 允许我们使用一个对象来查找另一个对象，它也被称作*关联数组*（associative array），因为它将对象和其它对象关联在一起；或者称作*字典*（dictionary），因为可以使用一个键对象来查找值对象，就像在字典中使用单词查找定义一样。 **Map** 是强大的编程工具。

### 添加元素组 ###

在 **java.util** 包中的 **Arrays** 和 **Collections** 类中都有很多实用的方法，可以在一个 **Collection** 中添加一组元素。

`Arrays.asList(T... a)` 方法接受一个数组或是逗号分隔的元素列表（使用可变参数），并将其转换为 **List** 对象（**这个实现是一个叫 Arrays.ArrayList 的内部类，底层实现是一个没法调整大小的数组，不能 add() 或 remove() 操作，可以修改**）。

`Collections.addAll(Collection<? super T> c, T... elements)` 方法接受一个 **Collection** 对象，以及一个数组或是一个逗号分隔的列表，将其中元素添加到 **Collection** 中。

> If you're adding elements from an **array**, you can `use Collections.addAll(col, arr)`
> Remember that varargs are also done using arrays
>
> If you're adding elements from a **Collection**, use `col.addAll(otherCol)`
> Do *NOT* e.g. `Collections.addAll(col, otherCol.toArray())`



### 集合的打印 ###

必须使用 `Arrays.toString()` 来生成数组的可打印形式。

```java
public class PrintingCollections {
  static Collection fill(Collection<String> collection) {
    collection.add("rat");
    collection.add("cat");
    collection.add("dog");
    collection.add("dog");
    return collection;
  }
  static Map fill(Map<String, String> map) {
    map.put("rat", "Fuzzy");
    map.put("cat", "Rags");
    map.put("dog", "Bosco");
    map.put("dog", "Spot");
    return map;
  }
  public static void main(String[] args) {
    Integer[] integers = new Integer[]{1,2,3,4,5,6};
    System.out.println(Arrays.toString(integers));
      
    System.out.println(fill(new ArrayList<>()));
      
    System.out.println(fill(new HashSet<>()));
      
    System.out.println(fill(new HashMap<>()));
  }
}
/* Output:
[1,2,3,4,5,6]
[rat, cat, dog, dog]
[rat, cat, dog]
{rat=Fuzzy, cat=Rags, dog=Spot}
*/
```

这显示了 Java 集合库中的两个主要类型。它们的区别在于集合中的每个“槽”（slot）保存的元素个数。 **Collection** 类型在每个槽中只能保存一个元素。此类集合包括： **List** ，它以特定的顺序保存一组元素； **Set** ，其中元素不允许重复； **Queue** ，只能在集合一端插入对象，并从另一端移除对象。 **Map** 在每个槽中存放了两个元素，**键和值**。

默认的打印行为，使用集合提供的 `toString()` 方法即可生成可读性很好的结果。 **Collection** 打印出的内容用方括号括住，每个元素由逗号分隔。 **Map** 则由大括号括住，每个键和值用等号连接（键在左侧，值在右侧）。

**ArrayList** 和 **LinkedList** 都是 **List** 的类型，它们都按插入顺序保存元素。两者之间的区别不仅在于执行某些类型的操作时的性能，而且 **LinkedList** 包含的操作多于 **ArrayList** 。

**HashSet** ， **TreeSet** 和 **LinkedHashSet** 是 **Set** 的类型。 **Set** 仅保存每个相同项中的一个，并且不同的 **Set** 实现存储元素的方式也不同。 **HashSet** 使用相当复杂的方法存储元素，这种技术是检索元素的最快方法（通常只关心某事物是否是 **Set** 的成员，而存储顺序并不重要）。如果存储顺序很重要，则可以使用 **TreeSet** ，它将按比较结果的升序保存对象。或 **LinkedHashSet** ，它按照被添加的先后顺序保存对象。

**Map** 使用键来查找对象，所关联的对象称为值，对于每个键， **Map** 只存储一次。这里没有指定（或考虑） **Map** 的大小，因为它会自动调整大小。 **Map** 的三种基本风格： **HashMap** ， **TreeMap** 和 **LinkedHashMap** 。键和值保存在 **HashMap** 中的顺序不是插入顺序，因为 **HashMap** 实现使用了非常快速的算法来控制顺序。 **TreeMap** 通过比较结果的升序来保存键， **LinkedHashMap** 在保持 **HashMap** 查找速度的同时按键的插入顺序保存键。

### 列表 List ###

有两种类型的 **List** ：

- 基本的 **ArrayList** ，擅长随机访问元素，但在 **List** 中间插入和删除元素时速度较慢。
- **LinkedList** ，它通过代价较低的在 **List** 中间进行的插入和删除操作，提供了优化的顺序访问。 **LinkedList** 对于随机访问来说相对较慢，但它具有比 **ArrayList** 更大的特征集。

### 迭代器 Iterators ###

迭代器是一个对象，它在一个序列中移动并选择该序列中的每个对象，而客户端程序员不知道或不关心该序列的底层结构。另外，迭代器通常被称为*轻量级对象*（lightweight object）：创建它的代价小。因此，经常可以看到一些对迭代器有些奇怪的约束。例如，Java 的 **Iterator** 只能单向移动。这个 **Iterator** 只能用来：

1. 使用 `iterator()` 方法要求集合返回一个 **Iterator**。 **Iterator** 将准备好返回序列中的第一个元素。
2. 使用 `next()` 方法获得序列中的下一个元素。
3. 使用 `hasNext()` 方法检查序列中是否还有元素。
4. 使用 `remove()` 方法将迭代器最近返回的那个元素删除。

有了 **Iterator** ，就不必再为集合中元素的数量操心了。这是由 `hasNext()` 和 `next()` 关心的事情。如只想向前遍历 **List** ，并不打算修改 **List** 对象本身，使用 **for-each 循环**语法更加简洁。

现在考虑创建一个 `display()` 方法，它不必知晓集合的确切类型。我们可以使用 **Iterable** 接口，该接口描述了“可以产生 **Iterator** 的任何东西”：

```java
public class CrossCollectionIteration2 {
  public static void display(Iterable<Pet> ip) {
    Iterator<Pet> it = ip.iterator();
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
  }
  public static void main(String[] args) {
    List<Pet> pets = Pets.list(7);
    LinkedList<Pet> petsLL = new LinkedList<>(pets);
    HashSet<Pet> petsHS = new HashSet<>(pets);
    TreeSet<Pet> petsTS = new TreeSet<>(pets);
    display(pets);
    display(petsLL);
    display(petsHS);
    display(petsTS);
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
5:Cymric 2:Cymric 1:Manx 3:Mutt 6:Pug 4:Pug 0:Rat
*/
```

`display()` 方法不包含任何有关它所遍历的序列的类型信息。这也展示了 **Iterator** 的真正威力：**能够将遍历序列的操作与该序列的底层结构分离**。出于这个原因，我们有时会说：迭代器统一了对集合的访问方式。

#### ListLterator ####

**ListIterator** 是一个更强大的 **Iterator** 子类型，它只能由各种 **List** 类生成。 **Iterator** 只能向前移动，而 **ListIterator** 可以双向移动。它可以生成迭代器在列表中指向位置的后一个和前一个元素的索引，并且可以使用 `set()` 方法替换它访问过的最近一个元素。可以通过调用 `listIterator()` 方法来生成指向 **List** 开头处的 **ListIterator** ，还可以通过调用 `listIterator(n)` 创建一个一开始就指向列表索引号为 **n** 的元素处的 **ListIterator** 。

```java
List<String> list = new ArrayList<>();
    list.add("A");
    list.add("B");
    list.add("C");

ListIterator<String> iterator = list.listIterator();
while (iterator.hasNext()) {
    System.out.println(iterator.nextIndex()+" "+iterator.next());
}

while (iterator.hasPrevious()) {
    System.out.println(iterator.previousIndex()+" "+iterator.previous());
}
```

### 链表 LinkedList ###

**LinkedList** 也像 **ArrayList** 一样实现了基本的 **List** 接口，但它在 **List** 中间执行插入和删除操作时比 **ArrayList** 更高效。然而,它在随机访问操作效率方面却要逊色一些。

**LinkedList 还添加了一些方法，使其可以被用作栈、队列或双端队列（deque）** 。在这些方法中，有些彼此之间可能只是名称有些差异，或者只存在些许差异，以使得这些名字在特定用法的上下文环境中更加适用（特别是在 **Queue** 中）。例如：

- `getFirst()` 和 `element()` 是相同的，它们都返回列表的头部（第一个元素）而并不删除它，如果 **List** 为空，则抛出 **NoSuchElementException** 异常。 `peek()` 方法与这两个方法只是稍有差异，它在列表为空时返回 **null** 。
- `removeFirst()` 和 `remove()` 也是相同的，它们删除并返回列表的头部元素，并在列表为空时抛出 **NoSuchElementException** 异常。 `poll()` 稍有差异，它在列表为空时返回 **null** 。
- `addFirst()` 在列表的开头插入一个元素。
- `offer()` 与 `add()` 和 `addLast()` 相同。 它们都在列表的尾部（末尾）添加一个元素。
- `removeLast()` 删除并返回列表的最后一个元素。

### 堆栈 Stack ###

堆栈是“后进先出”（LIFO）集合。它有时被称为*叠加栈*（pushdown stack），因为最后“压入”（push）栈的元素，第一个被“弹出”（pop）栈。

Java 1.0 中附带了一个 **Stack** 类，结果设计得很糟糕。

Java 6 添加了 **ArrayDeque** ，其中包含直接实现堆栈功能的方法。即使它是作为一个堆栈在使用，我们仍然必须将其声明为 **Deque** 。有时一个名为 **Stack** 的类更能把事情讲清楚：

```java
public class Stack<T> {
    private Deque<T> storage = new ArrayDeque<>();
	//压栈
    public void push(T v) {
        storage.push(v);
    }
	//获取栈顶元素
    public T peek() {
        return storage.peek();
    }
	//出栈
    public T pop() {
        return storage.pop();
    }

    public boolean isEmpty() {
        return storage.isEmpty();
    }

    @Override
    public String toString() {
        return storage.toString();
    }
}
```

类名称后面的 **<>** 告诉编译器这是一个参数化类型，而其中的类型参数 **T** 会在使用类时被实际类型替换。基本上，这个类是在声明“我们在定义一个可以持有 **T** 类型对象的 **Stack** 。” **Stack** 是使用 **ArrayDeque** 实现的，而 **ArrayDeque** 也被告知它将持有 **T** 类型对象。注意， `push()` 接受类型为 **T** 的对象，而 `peek()` 和 `pop()` 返回类型为 **T** 的对象。 `peek()` 方法将返回栈顶元素，但并不将其从栈顶删除，而 `pop()` 删除并返回顶部元素。

如果只需要栈的行为，那么使用继承是不合适的，因为这将产生一个具有 **ArrayDeque** 的其它所有方法的类中将会看到。 **Java 1.0** 设计者在创建 **java.util.Stack** 时，就犯了这个错误）。使用组合，可以选择要公开的方法以及如何命名它们。

### 集合 Set ###

**Set** 不保存重复的元素。 如果试图将相同对象的多个实例添加到 **Set** 中，那么它会阻止这种重复行为。 **Set** 最常见的用途是测试归属性，可以很轻松地询问某个对象是否在一个 **Set** 中。因此，查找通常是 **Set** 最重要的操作，因此通常会选择 **HashSet** 实现，该实现针对快速查找进行了优化。

**Set** 具有与 **Collection** 相同的接口，因此没有任何额外的功能，不像前面两种不同类型的 **List** 那样。实际上， **Set** 就是一个 **Collection** ，只是行为不同。（这是继承和多态思想的典型应用：表现不同的行为。）

```java
public class SetOfInteger {
  public static void main(String[] args) {
    Random rand = new Random(47);
    Set<Integer> intset = new HashSet<>();
    for(int i = 0; i < 1000; i++)
      intset.add(rand.nextInt(30));
    System.out.println(intset);
  }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 29]
*/
```

早期 Java 版本中的 **HashSet** 产生的输出没有可辨别的顺序。这是因为出于对速度的追求， **HashSet** 使用了散列。由 **HashSet** 维护的顺序与 **TreeSet** 或 **LinkedHashSet** 不同，因为它们的实现具有不同的元素存储方式。 **TreeSet** 将元素存储在**红-黑树**数据结构中，而 **HashSet** 使用散列函数。 **LinkedHashSet** 因为查询速度的原因也使用了散列，但是看起来使用了链表来维护元素的插入顺序。**Set** 最常见的操作之一是使用 `contains()` 测试成员归属性。

### 映射 Map ###

将对象映射到其他对象的能力是解决编程问题的有效方法。下面示例将使用一个 **String** 描述来查找 **Pet** 对象。它还展示了通过使用 `containsKey()` 和 `containsValue()` 方法去测试一个 **Map** ，以查看它是否包含某个键或某个值：

```java
public class PetMap {
  public static void main(String[] args) {
    Map<String, Pet> petMap = new HashMap<>();
    petMap.put("My Cat", new Cat("Molly"));
    petMap.put("My Dog", new Dog("Ginger"));
    petMap.put("My Hamster", new Hamster("Bosco"));
    System.out.println(petMap);
    Pet dog = petMap.get("My Dog");
    System.out.println(dog);
    System.out.println(petMap.containsKey("My Dog"));
    System.out.println(petMap.containsValue(dog));
  }
}
/* Output:
{My Dog=Dog Ginger, My Cat=Cat Molly, My
Hamster=Hamster Bosco}
Dog Ginger
true
true
*/
```

**Map** 与数组和其他的 **Collection** 一样，可以轻松地扩展到多个维度，只需要创建一个值为 **Map** 的 **Map**（这些 **Map** 的值可以是其他集合，甚至是其他 **Map**）。因此，能够很容易地将集合组合起来以快速生成强大的数据结构。

### 队列 Queue ###

队列是一个典型的“先进先出”（FIFO）集合。 即从集合的一端放入事物，再从另一端去获取它们，事物放入集合的顺序和被取出的顺序是相同的。队列通常被当做一种可靠的将对象从程序的某个区域传输到另一个区域的途径。队列在**并发编程**中尤为重要，因为它们可以安全地将对象从一个任务传输到另一个任务。

**LinkedList** 实现了 **Queue** 接口，并且提供了一些方法以支持队列行为，因此 **LinkedList** 可以用作 **Queue** 的一种实现。 

#### 优先级队列 PriorityQueue ####

先进先出（FIFO）描述了最典型的*队列规则*（queuing discipline）。队列规则是指在给定队列中的一组元素的情况下，确定下一个弹出队列的元素的规则。先进先出声明的是下一个弹出的元素应该是等待时间最长的元素。

优先级队列声明下一个弹出的元素是最需要的元素（具有最高的优先级）。例如，在机场，当飞机临近起飞时，这架飞机的乘客可以在办理登机手续时排到队头。如果构建了一个消息传递系统，某些消息比其他消息更重要，应该尽快处理，而不管它们何时到达。在Java 5 中添加了 **PriorityQueue** ，以便自动实现这种行为。

**当在 PriorityQueue 上调用 `offer()` 方法来插入一个对象时，该对象会在队列中被排序**。默认的排序使用队列中对象的*自然顺序*（natural order），但是可以通过提供自己的 **Comparator** 来修改这个顺序。 **PriorityQueue** 确保在调用 `peek()` ， `poll()` 或 `remove()` 方法时，获得的元素将是队列中优先级最高的元素。

**PriorityQueue** 是允许重复的，最小的值具有最高的优先级（如果是 **String** ，空格也可以算作值，并且比字母的优先级高）。为了展示如何通过提供自己的 **Comparator** 对象来改变顺序，对 **PriorityQueue<String>** 的调用使用了由 `Collections.reverseOrder()` （Java 5 中新添加的）产生的反序的 **Comparator** 。

```Java
public class PriorityQueueDemo {
    static void printQ(Queue queue) {
        while (queue.peek() != null)
            System.out.print(queue.remove() + " ");
        System.out.println();
    }

    public static void main(String[] args) {
        String fact = "EDUCATION SHOULD ESCHEW OBFUSCATION";
        List<String> strings = Arrays.asList(fact.split(""));

        PriorityQueue<String> stringPQ = new PriorityQueue<>(strings.size(), Collections.reverseOrder());
        stringPQ.addAll(strings);

        PriorityQueueDemo.printQ(stringPQ);
        //W U U U T T S S S O O O O N N L I I H H F E E E D D C C C B A A   
    }
}
```

### for-each 和 迭代器 ###

 Java 5 引入了一个名为 **Iterable** 的接口，该接口包含一个能够生成 **Iterator** 的 `iterator()` 方法。*for-each* 使用此 **Iterable** 接口来遍历序列。因此，如果创建了任何实现了 **Iterable** 的类，都可以将它用于 *for-each* 语句中。

```java
public class IterableClass implements Iterable<String> {
    protected String[] words = ("And that is how " +
            "we know the Earth to be banana-shaped."
    ).split(" ");

    @Override
    public Iterator<String> iterator() {
        return new Iterator<String>() {
            private int index = 0;

            @Override
            public boolean hasNext() {
                return index < words.length;
            }

            @Override
            public String next() {
                return words[index++];
            }

            @Override
            public void remove() { // Not implemented
                throw new UnsupportedOperationException();
            }
        };
    }

    public static void main(String[] args) {
        for (String s : new IterableClass())
            System.out.print(s + " ");
    }
}
```

### hashCode() 和 equals() ###

当你创建一个类的时候，它自动继承自 **Objcet** 类。如果你不覆盖 **equals()** ，这个方法会比较对象的地址是否相等，区分度非常高。但是通常这不是我们想要的，我们想要的是比较对象内容是否相等，所以需要自己覆盖 **equals()** 方法。

一个合适的 **equals()** 函数必须满足以下五点条件：

1. 反身性：对于任何 **x**， **x.equals(x)** 应该返回 **true**。
2. 对称性：对于任何 **x** 和 **y**， **x.equals(y)** 和 **y.equals(x)** 的返回值相同。
3. 传递性：对于任何 **x**，**y**，**z**，如果 **x.equals(y)** 返回 **true** 并且 **y.equals(z)** 返回 **true**，那么 **x.equals(z)** 应该返回 **true**。
4. 一致性：对于任何 **x** 和 **y**，在对象没有被改变的情况下，多次调用 **x.equals(y)** 应该总是返回 **true** 或者 **false**。
5. 对于任何非 **null** 的 **x**，**x.equals(null)** 应该返回 **false**。

覆盖 hashCode() 必须注意的点：

- 基于对象的内容生成散列码，散列码不必是唯一的。
- 好的 hashCode() 应该产生分布均匀的散列码。
- 单个属性可以使用 Objects.hashCode()，多个属性可以使用 Objects.hash() 。

**hashCode() 和 equals() 的关系（按照规范覆盖的情况下）**：

- equals() 相等的两个对象他们的 hashCode() 肯定相等
- hashCode() 相等的两个对象他们的 equals() 不一定相等
- hashCode() 的效率比 equals() 高很多，所以**在散列的数据结构中比较两个对象是否相等的流程是：先比较 hashCode() 是否相等，如果相等，再比较 equals() 是否相等，前面的比较都相等的话则两个对象相等。**

要使用散列的数据结构（HashSet，HashMap，LinkedHashst 和 LinkedHashMap）就必须正确为键覆盖 hashCode() 和 equals() 方法。

**总结：Java 提供了许多保存对象的方法**：

1. 数组将数字索引与对象相关联。它保存类型明确的对象，因此在查找对象时不必对结果做类型转换。它可以是多维的，可以保存基本类型的数据。虽然可以在运行时创建数组，但是一旦创建数组，就无法更改数组的大小。
2. **Collection** 保存单一的元素，而 **Map** 包含相关联的键值对。使用 Java 泛型，可以指定集合中保存的对象的类型，因此不能将错误类型的对象放入集合中，并且在从集合中获取元素时，不必进行类型转换。各种 **Collection** 和各种 **Map** 都可以在你向其中添加更多的元素时，自动调整其尺寸大小。集合不能保存基本类型，但自动装箱机制会负责执行基本类型和集合中保存的包装类型之间的双向转换。
3. 除 **TreeSet** 之外的所有 **Set** 都具有与 **Collection** 完全相同的接口。**List** 和 **Collection** 存在着明显的不同，尽管 **List** 所要求的方法都在 **Collection** 中。另一方面，在 **Queue** 接口中的方法是独立的，在创建具有 **Queue** 功能的实现时，不需要使用 **Collection** 方法。最后， **Map** 和 **Collection** 之间唯一的交集是 **Map** 可以使用 `entrySet()` 和 `values()` 来产生 **Collection** 。
4. 像数组一样， **List** 也将数字索引与对象相关联，因此，数组和 **List** 都是有序集合。
5. 如果要执行大量的随机访问，则使用 **ArrayList** ，如果要经常从表中间插入或删除元素，则应该使用 **LinkedList** 。
6. 队列和堆栈的行为是通过 **LinkedList** 提供的。
7. **Map** 是一种将对象（而非数字）与对象相关联的设计。 **HashMap** 专为快速访问而设计，而 **TreeMap** 保持键始终处于排序状态，所以没有 **HashMap** 快。 **LinkedHashMap** 按插入顺序保存其元素，但使用散列提供快速访问的能力。
8. **Set** 不接受重复元素。 **HashSet** 提供最快的查询速度，而 **TreeSet** 保持元素处于排序状态。 **LinkedHashSet** 按插入顺序保存其元素，但使用散列提供快速访问的能力。
9. 不要在新代码中使用遗留类 **Vector** ，**Hashtable** 和 **Stack** 。

下面是 Java 集合框架简图，黄色为接口，绿色为抽象类，蓝色为具体类。虚线箭头表示实现关系，实线箭头表示继承关系。

![collection](C:/Users/zhanghuihong/Documents/GitHub/Addicted-To-Learning/assets/java-foundation/collection.png)

<center>Collection 集合框架</center>

![map](C:/Users/zhanghuihong/Documents/GitHub/Addicted-To-Learning/assets/java-foundation/map.png)

<center>Map 集合框架</center>

### 持有引用 ###

java.lang.ref 包提供了与 Java 垃圾回收器密切相关的引用类。这些引用类对象可以指向其它对象，**其好处就在于使用者可以保持对使用对象的引用，同时 JVM 依然可以在内存不够用的时候对使用对象进行回收**。

**软引用**用于实现对内存敏感的缓存。**弱引用**用于实现“规范化映射”对象的实例可以在程序的多个位置同时使用，以节省存储，但不会阻止其键（或值）被回收。**虚引用**用于调度 pre-mortem 清理操作，这是一种比 Java 终结机制更灵活的方式。

#### 强引用 ####

`FinalReference` 代表强引用，`Finalizer` 是它的子类。 与`Finalizer`相关联的则是 `Object` 中的`finalize()`方法，在类加载的过程中，如果当前类有覆写`finalize()`方法，则其对象会被标记为 `Finalizer` 类，这种类型的对象被回收前会先调用其`finalize()`。

具体做法是，在 GC 进行可达性分析的时候，如果当前对象是 `Finalizer` 类型的对象，并且本身不可达，则会被加入到 `ReferenceQueue` 中。而系统在初始化的过程中，会启动一个`FinalizerThread`实例的守护线程（线程名Finalizer）该线程会不断消费引用队列中的对象，并执行其`finalize()`方法（runFinalizer），并且 runFinalizer() 会捕获 Throwable 级别的异常，也就是说`finalize()`方法的异常不会导致`FinalizerThread`运行中断退出。对象在执行`finalize()`方法后，只是断开了与`Finalizer`的关联，并不意味着会立即被回收，还是要等待下一次 GC，而每个对象的`finalize()`方法都只会执行一次，不会重复执行。

 JVM 中对象是被分配在堆（heap）上的，当程序行动中不再有引用指向这个对象时，这个对象就可以被垃圾回收器所回收。这里所说的引用也就是我们一般意义上申明的对象类型的变量（如 String, Object, ArrayList 等），也称为强引用。

```java
String tag = new String("abc");
```

此处的 tag 引用就称之为强引用。而强引用有以下特征：

- 强引用可以直接访问目标对象。
- 强引用所指向的对象在任何时候都不会被系统回收。
- 强引用可能导致内存泄漏。

#### 软引用 ####

`SoftReference` 在”弱引用”中属于最强的引用。`SoftReference` 所指向的对象，垃圾回收器会根据 JVM 内存的使用情况以及 `SoftReference` 的 `get()` 方法的调用情况来决定是否对其进行回收。具体使用一般是通过 `SoftReference` 的构造方法，将需要用弱引用来指向的对象包装起来。当需要使用的时候，调用 `SoftReference` 的 `get()` 方法来获取。当对象未被回收时 `SoftReference` 的 `get()` 方法会返回该对象的强引用。如下：

```java
SoftReference<Bean> bean = new SoftReference<Bean>(new Bean("name", 10));
System.out.println(bean.get());// "name:10”
```

软引用有以下特征：

- 软引用使用 `get()` 方法取得对象的强引用从而访问目标对象。
- 软引用所指向的对象按照 JVM 的使用情况（Heap 内存是否临近阈值）来决定是否回收。
- 软引用可以避免 Heap 内存不足所导致的异常。

当垃圾回收器决定对其回收时，会先清空它的 `SoftReference`，也就是说 `SoftReference` 的 `get()` 方法将会返回 `null`，然后再调用对象的 `finalize()` 方法，并在下一轮 GC 中对其真正进行回收。

#### 弱引用 ####

弱引用的特性和基本与软引用相似，区别就在于弱引用所指向的对象只要进行系统垃圾回收，不管内存使用情况如何，永远对其进行回收（get() 方法返回 null）。

弱引用有以下特征：

- 弱引用使用 `get()` 方法取得对象的强引用从而访问目标对象。
- 一旦系统内存回收，无论内存是否紧张，弱引用指向的对象都会被回收。
- 弱引用也可以避免 Heap 内存不足所导致的异常。

#### 虚引用 ####

`PhantomReference` 是所有”弱引用”中最弱的引用类型。不同于软引用和弱引用，虚引用无法通过 `get()` 方法来取得目标对象的强引用从而使用目标对象，观察源码可以发现 `get()` 被重写为永远返回 `null`。

那虚引用到底有什么作用？其实虚引用主要被用来 **跟踪对象被垃圾回收的状态** ，通过查看引用队列中是否包含对象所对应的虚引用来判断它是否 **即将** 被垃圾回收，从而采取行动。它并不被期待用来取得目标对象的引用，**目标对象被回收前，它的引用会被放入一个 `ReferenceQueue` 对象中**，从而达到跟踪对象垃圾回收的作用。

```java
public static void main(String[] args) {
    ReferenceQueue<String> refQueue = new ReferenceQueue<>();
    PhantomReference<String> referent = new PhantomReference<>(
            new String("abc"), refQueue);
    System.out.println(referent.get()); // null

    System.gc();
    System.runFinalization();
    System.out.println(refQueue.poll() == referent); //true
}
```

虚引用有以下特征：

- 虚引用永远无法使用 `get()` 方法取得对象的强引用从而访问目标对象。
- 虚引用所指向的对象在被系统内存回收前，虚引用自身会被放入 `ReferenceQueue` 对象中从而跟踪对象垃圾回收。
- 虚引用随时都可能被垃圾回收器回收。

#### 引用类型总结 ####

| **引用类型** | **取得目标对象方式** | **垃圾回收条件** | **是否可能内存泄漏** |
| :----------- | :------------------- | :--------------- | :------------------- |
| 强引用       | 直接调用             | 不回收           | 可能                 |
| 软引用       | 通过 get() 方法      | 视内存情况回收   | 不可能               |
| 弱引用       | 通过 get() 方法      | 永远回收         | 不可能               |
| 虚引用       | 无法取得             | 不回收           | 可能                 |

**注意：**如果想使用这些相对强引用来说较弱的引用来进行对象操作的时候，就必须保证没有强引用指向被操作对象。否则将会被视为强引用指向，不会具有任何的弱引用的特性。

### WeakHashMap ###

**WeakHashMap** 类可以轻松地创建规范化映射。在这种映射中，可以通过仅仅创建一个特定值的实例来节省存储空间。当程序需要该值时，它会查找映射中的现有对象并使用它。 该映射可以将值作为其初始化的一部分，但更有可能的是在需要时创建该值。

由于这是一种节省存储空间的技术，因此 **WeakHashMap** 允许垃圾收集器自动清理键和值，这是非常方便的。不能对放在 **WeakHashMap** 中的键和值做任何特殊操作，它们由 map 自动包装在 **WeakReference** 中。当键不再被使用的时候才允许清理。