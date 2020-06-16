#### Io

1. #### File对象

   - 文件是非常重要的存储方式。Java的标准库java.io提供了File对象来操作文件和目录。

     ```java
     File f = new File("C:\\Windows\\notepad.exe");
     ```

   - File对象既可以表示文件，也可以表示目录

   - 调用isFile()，判断该File对象是否是一个已存在的文件，调用isDirectory()，判断该File对象是否是一个已存在的目录

   - File对象获取到一个文件时，还可以进一步判断文件的权限和大小：

     ```java
     boolean canRead()：是否可读；
     boolean canWrite()：是否可写；
     boolean canExecute()：是否可执行；
     long length()：文件字节大小。
     ```

   - 对目录而言，是否可执行表示能否列出它包含的文件和子目录。

   - 当File对象表示一个文件时，可以通过createNewFile()创建一个新文件，用delete()删除该文件

   - 有些时候，程序需要读写一些临时文件，File对象提供了createTempFile()来创建一个临时文件，以及deleteOnExit()在JVM退出时自动删除该文件

     ```java
     File f = File.createTempFile("tmp-", ".txt"); // 提供临时文件的前缀和后缀
     f.deleteOnExit(); // JVM退出时自动删除
     ```

   - 当File对象表示一个目录时，可以使用list()和listFiles()列出目录下的文件和子目录名。listFiles()提供了一系列重载方法，可以过滤不想要的文件和目录

     ```java
     public static void main(String[] args) throws IOException {
             File f = new File("C:\\Windows");
             File[] fs1 = f.listFiles(); // 列出所有文件和子目录
             printFiles(fs1);
             File[] fs2 = f.listFiles(new FilenameFilter() { // 仅列出.exe文件
                 public boolean accept(File dir, String name) {
                     return name.endsWith(".exe"); // 返回true表示接受该文件
                 }
             });
             printFiles(fs2);
         }
     //list只会列举出当前目录下的文件名,例如C:\\windows\\paper.pdf只有paper.pdf
         static void printFiles(File[] files) {
             System.out.println("==========");
             if (files != null) {
                 for (File f : files) {
                     System.out.println(f);
                 }
             }
             System.out.println("==========");
         }
     }
     ```

2. #### Path对象

   Java标准库还提供了一个Path对象，它位于java.nio.file包。Path对象和File对象类似，但操作更加简单

   ```java
   Path p1 = Paths.get(".", "project", "study"); // 构造一个Path对象
   System.out.println(p1);
   Path p2 = p1.toAbsolutePath(); // 转换为绝对路径
   System.out.println(p2);
   Path p3 = p2.normalize(); // 转换为规范路径
   System.out.println(p3);
   File f = p3.toFile(); // 转换为File对象
   System.out.println(f);
   for (Path p : Paths.get("..").toAbsolutePath()) { // 可以直接遍历Path，例如C:\\windows\\paper.pdf只有windows  和   paper.pdf   分别打印
       System.out.println("  " + p);
   }
   ```

   如果需要对目录进行复杂的拼接、遍历等操作，使用`Path`对象更方便。

