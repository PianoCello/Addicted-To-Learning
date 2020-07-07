## 文件 ##

 Java 7 引入了 **java.nio.file** 包，过去人们通常把 **nio** 中的 **n** 理解为 **new** 即新的 **io**，现在应该理解为 **non-blocking** 非阻塞 **io**。

 Java 8 新增的 **stream** 与文件结合使得文件操作编程变得更加优雅。文件操作的两个基本组件：

1. 文件或者目录的路径；
2. 文件本身；

### 文件和目录路径 ###

一个 **Path** 对象表示一个文件或者目录的路径，是一个跨操作系统（OS）和文件系统的抽象，目的是在构造路径时不必关注底层操作系统，代码可以在不进行修改的情况下运行在不同的操作系统上。**java.nio.file.Paths** 类包含一个重载方法 **static get()**，该方法接受一系列 **String** 字符串或一个*统一资源标识符*（URI）作为参数，并且进行转换返回一个 **Path** 对象：

```java
public class PathInfo {
    static void show(String id, Object p) {
        System.out.println(id + ": " + p);
    }

    static void info(Path p) {
        show("toString", p);
        show("Exists", Files.exists(p));
        show("RegularFile", Files.isRegularFile(p));
        show("Directory", Files.isDirectory(p));
        show("Absolute", p.isAbsolute());
        show("FileName", p.getFileName());
        show("Parent", p.getParent());
        show("Root", p.getRoot());
        System.out.println("******************");
    }
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        info(Paths.get("C:", "path", "to", "nowhere", "NoFile.txt"));
        Path p = Paths.get("PathInfo.java");
        info(p);
        //绝对路径 将 parent 和当前文件或路径 简单拼接
        Path ap = p.toAbsolutePath();
        info(ap);
        info(ap.getParent());
        try {
            //真实绝对路径 文件或目录不存在会抛异常
            info(p.toRealPath());
        } catch(IOException e) {
           System.out.println(e);
        }
        URI u = p.toUri();
        System.out.println("URI: " + u);
        Path puri = Paths.get(u);
        System.out.println(Files.exists(puri));
        File f = ap.toFile(); // Don't be fooled
    }
}
```

#### 选取路径部分片段 ####

**Path** 对象可以非常容易地生成路径的某一部分：

```java
public class PartsOfPaths {
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        Path p = Paths.get("PartsOfPaths.java").toAbsolutePath();
        for(int i = 0; i < p.getNameCount(); i++)
            System.out.println(p.getName(i));
        System.out.println("ends with '.java': " +
        p.endsWith(".java"));
        for(Path pp : p) {
            System.out.print(pp + ": ");
            System.out.print(p.startsWith(pp) + " : ");
            System.out.println(p.endsWith(pp));
        }
        System.out.println("Starts with " + p.getRoot() + " " + p.startsWith(p.getRoot()));
    }
}
```

可以通过 **getName()** 来索引 **Path** 的各个部分，直到达到上限 **getNameCount()**。**Path** 也实现了 **Iterable** 接口，因此我们也可以通过增强的 for-each 进行遍历。请注意，即使路径以 **.java** 结尾，使用 **endsWith()** 方法也会返回 **false**。这是因为使用 **endsWith()** 比较的是整个路径部分，而不会包含文件路径的后缀。遍历 **Path** 对象并不包含根路径，只有使用 **startsWith()** 检测根路径时才会返回 **true**。

#### 路径分析 ####

**Files** 工具类包含一系列完整的方法用于获得 **Path** 相关的信息。

