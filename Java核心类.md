## Java核心类

1. #### 字符串和编码

   - ##### String

     在Java中，String是一个引用类型，它本身也是一个class。但是，Java编译器对String有特殊处理，即可以直接用"..."来表示一个字符串

     ```java
     String s1 = "Hello!";
     ```

     实际上字符串在String内部是通过一个char[]数组表示的

     ```java
     String s2 = new String(new char[] {'H', 'e', 'l', 'l', 'o', '!'});
     ```

     两个字符串比较，必须总是使用`equals()`法，要忽略大小写比较，使用`equalsIgnoreCase()`方法

   - ##### 常用的方法

     - `contains`
       注意到contains()方法的参数是CharSequence而不是String，因为CharSequence是String的父类

     - "Hello".indexOf("l"); // 2

     - "Hello".lastIndexOf("l"); // 3

     - "Hello".startsWith("He"); // true

     - "Hello".endsWith("lo"); // true

     - "Hello".substring(2); // "llo"

     - "Hello".substring(2, 4); //"ll"

     - "Hello".charAt(0);//"H"
     
- String.valueOf();
  
- 去除首尾空白字符
       使用trim()方法可以移除字符串首尾空白字符。空白字符包括空格，\t，\r，\n

       - trim()并没有改变字符串的内容，而是返回了一个新字符串
        
         "  \tHello\r\n ".trim(); // "Hello"
         
       - strip()方法也可以移除字符串首尾空白字符。它和trim()不同的是，类似中文的空格字符\u3000也会被移除：
        
         "\u3000Hello\u3000".strip(); // "Hello"
    " Hello ".stripLeading(); // "Hello "
         " Hello ".stripTrailing(); // " Hello"
   
       - isEmpty()和isBlank()来判断字符串是否为空和空白字符串：
   
    "".isEmpty(); // true，因为字符串长度为0
         "  ".isEmpty(); // false，因为字符串长度不为0
    "  \n".isBlank(); // true，因为只包含空白字符
         " Hello ".isBlank(); // false，因为包含非空白字符
   
- 替换子串
  
  - "hello".replace('l', 'w'); // "hewwo"，所有字符'l'被替换为'w'
       - "A,,B;C ,D".replaceAll("[\\,\\;\\s]+", ","); // "A,B,C,D"
  
