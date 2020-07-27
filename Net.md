## 目录：Net

[TOC]

### TCP编程

- Socket是一个抽象概念，一个应用程序通过一个Socket来建立一个远程连接，而Socket内部通过TCP/IP协议把数据传输到网络
- Socket、TCP和部分IP的功能都是由操作系统提供的，不同的编程语言只是提供了对操作系统调用的简单的封装。例如，Java提供的几个Socket相关的类就封装了操作系统提供的接口
- 一个Socket就是由IP地址和端口号（范围是0～65535）组成
- 对服务器端来说，它的Socket是指定的IP地址和指定的端口号
- 对客户端来说，它的Socket是它所在计算机的IP地址和一个由操作系统分配的随机端口号

#### 一、ServerSocket（服务器端）

- Java标准库提供了`ServerSocket`来实现对指定IP和指定端口的监听

  ```java
  public class ServerSocketTest {
      public static void main(String[] args) throws IOException {
          ServerSocket ss=new ServerSocket(6666);
          System.out.println("server is running...");
          //如果没有客户端连接进来，accept()方法会阻塞并一直等待。如果有多个客户端同时连接进来，ServerSocket会把连接扔到队列里，然后一个一个处理。对于Java程序而言，只需要通过循环不断调用accept()就可以获取新的连接
          while (true){
              //ss.accept()表示每当有新的客户端连接进来后，就返回一个Socket实例，这个Socket实例就是用来和刚连接的客户端进行通信的
              Socket sock = ss.accept();
              System.out.println("connected from " + sock.getRemoteSocketAddress());
              Thread t = new Handler(sock);
              t.start();
          }
      }
  }
  class Handler extends Thread {
      Socket sock;
  
      public Handler(Socket sock) {
          this.sock = sock;
      }
  
      @Override
      public void run() {
          try (InputStream input = this.sock.getInputStream()) {
              try(OutputStream output= this.sock.getOutputStream()) {
                  handle(input,output);
  
              }
          } catch (IOException e) {
              try {
                  this.sock.close();
              } catch (IOException e1) {
              }
              System.out.println("client disconnected.");
          }
      }
  
      private void handle(InputStream input,OutputStream output) throws IOException {
          BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
          BufferedReader reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
          writer.write("hello\n");
          writer.flush();
          while(true){
              String s =reader.readLine();
              if(s.equals("bye")){
                  writer.write("bye\n");
                  writer.flush();
                  break;
              }
              writer.write("ok"+s+"\n");
              writer.flush();
          }
      }
  }
  ```

#### 二、客户端

- 连接到服务器端，注意上述代码的服务器地址是`"localhost"`，表示本机地址，端口号是`6666`。如果连接成功，将返回一个`Socket`实例，用于后续通信

  ```java
  public class ClientTest {
          public static void main(String[] args) throws IOException {
              Socket sock = new Socket("localhost", 7474);
              try (InputStream input = sock.getInputStream()) {
                  try (OutputStream output = sock.getOutputStream()) {
                      handle(input, output);
                  }
              }
              sock.close();
              System.out.println("disconnected.");
          }
  
          private static void handle(InputStream input, OutputStream output) throws IOException {
              BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
              BufferedReader reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
              Scanner scanner = new Scanner(System.in);
              System.out.println("[Sever] " + reader.readLine());
              while (true) {
                  System.out.print(">>> "); // 打印提示
                  String s = scanner.nextLine(); // 读取一行输入
                  writer.write(s);
                  writer.newLine();
                  writer.flush();
                  String resp = reader.readLine();
                  System.out.println("<<< " + resp);
                  if (resp.equals("bye")) {
                      break;
                  }
              }
          }
      }
  ```

- ##### Socket流

  当Socket连接创建成功后，无论是服务器端，还是客户端，我们都使用Socket实例进行网络通信。因为TCP是一种基于流的协议，因此，Java标准库使用InputStream和OutputStream来封装Socket的数据流，这样我们使用Socket的流，和普通IO流类似

  ```java
  // 用于读取网络数据:
  InputStream in = sock.getInputStream();
  // 用于写入网络数据:
  OutputStream out = sock.getOutputStream();
  ```