```java
public class PathAnalysis {
    static void say(String id, Object result) {
        System.out.print(id + ": ");
        System.out.println(result);
    }

    public static void main(String[] args) throws IOException {
        System.out.println(System.getProperty("os.name"));
        Path p = Paths.get("PathAnalysis.java").toAbsolutePath();
        say("Exists", Files.exists(p));
        say("Directory", Files.isDirectory(p));
        say("Executable", Files.isExecutable(p));
        say("Readable", Files.isReadable(p));
        say("RegularFile", Files.isRegularFile(p));
        say("Writable", Files.isWritable(p));
        say("notExists", Files.notExists(p));
        say("Hidden", Files.isHidden(p));
        say("size", Files.size(p));
        say("FileStore", Files.getFileStore(p));
        say("LastModified: ", Files.getLastModifiedTime(p));
        say("Owner", Files.getOwner(p));
        say("ContentType", Files.probeContentType(p));
        say("SymbolicLink", Files.isSymbolicLink(p));
        if(Files.isSymbolicLink(p))
            say("SymbolicLink", Files.readSymbolicLink(p));
        if(FileSystems.getDefault().supportedFileAttributeViews().contains("posix"))
            say("PosixFilePermissions",
        Files.getPosixFilePermissions(p));
    }
}
```

在调用最后一个测试方法 **getPosixFilePermissions()** 之前我们需要确认一下当前文件系统是否支持 **Posix** 接口，否则会抛出运行时异常。

### 文件系统 ###

使用静态的 **FileSystems** 工具类获取默认的文件系统，但你同样也可以在 **Path** 对象上调用 **getFileSystem()** 以获取创建该 **Path** 的文件系统。你可以获得给定 *URI* 的文件系统，还可以构建新的文件系统(对于支持它的操作系统)。一个 **FileSystem** 对象也能生成 **WatchService** 和 **PathMatcher** 对象。

```java
public class FileSystemDemo {
    static void show(String id, Object o) {
        System.out.println(id + ": " + o);
    }

    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        FileSystem fsys = FileSystems.getDefault();
        for(FileStore fs : fsys.getFileStores())
            show("File Store", fs);
        for(Path rd : fsys.getRootDirectories())
            show("Root Directory", rd);
        show("Separator", fsys.getSeparator());
        show("UserPrincipalLookupService",
            fsys.getUserPrincipalLookupService());
        show("isOpen", fsys.isOpen());
        show("isReadOnly", fsys.isReadOnly());
        show("FileSystemProvider", fsys.provider());
        show("File Attribute Views",
        fsys.supportedFileAttributeViews());
    }
}
```

### 路径监听 ###

通过 **WatchService** 可以设置一个进程对目录中的更改做出响应。在这个例子中，**delTxtFiles()** 作为一个单独的任务执行，该任务将遍历整个目录并删除以 **.txt** 结尾的所有文件，**WatchService** 会对文件删除操作做出反应：

```java
import static java.nio.file.StandardWatchEventKinds.*;
public class PathWatcher {
    static Path test = Paths.get("test");

    static void delTxtFiles() {
        try {
            Files.walk(test)
            .filter(f ->
                f.toString()
                .endsWith(".txt"))
                .forEach(f -> {
                try {
                    System.out.println("deleting " + f);
                    Files.delete(f);
                } catch(IOException e) {
                    throw new RuntimeException(e);
                }
            });
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) throws Exception {
        Files.createFile(test.resolve("Hello.txt"));
        WatchService watcher = FileSystems.getDefault().newWatchService();
        test.register(watcher, ENTRY_DELETE);
        
        Executors.newSingleThreadScheduledExecutor()
        .schedule(PathWatcher::delTxtFiles,
        250, TimeUnit.MILLISECONDS);
        
        WatchKey key = watcher.take();
        for(WatchEvent evt : key.pollEvents()) {
            System.out.println("evt.context(): " + evt.context() +
            "\nevt.count(): " + evt.count() +
            "\nevt.kind(): " + evt.kind());
            System.exit(0);
        }
    }
}
```

注意的是在 **filter()** 中，我们必须显式地使用 **f.toString()** 转为字符串，否则我们调用 **endsWith()** 将会与整个 **Path** 对象进行比较，而不是路径名称字符串的一部分进行比较。

一旦我们从 **FileSystem** 中得到了 **WatchService** 对象，我们将其注册到 **test** 路径以及我们感兴趣的项目的变量参数列表中，可以选择 **ENTRY_CREATE**，**ENTRY_DELETE** 或 **ENTRY_MODIFY**(其中创建和删除不属于修改)。