- 分割字符串
  
  - String s = "A,B,C,D";
         String[] ss = s.split("\\,"); // {"A", "B", "C", "D"}

     - 拼接字符串

       1. **StringBuilder**
     
          它是一个可变对象，可以预分配缓冲区，这样，往StringBuilder中新增字符时，不会创建新的临时对象：
     
          ```java
          StringBuilder sb = new StringBuilder(1024);
          for (int i = 0; i < 1000; i++) {
              sb.append(',');
           sb.append(i);
          }
     String s = sb.toString();
          ```
     
     StringBuilder还可以进行链式操作
     
     如果我们查看StringBuilder的源码，可以发现，进行链式操作的关键是，定义的append()方法会返回this，这样，就可以不断调用自身的其他方法
          注意：对于普通的字符串+操作，并不需要我们将其改写为StringBuilder，因为Java编译器在编译时就自动把多个连续的+操作编码为StringConcatFactory的操作。在运行期，StringConcatFactory会自动把字符串连接操作优化为数组复制或者StringBuilder操作

          你可能还听说过StringBuffer，这是Java早期的一个StringBuilder的线程安全版本，它通过同步来保证多个线程操作StringBuffer也是安全的，但是同步会带来执行速度的下降
     
       2. **StringJoiner**
     
          ```java
          public class Main {
              public static void main(String[] args) {
                  String[] names = {"Bob", "Alice", "Grace"};
                  var sj = new StringJoiner(", ", "Hello ", "!");
                  for (String name : names) {
                      sj.add(name);
                  }
             System.out.println(sj.toString());
              }
     }
          ```

          StringJoiner内部实际上就是使用了StringBuilder，所以拼接效率和StringBuilder几乎是一模一样的

       3. **String.join**()
     
          String还提供了一个静态方法join()，这个方法在内部使用了StringJoiner来拼接字符串，在不需要指定“开头”和“结尾”的时候，用String.join()更方便
     
          ```java
          String[] names = {"Bob", "Alice", "Grace"};
     var s = String.join(", ", names);
          ```
     
       4. **Joiner**
     
          Google Guava提供了Joiner类专门用来连接String
     
          ```java
          Joiner joiner = Joiner.on(";");
          String r = joiner.join(new String[]{"a", "b", "c"});//a;b;c
          ```
          
       5. **Collectors.joining**()
     
          ```java
          List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
          String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
          System.out.println("合并字符串: " + mergedString);
          //合并字符串: abc, bc, efg, abcd, jkl
          ```
     
     - 格式化字符串
     
       ```java
       public class Main {
       	public static void main(String[] args) {
                      String s = "Hi %s, your score is %d!";
                      System.out.println(s.formatted("Alice", 80));
                      System.out.println(String.format("Hi %s, your score is %.2f!", "Bob", 59.5));
           }
       }
       ```
     
       ```
       常用的占位符有：
       %s：显示字符串；
       %d：显示整数；
       %x：显示十六进制整数；
       %f：显示浮点数。
       ```
     
       ~~~java
       占位符还可以带格式，例如%.2f表示显示两位小数。如果你不确定用啥占位符，那就始终用%s
        - 类型转换
          要把任意基本类型或引用类型转换为字符串，可以使用静态方法valueOf()。这是一个重载方法，编译器会根据参数自动选择合适的方法
       ```java
       String.valueOf(123); // "123"
       String.valueOf(45.67); // "45.67"
       String.valueOf(true); // "true"
       String.valueOf(new Object()); // 类似java.lang.Object@636be97c
       ```
       int n1 = Integer.parseInt("123"); // 123
       int n2 = Integer.parseInt("ff", 16); // 按十六进制转换，255
       boolean b1 = Boolean.parseBoolean("true"); // true
       boolean b2 = Boolean.parseBoolean("FALSE"); // false
       ~~~
     
       ```java
       要特别注意，Integer有个getInteger(String)方法，它不是将字符串转换为int，而是把该字符串对应的系统变量转换为Integer：
       ```
     
       ~~~java
       ```java
       Integer.getInteger("java.version"); // 版本号，11
       - 转换为char[]
       String和char[]类型可以互相c转换，方法是：
       ```java
       char[] cs = "Hello".toCharArray(); // String -> char[]
       String s = new String(cs); // char[] -> String
       ```
       如果修改了char[]数组，String并不会改变：
       ~~~
     
  
2. 集合

   1. List

      ```
      在末尾添加一个元素：void add(E e)
      在指定索引添加一个元素：void add(int index, E e)
      删除指定索引的元素：int remove(int index)
      删除某个元素：int remove(Object e)
      获取指定索引的元素：E get(int index)
      获取链表大小（包含元素的个数）：int size()
      ```

      ```java
      List<String> list = Arrays.asList("a","b","c","d");
      List<String> list = new ArrayList();//lsit.add()
      List<String> list = new ArrayList<>();//lsit.add()
      
      //对只读List调用add()、remove()方法会抛出UnsupportedOperationException。
      //要注意的是，返回的List不一定就是ArrayList或者LinkedList，因为List只是一个接口，如果我们调用List.of()，它返回的是一个只读List：
   List<String> list = List.of("a","b","c","d");
      List<Integer> list = List.of(1, 2, 5);
      
      List<String> temp = Lists.newArrayList();
      ```
   ```
      
      - 遍历List
      
        ```java
        List<String> list = List.of("apple", "pear", "banana");
        for (int i=0; i<list.size(); i++) {
        	String s = list.get(i);
     	System.out.println(s);
        }
   ```

     Iterator本身也是一个对象，但它是由List的实例调用iterator()方法的时候创建的。Iterator对象知道如何遍历一个List，并且不同的List类型，返回的Iterator对象实现也是不同

        Iterator对象有两个方法：boolean hasNext()判断是否有下一个元素，E next()返回下一个元素
          
        ```java
        for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
        	String s = it.next();
     	System.out.println(s);
        }
     ```
      
        实际上，只要实现了Iterable接口的集合类都可以直接用for each循环来遍历，Java编译器本身并不知道如何遍历集合对象，但它会自动把for each循环变成Iterator的调用，原因就在于Iterable接口定义了一个Iterator<E> iterator()方法，强迫集合类必须返回一个Iterator实例
      
        ```java
        for (String s : list) {
     	System.out.println(s);
        }
     ```

        Java 8 中我们可以通过 `::` 关键字来访问类的构造方法，对象方法，静态方法
          
        ```java
        public class Main {
            public static void printValur(String str) {
                System.out.println("print value : " + str);
            }
        
            public static void main(String[] args) {
                List<String> list = Arrays.asList("a", "b", "c", "d");
                for (String a : list) {
                    Main.printValur(a);
                }
                //下面的for each循环和上面的循环是等价的
                System.out.println('\n');
                list.forEach(x -> {
                    Main.printValur(x);
                });
                System.out.println('\n');
                list.forEach(Main::printValur);
                System.out.println('\n');
                //在JDK8中，接口Iterable 8中默认实现了forEach方法，调用了 JDK8中增加的接口Consumer内的accept方法，执行传入的方法参数
                Consumer<String> c = Main::printValur;
                list.forEach(x -> c.accept(x));
         }
        }
     ```
      
     　**另外补充一下，JDK8改动的，在接口里面可以有默认实现，就是在接口前加上default，实现这个接口的函数对于默认实现的方法可以不用再实现了。类似的还有static方法。现在这种接口除了上面提到的，还有BiConsumer,BiFunction,BinaryOperation等，在java.util.function包下的接口，大多数都有，后缀为Supplier的接口没有和别的少数接口**
      
      - List->Array
      
        1. ```java
           这种方法会丢失类型信息，所以实际应用很少
        List<String> list = List.of("apple", "pear", "banana");
           Object[] array = list.toArray();
     ```

        2. ```java
           给toArray(T[])传入一个类型相同的Array，List内部自动把元素复制到传入的Array中
        List<Integer> list = List.of(12, 34, 56);
           Integer[] array = list.toArray(new Integer[3]);
           ```
          
        3. ```java
           实际上可以传入其他类型的数组
           List<Integer> list = List.of(12, 34, 56);
           Number[] array = list.toArray(new Number[3]);
           如果我们传入类型不匹配的数组，例如，String[]类型的数组，由于List的元素是Integer，所以无法放入String数组，这个方法会抛出ArrayStoreException
           
        //最常用的是传入一个“恰好”大小的数组：
           Integer[] array = list.toArray(new Integer[list.size()]);
           ```
          
        4. ```java
           //函数式写法
        最后一种更简洁的写法是通过List接口定义的T[] toArray(IntFunction<T[]> generator)方法：
           Integer[] array = list.toArray(Integer[]::new);
        ```

      - Array->List
      
        ```java
        Integer[] array = { 1, 2, 3 };
        //好像被舍弃
        List<Integer> list = List.of(array);
        //要注意的是，返回的List不一定就是ArrayList或者LinkedList，因为List只是一个接口，如果我们调用List.of()，它返回的是一个只读List
        List<Integer> list = Arrays.asList(array);
        ```

       List<Integer> list = List.of(12, 34, 56);
        list.add(999); // 对只读List调用add()、remove()方法会抛出UnsupportedOperationException
        ```
          
        ```java
        //对于JDK 11之前的版本，使用Arrays.asList(T...)方法把数组转换成List
        List<String> List = Arrays.asList("a","b","c","d");
        ```

3. Stream()

   Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据，这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

   ```java
   +--------------------+       +------+   +------+   +---+   +-------+
   | stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
   +--------------------+       +------+   +------+   +---+   +-------+
   to java
   List<Integer> transactionsIds = 
   widgets.stream()
                .filter(b -> b.getColor() == RED)
                .sorted((x,y) -> x.getWeight() - y.getWeight())
                .mapToInt(Widget::getWeight)
                .sum();
   ```

   - 什么是 Stream？

     Stream（流）是一个来自数据源的元素队列并支持聚合操作

     - 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
     - **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
     - **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

     和以前的Collection操作不同， Stream操作还有两个基础的特征：

     - **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
     - **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

   - 生成流

     在 Java 8 中, 集合接口有两个方法来生成流：

     - **stream()** − 为集合创建串行流。
     - **parallelStream()** − 为集合创建并行流。

     ```java
     List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
     List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
     ```

   - forEach
     Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

     ```java
     Random random = new Random();
     random.ints().limit(10).forEach(System.out::println);
     ```

   - map

     map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

     ```java
     List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); // 获取对应的平方数 
     List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
     ```

   - filter

     filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

     ```java
     List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); // 获取空字符串的数量 
     long count = strings.stream().filter(string -> string.isEmpty()).count();
     ```

   - limit

     limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

     ```java
     Random random = new Random(); 
     random.ints().limit(10).forEach(System.out::println);
     ```

   - sorted

     sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

     ```java
     Random random = new Random(); 
     random.ints().limit(10).sorted().forEach(System.out::println);
     ```

   - 并行（parallel）程序

     parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：

     ```java
     List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
     // 获取空字符串的数量 
     int count = strings.parallelStream().filter(string -> string.isEmpty()).count();
     ```

     我们可以很容易的在顺序运行和并行直接切换。

   - Collectors

     Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

     ```java
     List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
     List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());  System.out.println("筛选列表: " + filtered); String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", ")); System.out.println("合并字符串: " + mergedString);
     ```

   - 统计

     另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

     ```java
     List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);  
     IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics(); 
     
     System.out.println("列表中最大的数 : " + stats.getMax()); 
     System.out.println("列表中最小的数 : " + stats.getMin()); 
     System.out.println("所有数之和 : " + stats.getSum()); 
     System.out.println("平均数 : " + stats.getAverage());
     ```

     
