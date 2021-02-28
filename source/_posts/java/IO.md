# IO

## 1 字符流和字节流

1. 字节与字符
- Bit最小的二进制单位 ，是计算机的操作部分。取值0或者1
- Byte（字节）是计算机操作数据的最小单位由8位bit组成 取值（-128-127）
- Char（字符）是用户的可读写的最小单位，在Java里面由16位bit组成 取值（0-65535）
2. 字节流
- 操作byte类型数据，主要操作类是OutputStream、InputStream的子类；不用缓冲区，直接对文件本身操作。
3. 字符流
- 操作字符类型数据，主要操作类是Reader、Writer的子类；使用缓冲区缓冲字符，不关闭流就不会输出任何内容。
4. 互相转换
- 整个IO包实际上分为字节流和字符流，但是除了这两个流之外，还存在一组字节流-字符流的转换类。
- OutputStreamWriter: 是Writer的子类，将输出的字符流变为字节流，即将一个字符流的输出对象变为字节流输出对象。
- InputStreamReader: 是Reader的子类，将输入的字节流变为字符流，即将一个字节流的输入对象变为字符流的输入对象。

## 2 字节流和字符流之间的相互转换

- OutputStreamWriter是字符流通向字节流的桥梁
- InputStreamReader是字节流通向字符流的桥梁

1. 字符流转成字节流
```
public static void main(String[] args) throws IOException {
    File f = new File("test.txt");

    // OutputStreamWriter 是字符流通向字节流的桥梁,创建了一个字符流通向字节流的对象
    OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(f),"UTF-8");

    osw.write("我是字符流转换成字节流输出的");
    osw.close();

}
```
2. 字节流转成字符流
```
  public static void main(String[] args) throws IOException {

        File f = new File("test.txt");

        InputStreamReader inr = new InputStreamReader(new FileInputStream(f),"UTF-8");

        char[] buf = new char[1024];

        int len = inr.read(buf);
        System.out.println(new String(buf,0,len));

        inr.close();

    }
```

## 3 同步， 异步和阻塞，非阻塞之间的区别

1. 同步：A调用B，B在接到A的调用后，会立即执行要做的事。A的本次调用可以得到结果。
2. 异步：如果是异步，B在接到A的调用后，不保证会立即执行要做的事，但是保证会去做，B在做好了之后会通知A。A的本次调用得不到结果，但是B执行完之后会通知A。

- 同步，异步，是描述被调用方的
- 阻塞，非阻塞，是描述调用方的
- 同步不一定阻塞，异步也不一定非阻塞。没有必然关系。
> 举个简单的例子，老张烧水。 1 老张把水壶放到火上，一直在水壶旁等着水开。（同步阻塞） 2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞） 3 老张把响水壶放到火上，一直在水壶旁等着水开。（异步阻塞） 4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）

总结：同步，异步的区别在于是否有回调。阻塞，非阻塞的区别在于调用方是否等待。