因为接下来对 **watcher.take()** 的调用会在发生某些事情之前停止所有操作，所以我们希望 **deltxtfiles()** 能够并行运行以便生成我们感兴趣的事件。为了实现这个目的，我通过调用 **Executors.newSingleThreadScheduledExecutor()** 产生一个 **ScheduledExecutorService** 对象，然后调用 **schedule()** 方法传递所需函数的方法引用，并且设置在运行之前应该等待的时间。

此时，**watcher.take()** 将等待并阻塞在这里。当目标事件发生时，会返回一个包含 **WatchEvent** 的 **Watchkey** 对象。展示的这三种方法是能对 **WatchEvent** 执行的全部操作。

查看输出的具体内容。即使我们正在删除以 **.txt** 结尾的文件，在 **Hello.txt** 被删除之前，**WatchService** 也不会被触发。你可能认为，如果说"监视这个目录"，自然会包含整个目录和下面子目录，但实际上：只会监视给定的目录，而不是下面的所有内容。如果需要监视整个树目录，必须在整个树的每个子目录上放置一个 **WatchService**。

### 文件查找 ###

到目前为止，为了找到文件，我们一直使用相当粗糙的方法，在 `path` 上调用 `toString()`，然后使用 `string` 操作查看结果。事实证明，`java.nio.file` 有更好的解决方案：通过在 `FileSystem` 对象上调用 `getPathMatcher()` 获得一个 `PathMatcher`，然后传入您感兴趣的模式。模式有两个选项：`glob` 和 `regex`。`glob` 比较简单，实际上功能非常强大，因此您可以使用 `glob` 解决许多问题。如果您的问题更复杂，可以使用 `regex`。

### 文件读写 ###

我们可以对路径和目录做任何事情。 现在让我们看一下操纵文件本身的内容。

如果一个文件很“小”，也就是说“它运行得足够快且占用内存小”，那么 `java.nio.file.Files` 类中的实用程序将帮助你轻松读写文本和二进制文件。

`Files.readAllLines()` 一次读取整个文件（因此，“小”文件很有必要），产生一个`List<String>`。 对于示例文件，我们将重用`Cheese.dat`：

```java
public class ListOfLines {
    public static void main(String[] args) throws Exception {
        Files.readAllLines(
        Paths.get("Cheese.dat"))
        .stream()
        .filter(line -> !line.startsWith("//"))
        .map(line ->
            line.substring(0, line.length()/2))
        .forEach(System.out::println);
    }
}
```

跳过注释行，其余的内容每行只打印一半。 这实现起来很简单：你只需将 `Path` 传递给 `readAllLines()` （以前的 java 实现这个功能很复杂）。`readAllLines()` 有一个重载版本，包含一个 `Charset` 参数来存储文件的 Unicode 编码。

`Files.write()` 被重载以写入 `byte` 数组或任何 `Iterable` 对象（它也有 `Charset` 选项）：

```java
public class Writing {
    static Random rand = new Random(47);
    static final int SIZE = 1000;

    public static void main(String[] args) throws Exception {
        // Write bytes to a file:
        byte[] bytes = new byte[SIZE];
        rand.nextBytes(bytes);
        Files.write(Paths.get("bytes.dat"), bytes);
        System.out.println("bytes.dat: " + 						 	 	  	      Files.size(Paths.get("bytes.dat")));

        // Write an iterable to a file:
        List<String> lines = Files.readAllLines(
          Paths.get("../streams/Cheese.dat"));
        Files.write(Paths.get("Cheese.txt"), lines);
        System.out.println("Cheese.txt: " + 		  						 	Files.size(Paths.get("Cheese.txt")));
    }
}
```

我们使用 `Random` 来创建一个随机的 `byte` 数组; 你可以看到生成的文件大小是 1000。

一个 `List` 被写入文件，任何 `Iterable` 对象也可以这么做。

如果文件大小有问题怎么办？ 比如说：