- ##### 小结

  ```
  使用Java进行TCP编程时，需要使用Socket模型：
  
  服务器端用ServerSocket监听指定端口；
  客户端使用Socket(InetAddress, port)连接服务器；
  服务器端用accept()接收连接并返回Socket；
  双方通过Socket打开InputStream/OutputStream读写数据；
  服务器端通常使用多线程同时处理多个客户端连接，利用线程池可大幅提升效率；
  flush()用于强制输出缓冲区到网络。
  ```

### UDP编程

#### 一、DatagramSocket（服务器端）

​		和TCP编程相比，UDP编程就简单得多，因为UDP没有创建连接，数据包也是一次收发一个，所以没有流的概念。在Java中使用UDP编程，仍然需要使用Socket，因为应用程序在使用UDP时必须指定网络接口（IP）和端口号。注意：UDP端口和TCP端口虽然都使用0~65535，但他们是两套独立的端口，即一个应用程序用TCP占用了端口1234，不影响另一个应用程序用UDP占用端口1234

```java
public static void main(String[] args) throws IOException {
        // 服务器端首先使用如下语句在指定的端口监听UDP数据包
        DatagramSocket ds = new DatagramSocket(6666);
        while (true) {
            // 要接收一个UDP数据包，需要准备一个byte[]缓冲区，并通过DatagramPacket实现接收
            byte[] buffer = new byte[1024];
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
            ds.receive(packet);//收取一个UDP数据包
            // 收取到的数据存储在buffer中，由packet.getOffset(), packet.getLength()指定起始位置和长度
            // 将其按UTF-8编码转换为String:
            // 假设我们收取到的是一个String，那么，通过DatagramPacket返回的packet.getOffset()和packet.getLength()确定数据在缓冲区的起止位置
            String s = new String(packet.getData(), packet.getOffset(), packet.getLength(), StandardCharsets.UTF_8);
            // 当服务器收到一个DatagramPacket后，通常必须立刻回复一个或多个UDP包，因为客户端地址在DatagramPacket中，每次收到的DatagramPacket可能是不同的客户端，如果不回复，客户端就收不到任何UDP包           
            // 发送数据
            byte[] data = "ACK".getBytes(StandardCharsets.UTF_8);
            packet.setData(data);
            // 发送UDP包也是通过DatagramPacket实现的，发送代码非常简单
            ds.send(packet);
        }
    }
```

#### 二、客户端

和服务器端相比，客户端使用UDP时，只需要直接向服务器端发送UDP包，然后接收返回的UDP包

```java
public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket();
        ds.setSoTimeout(1000);
        ds.connect(InetAddress.getByName("localhost"),6666);// 连接指定服务器和端口
        //发送
        byte[] data="Hello".getBytes();
        DatagramPacket packet = new DatagramPacket(data, data.length);
        ds.send(packet);
        //接收
        byte[] buffer= new byte[1024];
        packet=new DatagramPacket(buffer,buffer.length);
        ds.receive(packet);
        String resp=new String(packet.getData(),packet.getOffset(),packet.getLength());
        ds.disconnect();
    }
```

客户端的DatagramSocket还调用了一个connect()方法“连接”到指定的服务器端，这个connect()方法不是真连接，它是为了在客户端的DatagramSocket实例中保存服务器端的IP和端口号，确保这个DatagramSocket实例只能往指定的地址和端口发送UDP包，不能往其他地址和端口发送。这么做不是UDP的限制，而是Java内置了安全检查

- 小结

  ```
  使用UDP协议通信时，服务器和客户端双方无需建立连接：
  
  服务器端用DatagramSocket(port)监听端口；
  客户端使用DatagramSocket.connect()指定远程地址和端口；
  双方通过receive()和send()读写数据；
  DatagramSocket没有IO流接口，数据被直接写入byte[]缓冲区。
  ```

  