3. #### InputStream

   要特别注意的一点是，InputStream并不是一个接口，而是一个抽象类，它是所有输入流的超类。这个抽象类定义的一个最重要的方法就是int read()

   - ##### FileInputStream

     - 这个方法会读取输入流的下一个字节，并返回字节表示的int值（0~255）。如果已读到末尾，返回-1表示不能继续读取了

       ```java
       InputStream input = new FileInputStream("src/readme.txt")
       ```

     - 编译器并不会特别地为InputStream加上自动关闭。编译器只看try(resource = ...)中的对象是否实现了java.lang.AutoCloseable接口，如果实现了，就自动加上finally语句并调用close()方法。InputStream和OutputStream都实现了这个接口，因此，都可以用在try(resource)中

   - ##### 缓冲

     - int read(byte[] b)：读取若干字节并填充到byte[]数组，返回读取的字节数
       int read(byte[] b, int off, int len)：指定byte[]数组的偏移量和最大填充数
     - 利用上述方法一次读取多个字节时，需要先定义一个byte[]数组作为缓冲区，read()方法会尽可能多地读取字节到缓冲区， 但不会超过缓冲区的大小。read()方法的返回值不再是字节的int值，而是返回实际读取了多少个字节。如果返回-1，表示没有更多的数据了

     ```java
     try (InputStream input = new FileInputStream("src/readme.txt")) {
             // 定义1000个字节大小的缓冲区:
             byte[] buffer = new byte[1000];
             int n;
             while ((n = input.read(buffer)) != -1) { // 读取到缓冲区
                 System.out.println("read " + n + " bytes.");
             }
         }
     
     ByteArrayInputStream可以在内存中模拟一个InputStream：
     byte[] data = { 72, 101, 108, 108, 111, 33 };
             try (InputStream input = new ByteArrayInputStream(data)) {
                 int n;
                 while ((n = input.read()) != -1) {
                     System.out.println((char)n);
                 }
             }
     ```

   - ##### ByteArrayInputStream

     实际上是把一个byte[]数组在内存中变成一个InputStream，虽然实际应用不多，但测试的时候，可以用它来构造一个InputStream

4. #### OutputStream

   - OutputStream是Java标准库提供的最基本的输出流。OutputStream也是抽象类，它是所有输出流的超类。这个抽象类定义的一个最重要的方法就是**void write(int b)**

   - 这个方法会写入一个字节到输出流。要注意的是，虽然传入的是int参数，但只会写入一个字节，即只写入**int最低8位表示字节的部分**（相当于b & 0xff）

   - 要特别注意：OutputStream还提供了一个**flush**()方法，它的目的是将缓冲区的内容真正输出到目的地。

   - ##### FileOutputStream

     - 每次写入一个字节非常麻烦，更常见的方法是一次性写入若干字节。这时，可以用OutputStream提供的重载方法**void write(byte[])**来实现

       ```java
       OutputStream output = new FileOutputStream("out/readme.txt");
       output.write("Hello".getBytes("UTF-8")); // Hello
       output.close();
       ```

   - ##### ByteArrayOutputStream

     ```java
     ByteArrayOutputStream可以在内存中模拟一个OutputStream
     byte[] data;
     try (ByteArrayOutputStream output = new ByteArrayOutputStream()) {
         output.write("Hello ".getBytes("UTF-8"));
         output.write("world!".getBytes("UTF-8"));
         data = output.toByteArray();
     }
     System.out.println(new String(data, "UTF-8"));
     ```

     - ByteArrayOutputStream实际上是把一个byte[]数组在内存中变成一个OutputStream，虽然实际应用不多，但测试的时候，可以用它来构造一个OutputStream

5. #### 序列化

   ```java
   public class XuLieHua {
       public static void main(String[] args) {
           //序列化
           ByteArrayOutputStream buffer = new ByteArrayOutputStream();
           try (ObjectOutputStream output = new ObjectOutputStream(buffer)) {
   //            output.writeInt(123456);
   //            output.writeUTF("Hello");
               output.writeObject(Double.valueOf(123.456));
               output.writeObject(new Person());
   
           } catch (IOException e) {
               e.printStackTrace();
           }
           System.out.println(Arrays.toString(buffer.toByteArray()));
           
           //反序列化
           byte[] b ={-84, -19, 0, 5, 119, 11, 0, 1, -30, 64, 0, 5, 72, 101, 108, 108, 111, 115, 114, 0, 16, 106, 97, 118, 97, 46, 108, 97, 110, 103, 46, 68, 111, 117, 98, 108, 101, -128, -77, -62, 74, 41, 107, -5, 4, 2, 0, 1, 68, 0, 5, 118, 97, 108, 117, 101, 120, 114, 0, 16, 106, 97, 118, 97, 46, 108, 97, 110, 103, 46, 78, 117, 109, 98, 101, 114, -122, -84, -107, 29, 11, -108, -32, -117, 2, 0, 0, 120, 112, 64, 94, -35, 47, 26, -97, -66, 119, 115, 114, 0, 54, 101, 100, 117, 46, 117, 101, 115, 116, 99, 46, 97, 117, 116, 111, 46, 114, 101, 97, 115, 111, 110, 105, 110, 103, 46, 76, 111, 99, 97, 108, 67, 104, 76, 105, 117, 46, 76, 101, 97, 114, 110, 105, 110, 103, 46, 73, 111, 46, 80, 101, 114, 115, 111, 110, -110, -27, -9, 49, 119, 104, -16, -98, 2, 0, 1, 76, 0, 4, 110, 97, 109, 101, 116, 0, 18, 76, 106, 97, 118, 97, 47, 108, 97, 110, 103, 47, 83, 116, 114, 105, 110, 103, 59, 120, 112, 116, 0, 9, 67, 104, 101, 117, 110, 103, 76, 105, 117};
           ByteArrayInputStream buffer = new ByteArrayInputStream(b);
           try(ObjectInputStream input=new ObjectInputStream(buffer)) {
               Person p = (Person) input.readObject();
               System.out.println(p.name);
           } catch (ClassNotFoundException e) {
               e.printStackTrace();
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
   class Person implements Serializable {
       public String name = "CheungLiu";
   }
   ```