1. 文件太大，如果你一次性读完整个文件，你可能会耗尽内存。
2. 您只需要在文件的中途工作以获得所需的结果，因此读取整个文件会浪费时间。

`Files.lines()` 方便地将文件转换为行的 `Stream`：

```java
public class ReadLineStream {
    public static void main(String[] args) throws Exception {
        Files.lines(Paths.get("PathInfo.java"))
          .skip(13)
          .findFirst()
          .ifPresent(System.out::println);
    }
}
/* Output:
    show("RegularFile", Files.isRegularFile(p));
*/
```

这对本章中第一个示例代码做了流式处理，跳过 13 行，然后选择下一行并将其打印出来。

`Files.lines()` 对于把文件处理行的传入流时非常有用，但是如果你想在 `Stream` 中读取，处理或写入怎么办？这就需要稍微复杂的代码：

```java
public class StreamInAndOut {
    public static void main(String[] args) {
        try(
          Stream<String> input =
            Files.lines(Paths.get("StreamInAndOut.java"));
          PrintWriter output =
            new PrintWriter("StreamInAndOut.txt")
        ) {
            input.map(String::toUpperCase)
              .forEachOrdered(output::println);
        } catch(Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

因为我们在同一个块中执行所有操作，所以这两个文件都可以在相同的 try-with-resources 语句中打开。`PrintWriter` 是一个旧式的 `java.io` 类，允许你“打印”到一个文件，所以它是这个应用的理想选择。如果你看一下 `StreamInAndOut.txt`，你会发现它里面的内容确实是大写的。



## 传统 I/O ##

实现良好的输入/输出（I/O）系统是一项艰难的任务，不仅要覆盖到不同的 I/O 源和 I/O 接收器（如文件、控制台、网络连接等），还要实现多种与它们进行通信的方式（如顺序、随机访问、缓冲、二进制、字符、按行和按字等）。

### 输入流类型 ###

`InputStream` 表示那些从不同数据源产生输入的类，这些数据源包括：

1. 字节数组；
2. `String` 对象；
3. 文件；
4. “管道”，工作方式与实际生活中的管道类似：从一端输入，从另一端输出；
5. 一个由其它种类的流组成的序列，然后我们可以把它们汇聚成一个流；
6. 其它数据源，如 Internet 连接。

每种数据源都有相应的 `InputStream` 子类。另外，`FilterInputStream` 也属于一种 `InputStream`，它的作用是为“装饰器”类提供基类。其中，“装饰器”类可以把属性或有用的接口与输入流连接在一起。

**`InputStream` 的类型：**

| 类                                        | 功能                                                         | 构造器参数                                                   | 如何使用                                                     |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ByteArrayInputStream`                    | 允许将内存的缓冲区当做 `InputStream` 使用                    | 缓冲区，字节将从中取出                                       | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
| @Deprecated<br/>`StringBufferInputStream` | 将 `String` 转换成 `InputStream`                             | 字符串。底层实现实际使用 `StringBuffer`                      | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
| `FileInputStream`                         | 用于从文件中读取信息                                         | 字符串，表示文件名、文件或 `FileDescriptor` 对象             | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
| `PipedInputStream`                        | 产生用于写入相关 `PipedOutputStream` 的数据。实现“管道化”概念 | `PipedOutputSteam`                                           | 作为多线程中的数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
| `SequenceInputStream`                     | 将两个或多个 `InputStream` 对象转换成一个 `InputStream`      | 两个 `InputStream` 对象或一个容纳 `InputStream` 对象的容器 `Enumeration` | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
| `FilterInputStream`                       | 抽象类，作为“装饰器”的接口。其中，“装饰器”为其它的 `InputStream` 类提供有用的功能。 |                                                              |                                                              |



### 输出流类型 ###

`OutputStream` 表示输出所要去往的目标：字节数组、文件或管道。另外，`FilterOutputStream` 为“装饰器”类提供了一个基类，“装饰器”类把属性或者有用的接口与输出流连接了起来。

