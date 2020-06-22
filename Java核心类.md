### Java核心类

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

     - "Hello".substring(2, 4); "ll"

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

       常用的占位符有：

       ```
       %s：显示字符串；
       %d：显示整数；
       %x：显示十六进制整数；
       %f：显示浮点数。
       ```

       占位符还可以带格式，例如%.2f表示显示两位小数。如果你不确定用啥占位符，那就始终用%s

     - 类型转换

       要把任意基本类型或引用类型转换为字符串，可以使用静态方法valueOf()。这是一个重载方法，编译器会根据参数自动选择合适的方法

       ```java
       String.valueOf(123); // "123"
       String.valueOf(45.67); // "45.67"
       String.valueOf(true); // "true"
       String.valueOf(new Object()); // 类似java.lang.Object@636be97c
       ```

       ```java
       int n1 = Integer.parseInt("123"); // 123
       int n2 = Integer.parseInt("ff", 16); // 按十六进制转换，255
       ```

       ```java
       boolean b1 = Boolean.parseBoolean("true"); // true
       boolean b2 = Boolean.parseBoolean("FALSE"); // false
       ```

       要特别注意，Integer有个getInteger(String)方法，它不是将字符串转换为int，而是把该字符串对应的系统变量转换为Integer：

       ```java
       Integer.getInteger("java.version"); // 版本号，11
       ```

     - 转换为char[]

       String和char[]类型可以互相转换，方法是：

       ```java
       char[] cs = "Hello".toCharArray(); // String -> char[]
       String s = new String(cs); // char[] -> String
       ```

       如果修改了char[]数组，String并不会改变：