6. #### Reader

   Reader是Java的IO库提供的另一个输入流接口。和InputStream的区别是，InputStream是一个字节流，即以byte为单位读取，而Reader是一个字符流，即以char为单位读取

   ```java
   	InputStream									Reader
   字节流，以byte为单位						 字符流，以char为单位
   读取字节（-1，0~255）：int read()	  		 读取字符（-1，0~65535）：int read()
   读到字节数组：int read(byte[] b)			  读到字符数组：int read(char[] c)
   ```

   - java.io.Reader是所有字符**输入流的超类**，它最主要的方法是

     ```java
     public int read() throws IOException;
     ```

   - 这个方法读取字符流的下一个字符，并返回**字符表示的int**，范围是0~65535。如果已读到末尾，返回-1

   - ##### FileReader

     - FileReader是Reader的一个子类，它可以打开文件并获取Reader。下面的代码演示了如何完整地读取一个FileReader的所有字符

       ```java
       // 创建一个FileReader对象:
       Reader reader = new FileReader("src/readme.txt"); // 字符编码是???
       for (;;) {
       	int n = reader.read(); // 反复调用read()方法，直到返回-1
       	if (n == -1) {
       		break;
       	}
       	System.out.println((char)n); // 打印char
       }
       reader.close(); // 关闭流
       ```

     - 要避免乱码问题，我们需要在创建FileReader时指定编码：

       ```java
       Reader reader = new FileReader("src/readme.txt", StandardCharsets.UTF_8);
       ```

     - Reader还提供了一次性读取**若干字符**并填充到char[]数组的方法：

       ```java
       public int read(char[] c) throws IOException
       ```

     - 它**返回实际读入的字符个数**，最大不超过char[]数组的长度。返回-1表示流结束,利用这个方法，我们可以先设置一个缓冲区，然后，每次尽可能地填充缓冲区：

       ```java
       public void readFile() throws IOException {
       try (Reader reader = new FileReader("src/readme.txt", StandardCharsets.UTF_8)) 	{
               char[] buffer = new char[1000];
               int n;
               while ((n = reader.read(buffer)) != -1) {
                   System.out.println("read " + n + " chars.");
               }
          }
       }
       ```

   - ##### CharArrayReader

     - CharArrayReader可以在内存中模拟一个Reader，它的作用实际上是把一个char[]数组变成一个Reader，这和ByteArrayInputStream非常类似：

       ```java
       try (Reader reader = new CharArrayReader("Hello".toCharArray())) {
           ...
       }
       ```

   - ##### StringReader

     - StringReader可以直接把String作为数据源，它和CharArrayReader几乎一样：

       ```java
       try (Reader reader = new StringReader("Hello")) {
       	...
       }
       ```

   - ##### *InputStreamReader*

     **Reader和InputStream有什么关系？**

     - 除了特殊的CharArrayReader和StringReader，普通的Reader实际上是基于InputStream构造的，因为Reader需要从InputStream中读入字节流（byte），然后，根据编码设置，再转换为char就可以实现字符流。如果我们查看FileReader的源码，它在内部实际上持有一个FileInputStream。

     - 既然Reader本质上是一个基于InputStream的byte到char的转换器，那么，如果我们已经有一个InputStream，想把它转换为Reader，是完全可行的。InputStreamReader就是这样一个转换器，它可以把任何InputStream转换为Reader。示例代码如下：

       ```java
       // 持有InputStream:
       InputStream input = new FileInputStream("src/readme.txt");
       // 变换为Reader:
       Reader reader = new InputStreamReader(input, "UTF-8");
       ```

     - 构造InputStreamReader时，我们需要传入InputStream，还需要指定编码，就可以得到一个Reader对象。上述代码可以通过try (resource)更简洁地改写如下：

       ```java
       try (Reader r = new InputStreamReader(new FileInputStream("src/readme.txt"), "UTF-8")) {
           // TODO:
       }
       ```

     - 上述代码实际上就是FileReader的一种实现方式，使用try (resource)结构时，当我们关闭Reader时，它会在内部自动调用InputStream的close()方法，所以，只需要关闭最外层的Reader对象即可