**`OutputStream` 类型：**

| 类                      | 功能                                                         | 构造器参数                                       | 如何使用                                                     |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| `ByteArrayOutputStream` | 在内存中创建缓冲区。所有送往“流”的数据都要放置在此缓冲区     | 缓冲区初始大小（可选）                           | 用于指定数据的目的地：将其与 `FilterOutputStream` 对象相连以提供有用接口 |
| `FileOutputStream`      | 用于将信息写入文件                                           | 字符串，表示文件名、文件或 `FileDescriptor` 对象 | 用于指定数据的目的地：将其与 `FilterOutputStream` 对象相连以提供有用接口 |
| `PipedOutputStream`     | 任何写入其中的信息都会自动作为相关 `PipedInputStream` 的输出。实现“管道化”概念 | `PipedInputStream`                               | 指定用于多线程的数据的目的地：将其与 `FilterOutputStream` 对象相连以提供有用接口 |
| `FilterOutputStream`    | 抽象类，作为“装饰器”的接口。其中，“装饰器”为其它 `OutputStream` 提供有用功能。 |                                                  |                                                              |



### 流的装饰器 ###

Java I/O 类库需要多种不同功能的组合，这正是使用**装饰器模式**的原因所在。

#### FilterInputStream ####

`FilterInputStream` 类能够完成两件截然不同的事情。其中，`DataInputStream` 允许我们读取不同的基本数据类型和 `String` 类型的对象（所有方法都以 “read” 开头，例如 `readByte()`、`readFloat()`等等）。搭配其对应的 `DataOutputStream`，我们就可以通过数据“流”将基本数据类型的数据从一个地方迁移到另一个地方。

其它 `FilterInputStream` 类则在内部修改 `InputStream` 的行为方式：是否缓冲，是否保留它所读过的行（允许我们查询行数或设置行数），以及是否允许把单个字符推回输入流等等。

**`FilterInputStream` 类型：**

| 类                                      | 功能                                                         | 构造器参数                                | 如何使用                                                 |
| --------------------------------------- | ------------------------------------------------------------ | ----------------------------------------- | -------------------------------------------------------- |
| `DataInputStream`                       | 与 `DataOutputStream` 搭配使用，按照移植方式从流读取基本数据类型（`int`、`char`、`long` 等） | `InputStream`                             | 包含用于读取基本数据类型的全部接口                       |
| `BufferedInputStream`                   | 使用它可以防止每次读取时都得进行实际写操作。代表“使用缓冲区” | `InputStream`，可以指定缓冲区大小（可选） | 本质上不提供接口，只是向进程添加缓冲功能。与接口对象搭配 |
| @Deprecated<br/>`LineNumberInputStream` | 跟踪输入流中的行号，可调用 `getLineNumber()` 和 `setLineNumber(int)` | `InputStream`                             | 仅增加了行号，因此可能要与接口对象搭配使用               |
| `PushbackInputStream`                   | 具有能弹出一个字节的缓冲区，因此可以将读到的最后一个字符回退 | `InputStream`                             | 通常作为编译器的扫描器，我们可能永远也不会用到           |



#### FilterOutputStream ####

与 `DataInputStream` 对应的是 `DataOutputStream`，它可以将各种基本数据类型和 `String` 类型的对象格式化输出到“流”中，。这样一来，任何机器上的任何 `DataInputStream` 都可以读出它们。所有方法都以 “write” 开头，例如 `writeByte()`、`writeFloat()` 等等。

`PrintStream` 最初的目的就是为了以可视化格式打印所有基本数据类型和 `String` 类型的对象。这和 `DataOutputStream` 不同，后者的目的是将数据元素置入“流”中，使 `DataInputStream` 能够可移植地重构它们。

`PrintStream` 内有两个重要方法：`print()` 和 `println()`。它们都被重载了，可以打印各种各种数据类型。`print()` 和 `println()` 之间的差异是，后者在操作完毕后会添加一个换行符。

**`FilterOutputStream` 类型：**

| 类                     | 功能                                                         | 构造器参数                                                   | 如何使用                                                     |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `DataOutputStream`     | 与 `DataInputStream` 搭配使用，因此可以按照移植方式向流中写入基本数据类型（`int`、`char`、`long` 等） | `OutputStream`                                               | 包含用于写入基本数据类型的全部接口                           |
| `PrintStream`          | 用于产生格式化输出。其中 `DataOutputStream` 处理数据的存储，`PrintStream` 处理显示 | `OutputStream`，可以用 `boolean` 值指示是否每次换行时清空缓冲区（可选） | 应该是对 `OutputStream` 对象的 `final` 封装。可能会经常用到它 |
| `BufferedOutputStream` | 使用它以避免每次发送数据时都进行实际的写操作。代表“使用缓冲区”。可以调用 `flush()` 清空缓冲区 | `OutputStream`，可以指定缓冲区大小（可选）                   | 本质上并不提供接口，只是向进程添加缓冲功能。与接口对象搭配   |



### Reader 和 Writer ###

Java 1.1 在 I/O 流类库中加入了用于处理字符的`Reader` 和 `Writer` 。 `InputStream` 和 `OutputStream` 在面向字节 I/O 处理发挥着重要的作用，而 `Reader` 和 `Writer` 则提供兼容 Unicode 和面向字符 I/O 的功能。

有时我们必须把来自“字节”层级结构中的类和来自“字符”层次结构中的类结合起来使用。为了达到这个目的，需要用到“适配器（adapter）类”：`InputStreamReader` 可以把 `InputStream` 转换为 `Reader`，而 `OutputStreamWriter` 可以把 `OutputStream` 转换为 `Writer`。

设计 `Reader` 和 `Writer` 继承体系主要是为了国际化。老的 I/O 流继承体系仅支持 8 比特的字节流，并且不能很好地处理 16 比特的 Unicode 字符。由于 Unicode 用于字符国际化（Java 本身的 `char` 也是 16 比特的 Unicode），所以添加 `Reader` 和 `Writer` 继承体系就是为了让所有的 I/O 操作都支持 Unicode。

#### 数据的来源和去处 ####

几乎所有原始的 Java I/O 流类都有相应的 `Reader` 和 `Writer` 类来提供原生的 Unicode 操作。但是在某些场合，面向字节的 `InputStream` 和 `OutputStream` 才是正确的解决方案。`java.util.zip` 类库就是面向字节而不是面向字符的。所以，尽量尝试使用 `Reader` 和 `Writer`，必要时才使用面向字节的类库。

下表展示了在两个继承体系中，信息的来源和去处之间的对应关系：

| 来源与去处：Java 1.0 类             | 相应的 Java 1.1 类                    |
| ----------------------------------- | ------------------------------------- |
| `InputStream`                       | `Reader` 适配器：`InputStreamReader`  |
| `OutputStream`                      | `Writer` 适配器：`OutputStreamWriter` |
| `FileInputStream`                   | `FileReader`                          |
| `FileOutputStream`                  | `FileWriter`                          |
| `StringBufferInputStream`（已弃用） | `StringReader`                        |
| （无相应的类）                      | `StringWriter`                        |
| `ByteArrayInputStream`              | `CharArrayReader`                     |
| `ByteArrayOutputStream`             | `CharArrayWriter`                     |
| `PipedInputStream`                  | `PipedReader`                         |
| `PipedOutputStream`                 | `PipedWriter`                         |

#### 字符流的装饰器 ####

| 装饰器：Java 1.0 类     | 相应 Java 1.1 类                                             |
| ----------------------- | ------------------------------------------------------------ |
| `FilterInputStream`     | `FilterReader`                                               |
| `FilterOutputStream`    | `FilterWriter` (抽象类，没有子类)                            |
| `BufferedInputStream`   | `BufferedReader`（也有 `readLine()`)                         |
| `BufferedOutputStream`  | `BufferedWriter`                                             |
| `DataInputStream`       | 使用 `DataInputStream`（ 如果必须用到 `readLine()`，那你就得使用 `BufferedReader`。否则，一般情况下就用 `DataInputStream` |
| `PrintStream`           | `PrintWriter`                                                |
| `LineNumberInputStream` | `LineNumberReader`                                           |
| `StreamTokenizer`       | `StreamTokenizer`（使用具有 `Reader` 参数的构造器）          |
| `PushbackInputStream`   | `PushbackReader`                                             |