7. #### Writer

   - Reader是带编码转换器的InputStream，它把byte转换为char，而Writer就是带编码转换器的OutputStream，它把char转换为byte并输出

   - Writer和OutputStream的区别如下：

     ```
     OutputStream										Writer
     字节流，以byte为单位							字符流，以char为单位
     写入字节（0~255）：void write(int b)		写入字符（0~65535）：void write(int c)
     写入字节数组：void write(byte[] b)			写入字符数组：void write(char[] c)
     无对应方法								  写入String：void write(String s)
     ```

   - Writer是所有字符**输出流的超类**，它提供的方法主要有

     ```java
     写入一个字符（0~65535）：void write(int c)；
     写入字符数组的所有字符：void write(char[] c)；
     写入String表示的所有字符：void write(String s)。
     ```

   - ##### FileWriter

     - FileWriter就是向文件中写入字符流的Writer。它的使用方法和FileReader类似：

       ```java
       try (Writer writer = new FileWriter("readme.txt", StandardCharsets.UTF_8)) {
           writer.write('H'); // 写入单个字符
           writer.write("Hello".toCharArray()); // 写入char[]
           writer.write("Hello"); // 写入String
       }
       ```

   - ##### CharArrayWriter

     - CharArrayWriter可以在内存中创建一个Writer，它的作用实际上是构造一个缓冲区，可以写入char，最后得到写入的char[]数组，这和ByteArrayOutputStream非常类似：

       ```java
       try (CharArrayWriter writer = new CharArrayWriter()) {
           writer.write(65);
           writer.write(66);
           writer.write(67);
           char[] data = writer.toCharArray(); // { 'A', 'B', 'C' }
       }
       ```

   - ##### StringWriter

     - StringWriter也是一个基于内存的Writer，它和CharArrayWriter类似。实际上，StringWriter在内部维护了一个StringBuffer，并对外提供了Writer接口

   - ##### OutputStreamWriter

     - 除了CharArrayWriter和StringWriter外，普通的Writer实际上是基于OutputStream构造的，它接收char，然后在内部自动转换成一个或多个byte，并写入OutputStream。因此，OutputStreamWriter就是一个将任意的OutputStream转换为Writer的转换器：

       ```java
       try (Writer writer = new OutputStreamWriter(new FileOutputStream("readme.txt"), "UTF-8")) {
           // TODO:
       }
       ```

     - 上述代码实际上就是FileWriter的一种实现方式。这和上一节的InputStreamReader是一样的

8. #### 总结

   ```
   1.File
   2.Path
   3.InputStream							OutputStream
   	-FileInputStream				-FileOutputStream
   	-ByteArrayInputStream			-ByteArrayOutputStream
   4.Reader								Writer
   	-FileReader						-FileWriter
   	-CharArrayReader				-CharArrayWriter
   	-StringRader					-StringWriter
   	-InputStreamReader				-OutputStreamWriter			(3和4的转换)
   ```

   