#### 未发生改变的类 ####

有一些类在 Java 1.0 和 Java 1.1 之间未做改变。

```java
DataOutputStream
File
RandomAccessFile
SequenceInputStream
```

特别是 `DataOutputStream`，在使用时没有任何变化；因此如果想以可传输的格式存储和检索数据，请用 `InputStream` 和 `OutputStream` 继承体系。

### RandomAccessFile ###

RandomAccessFile 类是 Java IO 体系中功能最为丰富的文件访问类，它提供了众多的文件访问方法。RandomAccessFile 类支持“随机访问”方式，这里的“随机”是指程序可以直接跳到文件的任意位置来读写数据。

RandomAccessFile 用两个方法来操作文件记录指针：

```java
long getFilePointer() //返回文件记录指针的当前位置
void seek(long pos) //将文件记录指针定位到pos位置
```

RandomAccessFile 有两个构造器：

```java
RandomAccessFile(File file ,  String mode)  //创建文件流，文件属性由参数File对象指定
RandomAccessFile(String name ,  String mode) //创建文件流，文件名由参数name指定
```

除了指定文件以外，还需要指定一个 mode 参数，该参数指定 RandomAccessFile 的访问模式，该参数有如下四个值：

```
r：以只读方式打开指定文件。如果试图对该 RandomAccessFile 指定的文件执行写入方法则会抛出IOException。
rw：以读取、写入方式打开指定文件。如果该文件不存在，则尝试创建文件。
rws：以读取、写入方式打开指定文件。相对于rw模式，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备，默认情形下（rw 模式下）,是使用 buffer 的,只有 cache 满的或者使用 RandomAccessFile.close() 关闭流的时候儿才真正的写到文件。
rwd：与 rws 类似，只是仅对文件的内容同步更新到磁盘，而不修改文件的元数据。
```

在 Java 1.4 中，`RandomAccessFile` 的大多数功能都被 nio 中的**内存映射文件**（mmap）取代。

### I/O 流典型用途 ###

以下例子可以作为 I/O 典型用法的基本参照。

#### 缓冲输入文件 ####

如果想要打开一个文件并且按字符输入，我们可以使用一个 `FileReader` 对象，然后传入一个 `String` 或者 `File` 对象作为文件名。为了提高速度，我们希望对那个文件进行缓冲，那么我们可以将所产生的引用传递给一个 `BufferedReader` 构造器。`BufferedReader` 提供了 `line()` 方法，它会产生一个 `Stream<String>` 对象：

```java
public class BufferedInputFile {
    public static String read(String filename) {
        try (BufferedReader in = new BufferedReader(
                new FileReader(filename))) {
            return in.lines()
                    .collect(Collectors.joining("\n"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        System.out.print(
                read("BufferedInputFile.java"));
    }
}
```

#### 从内存输入 ####

下面示例中，从 `BufferedInputFile.read()` 读入的 `String` 被用来创建一个 `StringReader` 对象。然后调用其 `read()` 方法，每次读取一个字符，并把它显示在控制台上：

```
public class MemoryInput {
    public static void main(String[] args) throws IOException {
        StringReader in = new StringReader(
                BufferedInputFile.read("MemoryInput.java"));
        int c;
        while ((c = in.read()) != -1)
            System.out.print((char) c);
    }
}

```

注意 `read()` 是以 `int` 形式返回下一个字节，所以必须类型转换为 `char` 才能正确打印。

#### 格式化内存输入 ####



#### 基本文件的输出 ####



#### 文本文件输出快捷方式 ####



#### 存储和恢复数据 ####



#### 读写随机访问文件 ####



